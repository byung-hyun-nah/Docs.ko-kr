---
title: "ASP.NET Core의 팩터리 기반 미들웨어 활성화"
author: guardrex
description: "ASP.NET Core에서 팩터리 기반 활성화 구현으로 강력한 형식의 미들웨어를 사용하는 방법을 알아봅니다."
ms.author: riande
manager: wpickett
ms.custom: mvc
ms.date: 01/29/2018
ms.topic: article
ms.technology: aspnet
ms.prod: asp.net-core
uid: fundamentals/middleware/extensibility
ms.openlocfilehash: 57ff9db2edbf307f2442443dc14e69b0498f7475
ms.sourcegitcommit: f2a11a89037471a77ad68a67533754b7bb8303e2
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/01/2018
---
# <a name="factory-based-middleware-activation-in-aspnet-core"></a>ASP.NET Core의 팩터리 기반 미들웨어 활성화

[Luke Latham](https://github.com/guardrex)으로

[IMiddlewareFactory](/dotnet/api/microsoft.aspnetcore.http.imiddlewarefactory)/[IMiddleware](/dotnet/api/microsoft.aspnetcore.http.imiddleware)는 [미들웨어](xref:fundamentals/middleware/index) 활성화에 대한 확장성 지점입니다.

`UseMiddleware` 확장 메서드는 미들웨어의 등록된 형식이 `IMiddleware`를 구현하는지 확인합니다. 구현하는 경우 컨테이너에 등록된 `IMiddlewareFactory` 인스턴스는 규칙 기반 미들웨어 활성화 논리를 사용하는 대신 `IMiddleware` 구현을 확인하는 데 사용됩니다. 미들웨어는 앱의 서비스 컨테이너의 범위가 지정된 서비스 또는 일시적인 서비스로 등록됩니다.

혜택:

* 요청당 활성화(범위가 지정된 서비스의 주입)
* 미들웨어의 강력한 형식 지정

`IMiddleware`는 요청당 활성화되므로 범위가 지정된 서비스는 미들웨어의 생성자에 주입될 수 있습니다.

[샘플 코드 보기 또는 다운로드](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/middleware/extensibility/sample)([다운로드 방법](xref:tutorials/index#how-to-download-a-sample))

샘플 앱은 다음에 의해 활성화되는 미들웨어를 보여 줍니다.

* 규칙(`ConventionalMiddleware`). 규칙에 따른 미들웨어 활성화에 대한 자세한 내용은 [미들웨어](xref:fundamentals/middleware/index) 항목을 참조하세요.
* [IMiddlewareFactory](/dotnet/api/microsoft.aspnetcore.http.imiddlewarefactory) 구현(`IMiddlewareMiddleware`). 기본 [MiddlewareFactory 클래스](/dotnet/api/microsoft.aspnetcore.http.middlewarefactory)가 미들웨어를 활성화합니다.

미들웨어 구현은 동일하게 작동하며, 쿼리 문자열 매개 변수(`key`)에서 제공한 값을 기록합니다. 미들웨어는 주입된 데이터베이스 컨텍스트(범위가 지정된 서비스)를 사용하여 메모리 내 데이터베이스에 쿼리 문자열 값을 기록합니다.

## <a name="imiddleware"></a>IMiddleware

[IMiddleware](/dotnet/api/microsoft.aspnetcore.http.imiddleware)는 앱의 요청 파이프라인에 대한 미들웨어를 정의합니다. [InvokeAsync(HttpContext, RequestDelegate)](/dotnet/api/microsoft.aspnetcore.http.imiddleware.invokeasync#Microsoft_AspNetCore_Http_IMiddleware_InvokeAsync_Microsoft_AspNetCore_Http_HttpContext_Microsoft_AspNetCore_Http_RequestDelegate_) 메서드는 요청을 처리하고 미들웨어의 실행을 나타내는 `Task`를 반환합니다.

규칙에 따라 미들웨어 활성화:

[!code-csharp[Main](extensibility/sample/Middleware/ConventionalMiddleware.cs?name=snippet1)]

`MiddlewareFactory`에 따라 미들웨어 활성화:

[!code-csharp[Main](extensibility/sample/Middleware/IMiddlewareMiddleware.cs?name=snippet1)]

확장은 미들웨어에 대해 생성됩니다.

[!code-csharp[Main](extensibility/sample/Middleware/MiddlewareExtensions.cs?name=snippet1)]

개체를 `UseMiddleware`를 사용하여 팩터리 활성화된 미들웨어에 전달할 수 없습니다.

```csharp
public static IApplicationBuilder UseIMiddlewareMiddleware(
    this IApplicationBuilder builder, bool option)
{
    // Passing 'option' as an argument throws a NotSupportedException at runtime.
    return builder.UseMiddleware<IMiddlewareMiddleware>(option);
}
```

팩터리 활성화된 미들웨어는 *Startup.cs*의 기본 제공 컨테이너에 추가됩니다.

[!code-csharp[Main](extensibility/sample/Startup.cs?name=snippet1&highlight=6)]

두 미들웨어는 모두 `Configure`의 요청 처리 파이프라인에 등록됩니다.

[!code-csharp[Main](extensibility/sample/Startup.cs?name=snippet2&highlight=12-13)]

## <a name="imiddlewarefactory"></a>IMiddlewareFactory

[IMiddlewareFactory](/dotnet/api/microsoft.aspnetcore.http.imiddlewarefactory)는 미들웨어를 만드는 메서드를 제공합니다. 미들웨어 팩터리 구현은 범위가 지정된 서비스로 컨테이너에 등록됩니다.

기본 `IMiddlewareFactory` 구현인 [MiddlewareFactory](/dotnet/api/microsoft.aspnetcore.http.middlewarefactory)는 [Microsoft.AspNetCore.Http](https://www.nuget.org/packages/Microsoft.AspNetCore.Http/) 패키지에 있습니다([참조 소스](https://github.com/aspnet/HttpAbstractions/blob/release/2.0/src/Microsoft.AspNetCore.Http/MiddlewareFactory.cs)).

## <a name="additional-resources"></a>추가 리소스

* [미들웨어](xref:fundamentals/middleware/index)
