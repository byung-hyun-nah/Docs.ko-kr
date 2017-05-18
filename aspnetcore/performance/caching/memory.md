---
title: "ASP.NET Core에서 메모리 내 캐싱을 | Microsoft 문서"
author: rick-anderson
description: "ASP.NET Core의 메모리에 데이터를 캐시 하는 방법을 보여 줍니다."
keywords: "ASP.NET Core, 캐시 메모리에 성능"
ms.author: riande
manager: wpickett
ms.date: 12/14/2016
ms.topic: article
ms.assetid: 819511cf-d33e-410a-b5a9-bef7fa64d2f3
ms.technology: aspnet
ms.prod: asp.net-core
uid: performance/caching/memory
ms.custom: H1Hack27Feb2017
translationtype: Machine Translation
ms.sourcegitcommit: 010b730d2716f9f536fef889bc2f767afb648ef4
ms.openlocfilehash: 5297184f5d6f2849dc5dafa55edeba2b791f7045
ms.lasthandoff: 03/23/2017

---
# <a name="introduction-to-in-memory-caching-in-aspnet-core"></a>ASP.NET Core에서 메모리 내 캐싱을 소개

여 [Rick Anderson](https://twitter.com/RickAndMSFT), [John Luo](https://github.com/JunTaoLuo), 및 [Steve Smith](http://ardalis.com)

[샘플 코드 보기 또는 다운로드](https://github.com/aspnet/Docs/tree/master/aspnetcore/performance/caching/memory/sample)

## <a name="caching-basics"></a>캐싱 기본 사항

캐싱 크게 향상 시킬 수 성능 및 확장성 응용 프로그램의 콘텐츠를 생성 하는 데 필요한 작업을 줄여 합니다. 캐싱 동작을 가장 효율적으로 데이터를 자주 변경 되지 않습니다. 많은 반환 될 수 있는 데이터의 복사본을 만들어 캐싱 원본에서 보다 더 빠릅니다. 작성 하 고 캐시 된 데이터에 종속 되지 응용 프로그램을 테스트 해야 합니다.

ASP.NET Core 몇 가지 서로 다른 캐시를 지원합니다. 가장 간단한 캐시 기반는 [IMemoryCache](https://docs.microsoft.com/en-us/aspnet/core/api/microsoft.extensions.caching.memory.imemorycache)을 웹 서버의 메모리에 저장 된 캐시를 나타냅니다. 서버 팜에 여러 서버에서 실행 하는 앱은 메모리 내 캐시를 사용 하는 경우 세션은 고정 확인 해야 합니다. 고정 세션 동일한 서버로 이동 모든 클라이언트의 후속 요청을 확인 합니다. 예를 들어 Azure 웹 앱 사용 [응용 프로그램 요청 라우팅](http://www.iis.net/learn/extensions/planning-for-arr) (ARR) 동일한 서버에 모든 후속 요청을 라우팅할 수 있습니다.

웹 팜에서 아닌 고정 세션 필요는 [분산 캐시](distributed.md) 캐시 일관성 문제가 발생 하지 않도록 합니다. 일부 응용 프로그램에 대 한 분산된 캐시는 메모리 내 캐시 보다 더 높은 규모 확장을 지원할 수 있습니다. 분산된 캐시를 사용 하 여 캐시 메모리 외부 프로세스에 오프 로드 합니다. 

`IMemoryCache` 하지 않는 한 캐시 메모리가 부족 한 캐시 엔트리를 제거 하는 [우선 순위를 캐시](https://docs.microsoft.com/en-us/aspnet/core/api/microsoft.extensions.caching.memory.cacheitempriority) 로 설정 된 `CacheItemPriority.NeverRemove`합니다. 설정할 수는 `CacheItemPriority` 캐시를 우선 순위를 조정 하려면 메모리가 중에서 항목을 제거 합니다.

메모리 내 캐시는 모든 개체를 저장할 수 있습니다. 분산된 캐시 인터페이스는 제한 `byte[]`합니다.

## <a name="using-imemorycache"></a>IMemoryCache를 사용 하 여

메모리 내 캐싱은 *서비스* 사용 하 여 응용 프로그램에서 참조 하는 [종속성 주입](../../fundamentals/dependency-injection.md)합니다. Call `AddMemoryCache` in `ConfigureServices`:

[!code-csharp[주](memory/sample/WebCache/Startup.cs?highlight=8)] 

요청 된 `IMemoryCache` 생성자에서 인스턴스:

[!code-csharp[주](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_ctor&highlight=3,5-)] 

`IMemoryCache`NuGet 패키지 "Microsoft.Extensions.Caching.Memory" 필요합니다.

다음 코드에서는 [TryGetValue](https://docs.microsoft.com/en-us/aspnet/core/api/microsoft.extensions.caching.memory.imemorycache#Microsoft_Extensions_Caching_Memory_IMemoryCache_TryGetValue_System_Object_System_Object__) 현재 캐시에 있는지 확인 하려면. 새 항목이 생성 되어 캐시에 추가 항목이 캐시 되지 않으면 [설정](https://docs.microsoft.com/en-us/aspnet/core/api/microsoft.extensions.caching.memory.cacheextensions#Microsoft_Extensions_Caching_Memory_CacheExtensions_Set__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object___0_)합니다.

[!code-csharp[주](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet1)]

현재 시간 및 캐시 된 시간이 표시 됩니다.

[!code-html[주](memory/sample/WebCache/Views/Home/Cache.cshtml)]

캐시 된 `DateTime` 요청 제한 시간 (및 메모리 부족으로 인해 없는 제거)에 하는 동안 값이 캐시에 유지 됩니다. 아래 이미지는 현재 시간 및 캐시에서 검색 하는 이전 시간을 보여 줍니다.

![인덱스 뷰를 표시 하는 두 개의 서로 다른 시간](memory/_static/time.png)

다음 코드에서는 [GetOrCreate](https://docs.microsoft.com/en-us/aspnet/core/api/microsoft.extensions.caching.memory.cacheextensions#Microsoft_Extensions_Caching_Memory_CacheExtensions_GetOrCreate__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_System_Func_Microsoft_Extensions_Caching_Memory_ICacheEntry___0__) 및 [GetOrCreateAsync](https://docs.microsoft.com/en-us/aspnet/core/api/microsoft.extensions.caching.memory.cacheextensions#Microsoft_Extensions_Caching_Memory_CacheExtensions_GetOrCreateAsync__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_System_Func_Microsoft_Extensions_Caching_Memory_ICacheEntry_System_Threading_Tasks_Task___0___) 데이터를 캐시 합니다. 

[!code-csharp[주](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet2&highlight=3-7,14-19)]

다음 호출 코드 [가져오기](https://docs.microsoft.com/en-us/aspnet/core/api/microsoft.extensions.caching.memory.cacheextensions#Microsoft_Extensions_Caching_Memory_CacheExtensions_Get__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_) 캐시 된 시간을 인출할 수 있습니다.

[!code-csharp[주](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_gct)]

참조 [IMemoryCache 메서드](https://docs.microsoft.com/en-us/aspnet/core/api/microsoft.extensions.caching.memory.imemorycache) 및 [CacheExtensions 메서드](https://docs.microsoft.com/en-us/aspnet/core/api/microsoft.extensions.caching.memory.cacheextensions) 에 대 한 설명은 캐시 메서드.

## <a name="using-memorycacheentryoptions"></a>MemoryCacheEntryOptions를 사용 하 여

다음 샘플:

- 절대 만료 시간을 설정합니다. 이 항목을 캐시 될 수 있는 최대 시간 되며 항목 슬라이딩 만료 지속적으로 갱신 될 때 너무 부실 하지 못하도록 합니다.
- 슬라이딩 만료 시간을 설정합니다. 이 캐시 된 항목에 액세스 하는 요청에는 슬라이딩 만료 시계가 다시 설정 됩니다.
- 캐시 우선 순위 설정 `CacheItemPriority.NeverRemove`합니다. 
- 집합은 [PostEvictionDelegate](https://docs.microsoft.com/en-us/aspnet/core/api/microsoft.extensions.caching.memory.postevictiondelegate) 있는 후에 호출 됩니다 항목이 캐시에서 제거 됩니다. 콜백 캐시에서 항목을 제거 하는 코드에서 다른 스레드에서 실행 됩니다.

[!code-csharp[주](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_et&highlight=14-20)]

## <a name="cache-dependencies"></a>캐시 종속성

다음 샘플에는 종속 항목 만료 되는 경우 캐시 엔트리를 만료 하는 방법을 보여 줍니다. A `CancellationChangeToken` 캐시 된 항목에 추가 됩니다. 때 `Cancel` 라고 하는 `CancellationTokenSource`, 캐시 항목이 모두 제거 됩니다. 

[!code-csharp[주](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_ed)]

사용 하는 `CancellationTokenSource` 여러 캐시 항목이 그룹으로 제거할 수 있습니다. 와 `using` 내에 만들어진 위 코드에서 패턴, 캐시 항목의 `using` 블록 트리거 및 만료 설정을 상속 합니다.

### <a name="addition-notes"></a>추가 정보

- 콜백을 사용 하 여 캐시 항목을 다시 채우려면 하는 경우:

  - 여러 요청 되므로 찾을 수 있는 캐시 된 키 값 빈 콜백이 완료 되지 않은. 
  - 이 캐시 된 항목을 채우면 여러 스레드에서 발생할 수 있습니다.

- 하나의 캐시 항목을 사용 하 여 다른 만들을 자식 부모 항목의 만료 토큰 및 시간 기반 만료 설정을 복사 합니다. 자식 없거나 수동으로 제거 하 여 만료 된 부모 항목의 업데이트입니다.

### <a name="other-resources"></a>기타 리소스

* [분산된 캐시 작업](distributed.md)
* [응답의 캐싱 미들웨어](middleware.md)
