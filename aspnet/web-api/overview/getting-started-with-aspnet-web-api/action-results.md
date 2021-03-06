---
uid: web-api/overview/getting-started-with-aspnet-web-api/action-results
title: "Web API 2의에서 동작으로 인해 | Microsoft Docs"
author: MikeWasson
description: 
ms.author: aspnetcontent
manager: wpickett
ms.date: 02/03/2014
ms.topic: article
ms.assetid: 2fc4797c-38ef-4cc7-926c-ca431c4739e8
ms.technology: dotnet-webapi
ms.prod: .net-framework
msc.legacyurl: /web-api/overview/getting-started-with-aspnet-web-api/action-results
msc.type: authoredcontent
ms.openlocfilehash: d0db5c6d45020861d7295ab1db989caee525fff9
ms.sourcegitcommit: 060879fcf3f73d2366b5c811986f8695fff65db8
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/24/2018
---
<a name="action-results-in-web-api-2"></a>Web API 2의에서 작업 결과
====================
으로 [Mike Wasson](https://github.com/MikeWasson)

이 항목에서는 ASP.NET Web API HTTP 응답 메시지에 컨트롤러 작업에서 반환 값을 변환 하는 방법에 대해 설명 합니다.

Web API 컨트롤러 작업에는 다음 중 하나를 반환할 수 있습니다.

1. void
2. **HttpResponseMessage**
3. **IHttpActionResult**
4. 다른 일부 형식

에 따라 다음 중이 반환, 웹 API는 HTTP 응답을 만드는 다른 메커니즘을 사용 합니다.

| 반환 형식 | 웹 API 응답을 만드는 방법 |
| --- | --- |
| void | 빈 204 (콘텐츠 없음)를 반환 합니다. |
| **HttpResponseMessage** | 직접 HTTP 응답 메시지를 변환 합니다. |
| **IHttpActionResult** | 호출 **ExecuteAsync** 만들려는 **HttpResponseMessage**, HTTP 응답 메시지를 변환 합니다. |
| 다른 유형 | Serialize 된 반환 값은 고 응답 본문에 쓰기 200 (정상)를 반환 합니다. |

이 항목의 나머지 부분에서 자세히 각 옵션에 설명 합니다.

## <a name="void"></a>void

반환 형식이 `void`, Web API에서 빈 HTTP 응답 상태 코드 204 (콘텐츠 없음)와 함께 반환 합니다.

예제에서는 컨트롤러:

[!code-csharp[Main](action-results/samples/sample1.cs)]

HTTP 응답:

[!code-console[Main](action-results/samples/sample2.cmd)]

## <a name="httpresponsemessage"></a>HttpResponseMessage

작업에서 반환 하는 경우는 [HttpResponseMessage](https://msdn.microsoft.com/library/system.net.http.httpresponsemessage.aspx), 웹 API 변환 반환 값은 HTTP 응답 메시지에 직접 속성을 사용 하는 **HttpResponseMessage** 을 채우기 위해 개체는 응답입니다.

이 옵션을 많이 응답 메시지에 대 한 제어를 제공합니다. 예를 들어 컨트롤러 동작 캐시 제어 헤더를 설정합니다.

[!code-csharp[Main](action-results/samples/sample3.cs)]

응답:

[!code-console[Main](action-results/samples/sample4.cmd?highlight=2)]

도메인 모델을 전달 하는 경우는 **CreateResponse** 메서드를 사용 하 여 웹 API는 [미디어 포맷터](../formats-and-model-binding/media-formatters.md) 응답 본문에 serialize 된 모델을 작성할 수 있습니다.

[!code-csharp[Main](action-results/samples/sample5.cs)]

Web API 요청의 Accept 헤더를 사용 하 여 포맷터를 선택 합니다. 자세한 내용은 참조 [콘텐츠 협상](../formats-and-model-binding/content-negotiation.md)합니다.

## <a name="ihttpactionresult"></a>IHttpActionResult

**IHttpActionResult** 인터페이스 Web API 2에서 도입 되었습니다. 정의 기본적으로 **HttpResponseMessage** 팩터리입니다. 여기의 일부의 이점은 사용 하 여 **IHttpActionResult** 인터페이스:

- 간소화 [단위 테스트](../testing-and-debugging/unit-testing-controllers-in-web-api.md) 컨트롤러에 있습니다.
- HTTP 응답을 별도 클래스를 만들기 위한 공통 논리를 이동 합니다.
- 응답 구성의 하위 수준 세부 정보를 숨겨 명료 컨트롤러 동작의 의도를 만듭니다.

**IHttpActionResult** 메서드 하나가 포함 **ExecuteAsync**, 비동기적으로 만듭니다는 **HttpResponseMessage** 인스턴스.

[!code-csharp[Main](action-results/samples/sample6.cs)]

컨트롤러 작업을 반환 하는 경우는 **IHttpActionResult**, 웹 API 호출는 **ExecuteAsync** 만드는 메서드를 한 **HttpResponseMessage**합니다. 변환 후의 **HttpResponseMessage** HTTP 응답 메시지에 있습니다.

다음의 간단한 한 않아도 됨은 **IHttpActionResult** 를 만드는 일반 텍스트 응답:

[!code-csharp[Main](action-results/samples/sample7.cs)]

예제에서는 컨트롤러 동작:

[!code-csharp[Main](action-results/samples/sample8.cs)]

응답:

[!code-console[Main](action-results/samples/sample9.cmd)]

사용할 더 자주는 **IHttpActionResult** 에 정의 된 구현은  **[System.Web.Http.Results](https://msdn.microsoft.com/library/system.web.http.results.aspx)**  네임 스페이스입니다. **ApiController** 클래스는 이러한 기본 제공 작업 결과 반환 하는 도우미 메서드를 정의 합니다.

다음 예제에서 요청은 기존 제품 ID와 일치 하지 않으면 컨트롤러 호출 [ApiController.NotFound](https://msdn.microsoft.com/library/system.web.http.apicontroller.notfound.aspx) 404 (찾을 수 없음) 응답을 만들려고 합니다. 그렇지 않으면 컨트롤러 호출 [ApiController.OK](https://msdn.microsoft.com/library/dn314591.aspx), 제품을 포함 하는 200 (OK) 응답을 만듭니다.

[!code-csharp[Main](action-results/samples/sample10.cs)]

## <a name="other-return-types"></a>다른 반환 형식

반환 다른 모든 형식에 대 한 웹 API에 사용 된 [미디어 포맷터](../formats-and-model-binding/media-formatters.md) 반환 값을 직렬화 하는 데 있습니다. Web API 응답 본문으로 직렬화 된 값을 씁니다. 응답 상태 코드는 200 (정상)입니다.

[!code-csharp[Main](action-results/samples/sample11.cs)]

이 방법의 단점은 예: 404 오류 코드에 직접 반환할 수 없습니다. Throw 할 수 있습니다는 **HttpResponseException** 오류 코드에 대 한 합니다. 자세한 내용은 참조 [ASP.NET Web API의 예외 처리](../error-handling/exception-handling.md)합니다.

Web API 요청의 Accept 헤더를 사용 하 여 포맷터를 선택 합니다. 자세한 내용은 참조 [콘텐츠 협상](../formats-and-model-binding/content-negotiation.md)합니다.

예제 요청

[!code-console[Main](action-results/samples/sample12.cmd)]

예제 응답:

[!code-console[Main](action-results/samples/sample13.cmd)]
