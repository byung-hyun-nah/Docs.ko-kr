---
title: "ASP.NET Core의 미들웨어 캐싱 응답"
author: guardrex
description: "구성 및 ASP.NET 코어에서 응답 캐싱 미들웨어를 사용 하는 방법을 알아봅니다."
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 01/26/2017
ms.prod: asp.net-core
ms.topic: article
uid: performance/caching/middleware
ms.openlocfilehash: 29ef3cf3d8bcd6b4ebbf08d831dc146e830fa1ac
ms.sourcegitcommit: b83a5f731a9c02bdb1cc1e3f9a8bf273eb5b33e0
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/11/2018
---
# <a name="response-caching-middleware-in-aspnet-core"></a>ASP.NET Core의 미들웨어 캐싱 응답

여 [Luke Latham](https://github.com/guardrex) 및 [John Luo](https://github.com/JunTaoLuo)

[샘플 코드 보기 또는 다운로드](https://github.com/aspnet/Docs/tree/master/aspnetcore/performance/caching/middleware/sample)([다운로드 방법](xref:tutorials/index#how-to-download-a-sample))

이 문서에서는 ASP.NET Core 응용 프로그램의 응답 캐싱 미들웨어를 구성 하는 방법에 설명 합니다. 미들웨어는 응답에 캐시할 수 있는 경우, 저장소 응답 및 캐시에서 응답 하는 데 사용 결정 합니다. HTTP 캐시에 대 한 소개 및 `ResponseCache` 특성을 참조 하십시오. [응답 캐시](xref:performance/caching/response)합니다.

## <a name="package"></a>패키지

미들웨어는 프로젝트에 포함 하려면에 대 한 참조를 추가 [ `Microsoft.AspNetCore.ResponseCaching` ](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCaching/) 패키지 하거나 사용 하 여는 [ `Microsoft.AspNetCore.All` ](https://www.nuget.org/packages/Microsoft.AspNetCore.All/) 패키지 (ASP.NET 코어 2.0 이상.NET Core를 대상으로 할 때).

## <a name="configuration"></a>구성

`ConfigureServices`, 미들웨어 서비스 컬렉션에 추가 합니다.

[!code-csharp[Main](middleware/sample/Startup.cs?name=snippet1&highlight=3)]

사용 하 여 미들웨어를 사용 하도록 응용 프로그램 구성에서 `UseResponseCaching` 요청 처리 파이프라인에 미들웨어를 추가 하는 확장 메서드를 합니다. 샘플 응용 프로그램 추가 [ `Cache-Control` ](https://tools.ietf.org/html/rfc7234#section-5.2) 헤더를 최대 10 초 동안 캐시 가능한 응답을 캐시 하는 응답입니다. 샘플에서 보내기는 [ `Vary` ](https://tools.ietf.org/html/rfc7231#section-7.1.4) 를 경우에만 캐시 된 응답을 처리 하는 미들웨어를 구성 하는 헤더는 [ `Accept-Encoding` ](https://tools.ietf.org/html/rfc7231#section-5.3.4) 이후의 요청 헤더의 일치 하는 원래 요청 합니다.

[!code-csharp[Main](middleware/sample/Startup.cs?name=snippet2&highlight=3,7-12)]

응답의 캐싱 미들웨어는만 200 (정상) 상태 코드가 서버 응답을 캐시 합니다. 포함 하 여 모든 다른 응답 [오류 페이지](xref:fundamentals/error-handling), 미들웨어에서 무시 됩니다.

> [!WARNING]
> 인증 된 클라이언트에 대 한 콘텐츠를 포함 하는 응답에서 저장 하 고 해당 응답을 처리 하는 미들웨어를 방지 하기 위해 불가능으로 표시 되어야 합니다. 참조 [캐시에 대 한 조건을](#conditions-for-caching) 응답을 캐시할 수 있으면 미들웨어를 결정 하는 방법에 대 한 내용은 합니다.

## <a name="options"></a>옵션

미들웨어 응답 캐시 제어에 대 한 세 가지 옵션이 표시 됩니다.

| 옵션                | 기본값 |
| --------------------- | ------------- |
| UseCaseSensitivePaths | 대/소문자 구분 경로에 응답 하면 캐시를 결정 합니다.</p><p>기본값은 `false`입니다. |
| MaximumBodySize       | 바이트의 응답 본문에 대 한 가장 큰 캐시 가능한 크기입니다.</p>기본값은 `64 * 1024 * 1024` (64MB). |
| SizeLimit             | 바이트에 대 한 응답 캐시 미들웨어에 대 한 크기 제한입니다. 기본값은 `100 * 1024 * 1024` (100MB). |

다음 예제에서는 미들웨어를를 구성합니다.

* 1, 024 바이트 보다 작거나 응답을 캐시 합니다.
* 대/소문자 구분 경로 의해 응답을 저장 합니다. (예를 들어 `/page1` 및 `/Page1` 별도로 저장) 합니다.

```csharp
services.AddResponseCaching(options =>
{
    options.UseCaseSensitivePaths = true;
    options.MaximumBodySize = 1024;
});
```

## <a name="varybyquerykeys"></a>VaryByQueryKeys

MVC/Web API 컨트롤러 또는 Razor 페이지 페이지 모델을 사용 하는 경우는 `ResponseCache` 특성 응답 캐시에 대 한 적절 한 헤더를 설정 하는 데 필요한 매개 변수를 지정 합니다. 유일한 매개 변수인은 `ResponseCache` 엄격 하 게 미들웨어를 필요로 하는 특성은 `VaryByQueryKeys`, 하는 실제 HTTP 헤더에 해당 하지 않습니다. 자세한 내용은 참조 [ResponseCache 특성](xref:performance/caching/response#responsecache-attribute)합니다.

사용 하지 않는 경우는 `ResponseCache` 특성 응답 캐시 달라질 수 있습니다와 `VaryByQueryKeys` 기능입니다. 사용 하 여 `ResponseCachingFeature` 에서 직접는 `IFeatureCollection` 의 `HttpContext`:

```csharp
var responseCachingFeature = context.HttpContext.Features.Get<IResponseCachingFeature>();
if (responseCachingFeature != null)
{
    responseCachingFeature.VaryByQueryKeys = new[] { "MyKey" };
}
```

에 단일 값을 사용 하 여 `*` 에 `VaryByQueryKeys` 캐시 모든 요청에 대 한 쿼리 매개 변수에 따라 달라 집니다.

## <a name="http-headers-used-by-response-caching-middleware"></a>응답의 캐싱 미들웨어에서 사용 되는 HTTP 헤더

미들웨어에서 응답 캐시 HTTP 헤더를 사용 하 여 구성 됩니다.

| Header | 설명 |
| ------ | ------- |
| 권한 부여 | 응답 헤더에 있는 경우 캐시 되지 않습니다. |
| Cache-Control | 미들웨어로 표시 하는 응답을 캐시 고려는 `public` 캐시 지시문입니다. 매개 변수가 캐싱을 제어:<ul><li>max-age</li><li>max-stale&#8224;</li><li>최소 새로</li><li>must-revalidate</li><li>캐시 없음</li><li>저장소 아니요</li><li>전용-if-캐시</li><li>private</li><li>public</li><li>기간</li><li>proxy-revalidate&#8225;</li></ul>&#8224;에 제한이 없음을 지정 하는 경우 `max-stale`, 미들웨어 아무 작업도 수행 합니다.<br>&#8225; `proxy-revalidate` 것과 동일한 결과가 `must-revalidate`합니다.<br><br>자세한 내용은 참조 [RFC 7231: 캐시 제어 지시문 요청](https://tools.ietf.org/html/rfc7234#section-5.2.1)합니다. |
| Pragma | A `Pragma: no-cache` 요청의 헤더 생성 한 것과 같습니다 `Cache-Control: no-cache`합니다. 이 헤더에 관련 지시문에 의해 재정의 되는 `Cache-Control` 헤더로, 있는 경우. HTTP/1.0과 함께 이전 버전과 호환성에 대 한 것으로 간주 합니다. |
| Set-Cookie | 응답 헤더에 있는 경우 캐시 되지 않습니다. |
| 변경 | `Vary` 헤더 다른 헤더에 의해 캐시 된 응답을 변경 하기 위해 사용 됩니다. 예를 들어, 포함 하 여 인코딩할 응답을 캐시는 `Vary: Accept-Encoding` 헤더와 요청에 대 한 응답을 캐시 하는 헤더 `Accept-Encoding: gzip` 및 `Accept-Encoding: text/plain` 별도로 합니다. 포함 된 응답의 헤더 값으로 `*` 저장 되지 않습니다. |
| 만료 | 이 헤더에 의해 부실 하다 고 판단 하는 응답 저장 아니거나 다른 재정의 되지 않는 경우 검색 `Cache-Control` 헤더입니다. |
| None-If-match | 값이 없는 경우 캐시에서 제공 되는 전체 응답 `*` 및 `ETag` 응답의은 제공 된 값이 하나라도 일치 하지 않습니다. 그렇지 않으면 304 (수정 되지 않음) 응답 서비스 됩니다. |
| If-수정-이후 | 경우는 `If-None-Match` 헤더 존재 하지, 캐시 된 응답 날짜에 제공 된 값 보다 최신인 경우 전체 응답 캐시에서 제공 됩니다. 그렇지 않으면 304 (수정 되지 않음) 응답 서비스 됩니다. |
| 날짜 | 캐시에서 서비스를 제공할 때는 `Date` 원래 응답에 제공 되지 않은 경우 헤더는 미들웨어에서 설정 됩니다. |
| 콘텐츠 길이 | 캐시에서 서비스를 제공할 때는 `Content-Length` 원래 응답에 제공 되지 않은 경우 헤더는 미들웨어에서 설정 됩니다. |
| 보존 기간 | `Age` 원래 응답으로 전송 하는 헤더는 무시 됩니다. 캐시 된 응답을 처리 하는 경우 새 값을 계산 하는 미들웨어입니다. |

## <a name="caching-respects-request-cache-control-directives"></a>요청 캐시 제어 지시문은 존중 캐싱

미들웨어의 규칙을 적용 된 [HTTP 1.1 캐싱 사양](https://tools.ietf.org/html/rfc7234#section-5.2)합니다. 규칙 (를) 처리할 유효한 캐시가 필요한 `Cache-Control` 클라이언트에서 보낸 헤더입니다. 사양에서 클라이언트 요청을 만들 수는 `no-cache` 헤더 값과 강제로 서버에 대 한 모든 요청에 대 한 새 응답을 생성 합니다. 현재는 개발자가 캐싱 동작을 제어할 미들웨어 공식 캐싱 사양을 준수 하기 때문에 미들웨어를 사용 하는 경우.

캐싱 동작 보다 자세히 제어 ASP.NET 코어의 다른 캐싱 기능을 탐색 합니다. 다음 항목을 참조하십시오.

* [메모리 내 캐싱](xref:performance/caching/memory)
* [분산 캐시 사용](xref:performance/caching/distributed)
* [ASP.NET Core MVC에서에서 태그 도우미를 캐시 합니다.](xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper)
* [분산 캐시 태그 도우미](xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper)

## <a name="troubleshooting"></a>문제 해결

캐싱 동작 없는 경우 예상 대로, 응답은 캐시할 수 되며 캐시에서 제공 되 고 수 있는지 확인 합니다. 들어오는 요청의 헤더 및 응답의 나가는 헤더를 검사 합니다. 사용 하도록 설정 [로깅](xref:fundamentals/logging/index) 디버깅에 도움이 되도록 합니다.

캐싱 동작 문제 해결을 테스트할 때 브라우저 바람직하지 않은 방법으로 캐싱에 영향을 주는 요청 헤더를 설정할 수 있습니다. 예를 들어 브라우저 설정할 수 있습니다는 `Cache-Control` 헤더를 `no-cache` 또는 `max-age=0` 페이지를 새로 고칠 때. 다음 도구 요청 헤더에 명시적으로 설정할 수 및 캐싱 테스트에 대 한 기본 설정 됩니다.

* [Fiddler](https://www.telerik.com/fiddler)
* [Postman](https://www.getpostman.com/)

### <a name="conditions-for-caching"></a>캐시에 대 한 조건

* 서버 응답 (정상) 상태 코드가 200 인 요청 발생 해야 합니다.
* 요청 메서드가 GET 또는 HEAD 이어야 합니다.
* 터미널 미들웨어와 같은 [정적 파일 미들웨어](xref:fundamentals/static-files), 응답 캐싱 미들웨어 하기 전에 응답을 처리 해야 합니다.
* `Authorization` 헤더 없어야 합니다.
* `Cache-Control`헤더 매개 변수는 유효 해야 하 고 응답을 표시 되어야 합니다 `public` 표시 되어 있지 `private`합니다.
* `Pragma: no-cache` 헤더 없어야 하는 경우는 `Cache-Control` 헤더로 존재 하지는 `Cache-Control` 헤더 재정의 `Pragma` 있는 경우 헤더입니다.
* `Set-Cookie` 헤더 없어야 합니다.
* `Vary`헤더 매개 변수는 유효 하 고 같지 않음 이어야 `*`합니다.
* `Content-Length` 헤더 값 (하는 경우 설정)는 응답 본문의 크기와 일치 해야 합니다.
* [IHttpSendFileFeature](/aspnet/core/api/microsoft.aspnetcore.http.features.ihttpsendfilefeature) 사용 되지 않습니다.
* 응답에 지정 된 대로 오래 된 않아야는 `Expires` 헤더 및 `max-age` 및 `s-maxage` 지시문을 캐시 합니다.
* 응답 버퍼링 성공 이어야 하며 응답 크기가 구성 된 보다 작은 또는 기본 `SizeLimit`합니다.
* 응답에 따라 캐시 가능 해야 합니다.는 [RFC 7234](https://tools.ietf.org/html/rfc7234) 사양입니다. 예를 들어는 `no-store` 지시문 요청 또는 응답 헤더 필드에 없어야 합니다. 참조 *섹션 3: 응답을 캐시에 저장* 의 [RFC 7234](https://tools.ietf.org/html/rfc7234) 대 한 자세한 내용은 합니다.

> [!NOTE]
> 교차 사이트 요청 위조 CSRF ()을 방지 하기 위해 보안 토큰을 생성 하기 위한 Antiforgery 시스템 공격 집합은 `Cache-Control` 및 `Pragma` 헤더를 `no-cache` 응답은 캐시 되지 않도록 합니다.

## <a name="additional-resources"></a>추가 리소스

* [응용 프로그램 시작](xref:fundamentals/startup)
* [미들웨어](xref:fundamentals/middleware/index)
* [메모리 내 캐싱](xref:performance/caching/memory)
* [분산 캐시 사용](xref:performance/caching/distributed)
* [변경 토큰을 사용하여 변경 내용 검색](xref:fundamentals/primitives/change-tokens)
* [응답 캐싱](xref:performance/caching/response)
* [캐시 태그 도우미](xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper)
* [분산 캐시 태그 도우미](xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper)
