---
title: "ASP.NET Core에서 HTTP.sys 웹 서버 구현"
author: rick-anderson
description: "Windows의 ASP.NET Core에 대한 웹 서버인, HTTP.sys를 소개합니다. Http.Sys 커널 모드 드라이버를 기반으로 한 HTTP.sys는 IIS 없이 인터넷에 대한 직접 연결에 사용될 수 있는 Kestrel에 대한 대안입니다."
manager: wpickett
ms.author: riande
ms.date: 08/07/2017
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: fundamentals/servers/httpsys
ms.openlocfilehash: f36a86fc67e715165afd0c38992f9767f90cf3b4
ms.sourcegitcommit: a510f38930abc84c4b302029d019a34dfe76823b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/30/2018
---
# <a name="httpsys-web-server-implementation-in-aspnet-core"></a>ASP.NET Core에서 HTTP.sys 웹 서버 구현

작성자: [Tom Dykstra](https://github.com/tdykstra) 및 [Chris Ross](https://github.com/Tratcher)

> [!NOTE]
> 이 주제는 ASP.NET Core 2.0 이상에만 적용됩니다. 이전 버전의 ASP.NET Core에서 HTTP.sys는 [WebListener](xref:fundamentals/servers/weblistener)라고 합니다.

HTTP.sys는 Windows에서만 실행되는 [ASP.NET Core에 대한 웹 서버](index.md)입니다. [Http.Sys 커널 모드 드라이버](https://msdn.microsoft.com/library/windows/desktop/aa364510.aspx)를 기반으로 합니다. HTTP.sys는 Kestel이 하지 않는 일부 기능을 제공하는 [Kestrel](kestrel.md)에 대한 대안입니다. **HTTP.sys는 [ASP.NET Core 모듈](aspnet-core-module.md)과 호환되지 않으므로 IIS 또는 IIS Express와 함께 사용될 수 없습니다.**

HTTP.sys는 다음과 같은 기능을 지원합니다.

- [Windows 인증](xref:security/authentication/windowsauth)
- 포트 공유
- SNI를 사용하는 HTTPS
- TLS를 통한 HTTP/2(Windows 10)
- 직접 파일 전송
- 응답 캐싱
- WebSockets(Windows 8)

지원되는 Windows 버전:

- Windows 7 및 Windows Server 2008 R2 이상

[샘플 코드 보기 또는 다운로드](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/servers/httpsys/sample)([다운로드 방법](xref:tutorials/index#how-to-download-a-sample))

## <a name="when-to-use-httpsys"></a>HTTP.sys를 사용하는 경우

HTTP.sys는 IIS를 사용하지 않고 인터넷에 서버를 직접 노출해야 하는 배포에 유용합니다.

![HTTP.sys는 인터넷과 직접 통신합니다.](httpsys/_static/httpsys-to-internet.png)

Http.Sys를 기반으로 하기 때문에 HTTP.sys는 공격으로부터 보호하기 위한 역방향 프록시 서버가 필요하지 않습니다. Http.Sys는 많은 유형의 공격으로부터 보호하고 모든 기능을 갖춘 웹 서버의 견고성, 보안 및 확장성을 제공하는 완성도 높은 기술입니다. IIS 자체는 Http.Sys 위에 HTTP 수신기로 실행됩니다. 

HTTP.sys는 Windows 인증과 같이 Kestrel에서 사용할 수 없는 기능이 필요한 경우 내부 배포에 적합한 선택입니다.

![HTTP.sys는 내부 네트워크와 직접 통신합니다.](httpsys/_static/httpsys-to-internal.png)

## <a name="how-to-use-httpsys"></a>HTTP.sys 사용 방법

호스트 OS 및 사용자의 ASP.NET Core 응용 프로그램에 대한 설치 작업의 개요입니다.

### <a name="configure-windows-server"></a>Windows Server 구성

* [.NET Core](https://www.microsoft.com/net/download/core) 또는 [.NET Framework](https://www.microsoft.com/net/download/framework)와 같은 응용 프로그램에 필요한 .NET 버전을 설치합니다.

* HTTP.sys에 대해 바인딩하고 SSL 인증서를 설정하도록 URL 접두사 미리 등록

   Windows에서 URL 접두사를 미리 등록하지 않은 경우 관리자 권한으로 응용 프로그램을 실행해야 합니다. 유일한 예외는 포트 번호가 1024보다 큰 HTTP(HTTPS 아님)를 사용하여 로컬 호스트에 바인딩하는 경우로, 이 경우 관리자 권한은 필요하지 않습니다.

   자세한 내용은 이 문서의 뒷부분에 나오는 [접두사를 미리 등록하고 SSL을 구성하는 방법](#preregister-url-prefixes-and-configure-ssl)을 참조하세요.

* 방화벽 포트를 열어 HTTP.sys에 도달하는 트래픽을 허용합니다.

   *netsh.exe* 또는 [PowerShell cmdlet](https://technet.microsoft.com/library/jj554906)을 사용할 수 있습니다.

[Http.Sys 레지스트리 설정](https://support.microsoft.com/kb/820129)도 있습니다.

### <a name="configure-your-aspnet-core-application-to-use-httpsys"></a>HTTP.sys를 사용하도록 ASP.NET Core 응용 프로그램 구성

* [Microsoft.AspNetCore.All](xref:fundamentals/metapackage) 메타패키지를 사용하는 경우에는 패키지를 설치할 필요가 없습니다. [Microsoft.AspNetCore.Server.HttpSys](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.HttpSys/) 패키지는 메타패키지에 포함되어 있습니다.

* 다음 예제와 같이 필요한 [HTTP.sys 옵션](https://github.com/aspnet/HttpSysServer/blob/rel/2.0.0/src/Microsoft.AspNetCore.Server.HttpSys/HttpSysOptions.cs)을 지정하여 `Main` 메서드에서 `WebHostBuilder`의 `UseHttpSys` 확장 메서드를 호출합니다.

  [!code-csharp[](httpsys/sample/Program.cs?name=snippet_Main&highlight=11-19)]

### <a name="configure-httpsys-options"></a>HTTP.sys 옵션 구성

다음은 몇 가지 구성 가능한 HTTP.sys 설정 및 제한입니다.

**최대 클라이언트 연결**

*Program.cs*의 다음 코드를 사용하여 전체 응용 프로그램에 대한 동시 개방 TCP 연결의 최대 수를 설정할 수 있습니다.

[!code-csharp[](httpsys/sample/Program.cs?name=snippet_Options&highlight=5)]

연결의 최대 수는 기본적으로 무제한(null)입니다.

**최대 요청 본문 크기**

기본 최대 요청 본문 크기는 약 28.6MB인 30,000,000바이트입니다.

ASP.NET Core MVC 앱에서 한도를 재정의할 때는 작업 메서드에서 [RequestSizeLimit](https://github.com/aspnet/Mvc/blob/rel/2.0.0/src/Microsoft.AspNetCore.Mvc.Core/RequestSizeLimitAttribute.cs) 특성을 사용하는 방법이 좋습니다.

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

다음 예제는 전체 응용 프로그램, 모든 요청에 대한 제약 조건을 구성하는 방법을 보여 줍니다.

[!code-csharp[](httpsys/sample/Program.cs?name=snippet_Options&highlight=6)]

*Startup.cs*에서 특정 요청에 대한 설정을 재정의할 수 있습니다.

[!code-csharp[](httpsys/sample/Startup.cs?name=snippet_Configure&highlight=9-10)]
 
응용 프로그램에서 요청을 읽기 시작한 후 요청에 대한 제한을 구성하려고 하면 예외가 throw됩니다. `MaxRequestBodySize` 속성이 제한을 구성하기에 너무 늦은, 읽기 전용 상태인지를 알려주는 `IsReadOnly` 속성이 있습니다.

다른 HTTP.sys 옵션에 대한 정보는 [HttpSysOptions](https://github.com/aspnet/HttpSysServer/blob/rel/2.0.0/src/Microsoft.AspNetCore.Server.HttpSys/HttpSysOptions.cs)를 참조하세요. 

### <a name="configure-urls-and-ports-to-listen-on"></a>수신 대기하는 URL 및 포트 구성 

기본적으로 ASP.NET Core는 `http://localhost:5000`으로 바인딩합니다. URL 접두사 및 포트를 구성하려면 [HttpSysOptions](https://github.com/aspnet/HttpSysServer/blob/rel/2.0.0/src/Microsoft.AspNetCore.Server.HttpSys/HttpSysOptions.cs)에서 `UseUrls` 확장 메서드, `urls` 명령줄 인수, ASPNETCORE_URLS 환경 변수 또는 `UrlPrefixes` 속성을 사용할 수 있습니다. 다음 코드 예제에서는 `UrlPrefixes`를 사용합니다.

[!code-csharp[](httpsys/sample/Program.cs?name=snippet_Main&highlight=17)]

`UrlPrefixes`의 이점은 서식이 잘못 지정된 접두사를 추가하려고 하면 오류 메시지가 즉시 표시된다는 점입니다. `UseUrls`(`urls` 및 ASPNETCORE_URLS과 공유)의 이점은 더 쉽게 Kestrel 및 HTTP.sys 간 전환이 가능하다는 점입니다.

`UseUrls`(또는 `urls` 또는 ASPNETCORE_URLS) 및 `UrlPrefixes`를 모두 사용하는 경우 `UrlPrefixes`의 설정은 `UseUrls`의 설정을 재정의합니다. 자세한 내용은 [호스팅](xref:fundamentals/hosting)을 참조하세요.

HTTP.sys는 [HTTP Server API UrlPrefix 문자열 형식](https://msdn.microsoft.com/library/windows/desktop/aa364698.aspx)을 사용합니다.

> [!NOTE]
> 서버에서 미리 등록한 동일한 접두사 문자열을 `UseUrls` 또는 `UrlPrefixes`에서 지정했는지 확인합니다. 

### <a name="dont-use-iis"></a>IIS 사용 안 함

응용 프로그램이 IIS 또는 IIS Express를 실행하도록 구성되지 않았는지 확인합니다.

Visual Studio에서 기본 실행 프로필은 IIS Express용입니다. 프로젝트를 콘솔 응용 프로그램으로 실행하려면 다음 스크린샷에 표시된 것처럼 선택한 프로필을 수동으로 변경합니다.

![콘솔 앱 프로필 선택](httpsys/_static/vs-choose-profile.png)

## <a name="preregister-url-prefixes-and-configure-ssl"></a>URL 접두사 미리 등록 및 SSL 구성

IIS와 HTTP.sys는 모두 기본 Http.Sys 커널 모드 드라이버를 사용하여 요청을 수신하고 초기 처리를 수행합니다. IIS에서 관리 UI는 모든 항목을 구성하는 상대적으로 쉬운 방법을 제공합니다. 그러나 Http.Sys는 직접 구성해야 합니다. 작업 수행을 위한 기본 도구는 *netsh.exe*입니다. 

*netsh.exe*를 사용하면 URL 접두사를 예약하고 SSL 인증서를 할당할 수 있습니다. 도구를 사용하려면 관리 권한이 있어야 합니다.

다음 예제에서는 포트 80 및 443에 대해 URL 접두사를 예약하는 데 필요한 최소한의 것을 보여 줍니다.

```console
netsh http add urlacl url=http://+:80/ user=Users
netsh http add urlacl url=https://+:443/ user=Users
```

다음 예제에서는 SSL 인증서를 할당하는 방법을 보여 줍니다.

```console
netsh http add sslcert ipport=0.0.0.0:443 certhash=MyCertHash_Here appid={00000000-0000-0000-0000-000000000000}"
```

다음은 *netsh.exe*에 대한 참조 문서입니다.

* [HTTP(Hypertext Transfer Protocol)에 대한 Netsh 명령](https://technet.microsoft.com/library/cc725882.aspx)
* [UrlPrefix 문자열](https://msdn.microsoft.com/library/windows/desktop/aa364698.aspx)

다음 리소스는 여러 시나리오에 대한 자세한 지침을 제공합니다. HttpListener를 참조하는 문서는 모두 Http.Sys를 기반으로 하므로 HTTP.sys에 동일하게 적용됩니다.

* [방법: SSL 인증서로 포트 구성](https://docs.microsoft.com/dotnet/framework/wcf/feature-details/how-to-configure-a-port-with-an-ssl-certificate)
* [HTTPS 통신 - HttpListener 기반 호스팅 및 클라이언트 인증](http://sunshaking.blogspot.com/2012/11/https-communication-httplistener-based.html) 이는 타사 블로그이며 비교적 오래됐지만 여전히 유용한 정보가 있습니다.
* [방법: SSL 단순 서버로 HttpListener 또는 Http 서버 비관리 코드(C++)를 사용한 연습](https://blogs.msdn.microsoft.com/jpsanders/2009/09/29/how-to-walkthrough-using-httplistener-or-http-server-unmanaged-code-c-as-an-ssl-simple-server/) 유용한 정보가 있는 오래된 블로그입니다.

다음은 *netsh.exe* 명령줄보다 쉽게 사용할 수 있는 몇 가지 타사 도구입니다. 이러한 도구는 Microsoft에서 제공되거나 보증되지 않습니다. *netsh.exe* 자체는 관리자 권한이 필요하므로 도구는 기본적으로 관리자 권한으로 실행됩니다.

* [http.sys 관리자](http://httpsysmanager.codeplex.com/)는 SSL 인증서 및 옵션, 접두사 예약 및 인증서 신뢰 목록의 나열 및 구성을 위한 UI를 제공합니다. 
* [HttpConfig](http://www.stevestechspot.com/ABetterHttpcfg.aspx)를 통해 SSL 인증서와 URL 접두사를 나열하거나 구성할 수 있습니다. UI는 http.sys 관리자보다 성능이 우수하며 몇 가지 추가 구성 옵션을 노출하지만 그렇지 않은 경우 유사한 기능을 제공합니다. 새 CTL(인증서 신뢰 목록)을 만들 수 없지만 기존 것을 할당할 수 있습니다.

[!INCLUDE[How to make an SSL cert](../../includes/make-ssl-cert.md)]

## <a name="next-steps"></a>다음 단계

자세한 내용은 다음 리소스를 참조하세요.

* [이 문서에 대한 샘플 앱](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/servers/httpsys/sample)
* [HTTP.sys 소스 코드](https://github.com/aspnet/HttpSysServer/)
* [호스팅](xref:fundamentals/hosting)
