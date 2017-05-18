---
title: "전역화 및 지역화 | Microsoft 문서"
author: rick-anderson
description: 
keywords: ASP.NET Core
ms.author: riande
manager: wpickett
ms.date: 01/14/2017
ms.topic: article
ms.assetid: 7f275a09-f118-41c9-88d1-8de52d6a5aa1
ms.technology: aspnet
ms.prod: asp.net-core
uid: fundamentals/localization
translationtype: Machine Translation
ms.sourcegitcommit: 1894789727a7c3ae66f25040d1a70c4f1e64c3da
ms.openlocfilehash: 403797184ce3c08066e350aec523bb1ddc4ab176
ms.lasthandoff: 03/23/2017

---
# <a name="globalization-and-localization"></a>전역화 및 지역화

여 [Rick Anderson](https://twitter.com/RickAndMSFT), [Damien Bowden](https://twitter.com/damien_bod), [구재석 Calixto](https://twitter.com/bartmax), [Nadeem Afana](https://twitter.com/NadeemAfana), 및 [Hisham Bin Ateya](https://twitter.com/hishambinateya)

ASP.NET Core를 사용 하 여 다국어 웹 사이트를 만들기 하면 더 광범위 한 도달 하 여 사이트. ASP.NET Core 다른 언어와 문화권으로 지역화를 위한 미들웨어 및 서비스를 제공 합니다.

국제화 관련 [세계화](https://msdn.microsoft.com/en-us/library/aa292081(v=vs.71).aspx) 및 [지역화](https://msdn.microsoft.com/en-us/library/aa292137(v=vs.71).aspx)합니다. 세계화는 서로 다른 culture를 지 원하는 앱을 디자인 하는 과정입니다. 전역화에는 입력, 표시 및 정의 된 특정 지역과 관련 된 언어 스크립트 집합의 출력에 대 한 지원을 추가 합니다.

지역화는 지역화 가능성 특정 문화권/로캘을를 이미 처리 세계화 된 응용 프로그램을 못지 않게 과정.  자세한 내용은 참조 **전역화 및 지역화 용어** 이 문서의 끝 부분입니다.

응용 프로그램 지역화에는 다음이 포함 됩니다.

1. 응용 프로그램의 콘텐츠를 지역화할 수 있도록 만들기

2. 언어와 문화 지원에 대 한 지역화 된 리소스를 제공 합니다.

3. 각 요청에 대 한 언어/문화권을 선택 하는 전략을 구현 합니다.

## <a name="make-the-apps-content-localizable"></a>응용 프로그램의 콘텐츠를 지역화할 수 있도록 만들기

ASP.NET Core에 도입 된 `IStringLocalizer` 및 `IStringLocalizer<T>` 지역화 된 응용 프로그램을 개발할 때 생산성을 개선 하기 위해 설계 되었습니다. `IStringLocalizer`사용 하 여는 [ResourceManager](https://msdn.microsoft.com/en-us/library/system.resources.resourcemanager(v=vs.110).aspx) 및 [ResourceReader](https://msdn.microsoft.com/en-us/library/system.resources.resourcereader(v=vs.110).aspx) 런타임 시 문화권별 리소스를 제공 합니다. 간단한 인터페이스에는 인덱서 및 `IEnumerable` 지역화 된 문자열을 반환 합니다. `IStringLocalizer`리소스 파일에는 기본 언어 문자열을 저장 하도록 요구 하지 않습니다. 지역화를 위한 대상 응용 프로그램을 개발할 수 있으며 개발 초기 단계에서는 리소스 파일을 만들 필요가 없을 수 있습니다. 아래 코드에는 지역화를 위해 "에 대 한 제목" 문자열을 래핑하는 방법을 보여 줍니다.

[!code-csharp[주](localization/sample/Controllers/AboutController.cs)]

위의 코드에는 `IStringLocalizer<T>` 구현에서 가져온 [종속성 주입](dependency-injection.md)합니다. "에 대 한 제목"의 지역화 된 값이 없을 경우 인덱서 키 반환 되 면, 즉 "에 대 한 제목" 문자열입니다. 기본값을 그대로 둡니다 언어 리터럴 문자열 앱 수 있으며 앱 개발에 집중할 수 있도록은 지역화 담당자가에 래핑할 수 있습니다. 기본 언어를 사용 하 여 응용 프로그램을 개발 하 고이 정보를 먼저 기본 리소스 파일을 만들지 않고 지역화 단계에 대 한 준비 합니다. 또는 기존의 접근 방식을 사용 하 고 기본 언어 문자열을 검색 하는 키를 제공 수 있습니다. 대부분의 개발자의 기본 언어를가지고 있지 않은 경우 새 워크플로 *.resx* 파일 및 문자열 리터럴에 줄 바꿈 단순히 응용 프로그램을 지역화 하는 오버 헤드를 줄일 수 있습니다. 다른 개발자를 더 긴 문자열 리터럴이 작업 하 고 쉽게 지역화 된 문자열을 업데이트 하는 보다 쉽게 만들 수 것 처럼 기존의 작업 흐름을 선호 합니다.

사용 된 `IHtmlLocalizer<T>` HTML을 포함 하는 리소스에 대 한 구현 합니다. `IHtmlLocalizer`HTML은 리소스 문자열이 아니라 하지만 리소스 문자열에 서식이 지정 되는 인수를 인코딩합니다. 이 샘플에서 아래에 강조 표시의 값만 `name` 매개 변수는 HTML로 인코딩하지 합니다.

[!code-csharp[주](../fundamentals/localization/sample/Controllers/BookController.cs?highlight=3,5,20&start=1&end=24)]

> [!NOTE]
> 일반적으로 텍스트 및 HTML이 아닌만 지역화 하려고 합니다.

가장 낮은 수준에서 얻을 수 있습니다 `IStringLocalizerFactory` 부재 중 [종속성 주입](dependency-injection.md):

[!code-csharp[주](localization/sample/Controllers/TestController.cs?start=9&end=26&highlight=7-13)]

위의 코드를 보여 줍니다 두 팩터리의 각 메서드를 만듭니다.

컨트롤러, 영역, 지역화 된 문자열을 분할 하거나 하나의 컨테이너를 수 있습니다. 샘플 응용 프로그램에서 더미 클래스 라는 `SharedResource` 공유 리소스에 사용 됩니다.

[!code-csharp[주](localization/sample/Resources/SharedResource.cs)]

다음을 사용 하 여 일부 개발자는 `Startup` 전역 또는 공유 문자열을 포함 하는 클래스입니다.  아래 샘플에는 `InfoController` 및 `SharedResource` 지역화 담당자가 사용 됩니다.

[!code-csharp[주](localization/sample/Controllers/InfoController.cs?range=9-26)]

## <a name="view-localization"></a>보기 지역화

`IViewLocalizer` 서비스 제공에 대 한 지역화 된 문자열을 [보기](http://docs.asp.net/projects/mvc/en/latest/views/index.html)합니다. `ViewLocalizer` 클래스는이 인터페이스를 구현 하 고 보기 파일 경로에서 리소스 위치를 찾습니다. 다음 코드의 기본 구현을 사용 하는 방법을 보여 줍니다 `IViewLocalizer`:

[!code-HTML[주](localization/sample/Views/Home/About.cshtml)]

기본 구현은 `IViewLocalizer` 보기의 파일 이름에 따라 리소스 파일을 찾습니다. 공유 전역 리소스 파일을 사용할 수 있는 옵션이 있습니다. `ViewLocalizer`사용 하 여 지역화 담당자 구현 `IHtmlLocalizer`Razor HTML 하지 않으므로 지역화 된 문자열을 인코딩합니다. 리소스 문자열을 매개 변수화 할 수 및 `IViewLocalizer` 는 HTML 인코딩 매개 변수를 하지만 리소스 문자열이 아닙니다. Razor에 다음 태그를 고려해 야 합니다.

```HTML
@Localizer["<i>Hello</i> <b>{0}!</b>", UserManager.GetUserName(User)]
   ```

프랑스어 리소스 파일에 다음 포함 될 수 있습니다.

| 키 | 값 |
| ----- | ------ |
| `<i>Hello</i> <b>{0}!</b>` | `<i>Bonjour</i> <b>{0}!</b>` |

렌더링된 된 뷰는 HTML 태그는 리소스 파일에 포함 됩니다.

> [!NOTE]
> 일반적으로 텍스트 및 HTML이 아닌만 지역화 하려고 합니다.

보기에서 공유 리소스 파일을 사용 하려면 주입 `IHtmlLocalizer<T>`:

[!code-HTML[주](../fundamentals/localization/sample/Views/Test/About.cshtml?highlight=5,12)]

## <a name="dataannotations-localization"></a>DataAnnotations 지역화

DataAnnotations 오류 메시지와 지역화 되어 `IStringLocalizer<T>`합니다. 옵션을 사용 하 여 `ResourcesPath = "Resources"`에서 오류 메시지를 `RegisterViewModel` 다음 경로 중 하나에 저장할 수 있습니다.

* Resources/ViewModels.Account.RegisterViewModel.fr.resx
* Resources/ViewModels/Account/RegisterViewModel.fr.resx

[!code-csharp[주](localization/sample/ViewModels/Account/RegisterViewModel.cs?start=9&end=26)]

ASP.NET Core MVC 1.1.0 및 더 높은, 비-유효성 검사 특성의 지역화 됩니다. ASP.NET Core MVC 1.0 않습니다 **하지** 유효성 검사 아닌 특성에 대 한 지역화 된 문자열을 조회 합니다.

## <a name="provide-localized-resources-for-the-languages-and-cultures-you-support"></a>언어와 문화 지원에 대 한 지역화 된 리소스를 제공 합니다.  

### <a name="supportedcultures-and-supporteduicultures"></a>SupportedCultures 및 SupportedUICultures

ASP.NET Core를 사용 하면 두 문화권 값을 지정할 수 있습니다 `SupportedCultures` 및 `SupportedUICultures`합니다. [CultureInfo](https://msdn.microsoft.com/en-us/library/system.globalization.cultureinfo(v=vs.110).aspx) 개체에 대 한 `SupportedCultures` 날짜, 시간, 숫자 및 통화 형식과 같은 culture에 종속 된 함수의 결과 결정 합니다. `SupportedCultures`또한 텍스트, 대/소문자 규칙 및 문자열 비교의 정렬 순서를 결정합니다. 참조 [CultureInfo.CurrentCulture](https://msdn.microsoft.com/en-us/library/system.globalization.cultureinfo.currentculture%28v=vs.110%29.aspx) 에 대 한 자세한 내용은 서버 문화권을 가져오는 방법을 합니다. `SupportedUICultures` 문자열 변환 하는 결정 (에서 *.resx* 파일)을 조회 하 여는 [ResourceManager](https://msdn.microsoft.com/en-us/library/system.resources.resourcemanager(v=vs.110).aspx)합니다. `ResourceManager` 의해 결정 되는 문화권 관련 문자열을 단순히 조회 `CurrentUICulture`합니다. .NET의 모든 스레드에 포함 `CurrentCulture` 및 `CurrentUICulture` 개체입니다. ASP.NET Core 문화권 종속 기능을 렌더링 하는 경우 이러한 값을 검사 합니다. 예를 들어, 현재 스레드의 문화권 "EN-US" (영어, 미국)로 설정 되어 있으면 `DateTime.Now.ToLongDateString()` 이지만 if "목요일, 2 월 18, 2016", 표시 `CurrentCulture` 설정 된 "ES-ES" (스페인어, 스페인)에 출력 됩니다 "jueves, 18 febrero de-de 2016" 입니다.

## <a name="working-with-resource-files"></a>리소스 파일 작업

리소스 파일은 코드에서 지역화 가능한 문자열을 분리 하는 유용한 메커니즘입니다. 기본이 아닌 언어에 대 한 번역 된 문자열은 격리 된 *.resx* 리소스 파일입니다. 예를 들어 라는 스페인어 리소스 파일을 만드는 경우가 *Welcome.es.resx* 번역 된 문자열을 포함 합니다. "es"는 스페인어 언어 코드입니다. 이 리소스 파일을 만들려면 Visual Studio에서:

1. **솔루션 탐색기**, 리소스 파일을 포함할 폴더를 마우스 오른쪽 단추로 클릭 > **추가** > **새 항목**합니다.

    ![중첩 된 상황에 맞는 메뉴: 솔루션 탐색기에서 상황에 맞는 메뉴는 리소스에 대 한 열려 있습니다. 두 번째 상황에 맞는 메뉴를 강조 표시 된 새 항목 명령을 보여 주는 추가 대 한 열.](localization/_static/newi.png)

2. 에 **설치 된 템플릿 검색** 상자에 "리소스"를 입력 한 파일 이름을 지정 합니다.

    ![새 항목 추가 대화 상자](localization/_static/res.png)

3. 에 키 값 (네이티브 문자열)을 입력의 **이름** 열과의 번역된 된 문자열의 **값** 열입니다.

    ![이름 열에서 hello 값 열에 있는 단어 Hola (스페인어에서 Hello)와 Welcome.es.resx 파일 (스페인어 시작 리소스 파일)](localization/_static/hola.png)

    Visual Studio에서는 *Welcome.es.resx* 파일입니다.

    ![시작 스페인어 (es) 리소스 파일을 보여 주는 솔루션 탐색기](localization/_static/se.png)

## <a name="resource-file-naming"></a>리소스 파일 이름 지정

리소스는 어셈블리 이름 뺀 해당 클래스의 전체 형식 이름에 대해 이름이 지정 됩니다. 예를 들어 해당 주 어셈블리가 프로젝트에는 프랑스어 리소스 `LocalizationWebsite.Web.dll` 클래스에 대 한 `LocalizationWebsite.Web.Startup` 이름이 지정 됩니다. *Startup.fr.resx*합니다. 클래스에 대 한 리소스 `LocalizationWebsite.Web.Controllers.HomeController` 이름이 지정 됩니다. *Controllers.HomeController.fr.resx*합니다. 대상된 클래스의 네임 스페이스는 어셈블리 이름과 동일 하지는 경우 전체 형식 이름입니다. 예를 들어 샘플 프로젝트에서 형식에 대 한 리소스 `ExtraNamespace.Tools` 이름이 지정 됩니다. *ExtraNamespace.Tools.fr.resx*합니다.

샘플 프로젝트에는 `ConfigureServices` 메서드는 `ResourcesPath` 홈 컨트롤러의 프랑스어 리소스 파일에 대 한 프로젝트 상대 경로 하므로 "리소스" *Resources/Controllers.HomeController.fr.resx*합니다. 또는 리소스 파일을 구성 하려면 폴더를 사용할 수 있습니다. Home 컨트롤러에 대 한 경로 것 *Resources/Controllers/HomeController.fr.resx*합니다. 사용 하지 않는 경우는 `ResourcesPath` 옵션을는 *.resx* 프로젝트 기본 디렉터리에 파일을 거쳐야 합니다. 리소스 파일에 대 한 `HomeController` 이름이 지정 됩니다. *Controllers.HomeController.fr.resx*합니다. 리소스 파일을 구성 하는 방법에 따라 점 또는 경로 명명 규칙을 사용 하 여 선택 합니다.

| 리소스 이름 | 점 또는 경로 이름 지정 |
| ------------   | ------------- |
| Resources/Controllers.HomeController.fr.resx | 내적  |
| Resources/Controllers/HomeController.fr.resx  | Path |
|    |     |

리소스 파일을 사용 하 여 `@inject IViewLocalizer` Razor 뷰에서 비슷한 패턴을 따릅니다. 보기에 대 한 리소스 파일 점을 지정 하거나 경로 이름 지정을 사용 하 여 명명할 수 있습니다. Razor 보기 리소스 파일 관련된 뷰 파일의 경로 모방 합니다. 설정 하는 것으로 가정는 `ResourcesPath` "리소스"와 연결 된 프랑스어 리소스 파일의 *Views/Home/About.cshtml* 보기에서 다음 중 하나가 될 수 있습니다.

* Resources/Views/Home/About.fr.resx

* Resources/Views.Home.About.fr.resx

사용 하지 않는 경우는 `ResourcesPath` 옵션을는 *.resx* 보기는 보기와 같은 폴더에 있을 것에 대 한 파일입니다.

".Fr" 문화권 지정자를 제거 하는 경우 프랑스어로 설정 된 쿠키 또는 다른 메커니즘) (통해 문화권을 있으면 기본 리소스 파일을 읽고 문자열이 지역화 됩니다. 리소스 관리자는 아무 것도 문화권 지정자 없이 *.resx 파일을 제공 하 여 요청 된 문화권을 충족 하는 경우 기본 또는 대체 (fallback) 리소스를 지정 합니다. 요청 된 문화권에 대 한 리소스를 누락 된 기본 리소스 파일을가 필요한 경우 키에만 반환 하려는 경우

### <a name="generating-resource-files-with-visual-studio"></a>Visual Studio를 사용 하 여 리소스 파일을 생성합니다.

파일 이름에는 문화권 없이 Visual Studio에서 리소스 파일을 만드는 경우 (예를 들어 *Welcome.resx*), Visual Studio에서 각 문자열에 대 한 속성으로는 C# 클래스를 만듭니다. 가 성공적으로 원하지 ASP.NET 코어입니다. 일반적으로 기본값이 없는 *.resx* 리소스 파일 (A *.resx* 문화권 이름 사용 하지 않고 파일). 만들어야 하는 것이 좋습니다는 *.resx* 문화권 이름으로 파일 (예를 들어 *Welcome.fr.resx*). 만들 때 한 *.resx* 문화권 이름을 사용할 경우 Visual Studio 파일 클래스 파일을 생성 하지 것입니다. 많은 개발자 들이 하는 것이 기대 **하지** 기본 언어 리소스 파일을 만듭니다.

### <a name="adding-other-cultures"></a>다른 문화권을 추가합니다.

각 언어 및 culture 조합 (외의 기본 언어)에 고유한 리소스 파일이 필요합니다. ISO 언어 코드를 파일 이름에 있는 새 리소스 파일을 만들어 서로 다른 문화권 및 로캘에 대 한 리소스 파일을 만들 수 있습니다 (예를 들어 **en-우리**, **fr ca**, 및 **경우 en**). 이러한 ISO 코드 사이 위치한 파일 이름 및.resx 파일 이름 확장명에서와 같이 *Welcome.es MX.resx* (스페인어/멕시코). 중립 문화권에 따라 언어를 지정 하려면 제거 합니다 국가 코드와 같은 *Welcome.fr.resx* 프랑스어입니다.

## <a name="implement-a-strategy-to-select-the-languageculture-for-each-request"></a>각 요청에 대 한 언어/문화권을 선택 하는 전략을 구현 합니다.  

### <a name="configuring-localization"></a>지역화를 구성합니다.

지역화에 구성 되어는 `ConfigureServices` 메서드:

[!code-csharp[주](localization/sample/Startup.cs?range=45-49)]

* `AddLocalization`서비스 컨테이너에 번역 서비스를 추가합니다. 위의 코드는 또한 "리소스"에 대 한 리소스 경로 설정합니다.

* `AddViewLocalization`지역화 된 보기 파일에 대 한 지원을 추가합니다. 이 샘플 보기에서 지역화는 보기 파일 접미사를 기반으로 합니다. 예를 들어 "fr"에 *Index.fr.cshtml* 파일입니다.

* `AddDataAnnotationsLocalization`각 언어에 대 한 지원을 추가 `DataAnnotations` 유효성 검사를 통해 메시지 `IStringLocalizer` 추상화 합니다.

### <a name="localization-middleware"></a>지역화 미들웨어

현재 요청에 설정 된 지역화에 [미들웨어](middleware.md)합니다. 지역화 미들웨어를 사용할 수는 `Configure` 메서드의 *Startup.cs* 파일입니다. 참고, 지역화 미들웨어 요청 culture를 확인할 수 있는 모든 미들웨어 하기 전에 구성 해야 합니다 (예를 들어 `app.UseMvc()`).

[!code-csharp[주](localization/sample/Startup.cs?highlight=1-21&range=136-159)]

`UseRequestLocalization`초기화는 `RequestLocalizationOptions` 개체입니다. 모든 요청 목록에서의 `RequestCultureProvider` 에 `RequestLocalizationOptions` 열거 및 요청 culture를 성공적으로 확인할 수 있는 첫 번째 공급자가 사용 됩니다. 기본 공급자에서 제공 된 `RequestLocalizationOptions` 클래스:

1. `QueryStringRequestCultureProvider`
2. `CookieRequestCultureProvider`
3. `AcceptLanguageHeaderRequestCultureProvider`

기본 목록 가장 구체적인에서 가장 구체적으로 이동 합니다. 이 문서 뒷부분에 나오는 순서를 변경 및 사용자 지정 문화권 공급자를 추가 하는 방법을 살펴보겠습니다. 어떤 공급자가 요청 문화권을 결정할 수 있으면는 `DefaultRequestCulture` 사용 됩니다.

### <a name="querystringrequestcultureprovider"></a>QueryStringRequestCultureProvider

일부 응용 프로그램 쿼리 문자열을 사용 하 여 설정 하는 [문화권 및 UI 문화권](https://msdn.microsoft.com/en-us/library/system.globalization.cultureinfo.aspx#Current)합니다. 쿠키 또는 Accept-language 헤더 접근 방식 사용 하는 응용 프로그램의 경우 URL에 쿼리 문자열을 추가 디버깅 및 테스트 코드에 유용 합니다. 기본적으로는 `QueryStringRequestCultureProvider` 에서 첫 번째 지역화 공급자로 등록은 `RequestCultureProvider` 목록입니다. 쿼리 문자열 매개 변수를 전달 `culture` 및 `ui-culture`합니다. 다음 예제에서는 특정 문화권 (언어 및 지역) 스페인어/멕시코 설정:

   `http://localhost:5000/?culture=es-MX&ui-culture=es-MX`

둘 중 하나에 전달 하는 경우 (`culture` 또는 `ui-culture`), 쿼리 문자열 공급자에 전달 된 것을 사용 하 여 두 값을 설정 합니다. 예를 들어, 설정 된 문화권에만 설정 됩니다 모두는 `Culture` 및 `UICulture`:

   `http://localhost:5000/?culture=es-MX`

### <a name="cookierequestcultureprovider"></a>CookieRequestCultureProvider

프로덕션 응용 프로그램 제공 하는 ASP.NET 핵심 문화권 쿠키와 culture를 설정 하는 메커니즘 경우가 있습니다. 사용 된 `MakeCookieValue` 메서드를 쿠키를 만듭니다.

`CookieRequestCultureProvider` `DefaultCookieName` 문화권 정보를 사용자를 추적 하는 데 사용 하는 기본 쿠키 이름의 기본 설정으로 반환 합니다. 기본 쿠키 이름은 "입니다. AspNetCore.Culture "로 설정 합니다.

쿠키 형식은 `c=%LANGCODE%|uic=%LANGCODE%`여기서 `c` 는 `Culture` 및 `uic` 는 `UICulture`, 예:

    c='en-UK'|uic='en-US'

문화권 정보 및 UI 문화권 중 하나만 지정 하면 지정된 된 문화권 한 문화권 정보 및 UI 문화권을 모두 사용 됩니다.

### <a name="the-accept-language-http-header"></a>HTTP Accept-language 헤더

[Accept-language 헤더](https://www.w3.org/International/questions/qa-accept-lang-locales) 사용자의 언어를 지정 하려면 원래 의도 된 및 대부분의 브라우저에서 설정할 수 있습니다. 이 설정은 브라우저 전송으로 설정 된 또는 기본 운영 체제에서 상속한 것을 나타냅니다. 사용할 수 없는 사용자의 기본 설정된 언어를 확인 하는 infallible 방법을 브라우저 요청에서 HTTP Accept-language 헤더 (참조 [브라우저의 언어 기본 설정](https://www.w3.org/International/questions/qa-lang-priorities.en.php)). 프로덕션 응용 프로그램에는 사용자가 선택한 문화권의 사용자 지정 하는 방법을 포함 해야 합니다.

### <a name="setting-the-accept-language-http-header-in-ie"></a>IE에서 HTTP Accept-language 헤더를 설정합니다.

1. 기어 아이콘에서 누릅니다 **인터넷 옵션**합니다.

2. 누르기 **언어**합니다.

    ![인터넷 옵션](localization/_static/lang.png)

3. 누르기 **언어 기본 설정을**합니다.

4. 누르기 **언어 추가**합니다.

5. 언어를 추가 합니다.

6. 언어를 차례로 누릅니다 **위로 이동**합니다.

### <a name="using-a-custom-provider"></a>사용자 지정 공급자를 사용 하 여

언어 및 culture를 데이터베이스에 저장 하 여 고객에 게 알려주는 한다고 가정 합니다. 사용자에 대 한 이러한 값을 조회 하는 공급자를 작성할 수 있습니다. 다음 코드는 사용자 지정 공급자를 추가 하는 방법을 보여 줍니다.

```csharp
services.Configure<RequestLocalizationOptions>(options =>
   {
       var supportedCultures = new[]
       {
           new CultureInfo("en-US"),
           new CultureInfo("fr")
       };

       options.DefaultRequestCulture = new RequestCulture(culture: "en-US", uiCulture: "en-US");
       options.SupportedCultures = supportedCultures;
       options.SupportedUICultures = supportedCultures;

       options.RequestCultureProviders.Insert(0, new CustomRequestCultureProvider(async context =>
       {
         // My custom request culture logic
         return new ProviderCultureResult("en");
       }));
   });
   ```

사용 하 여 `RequestLocalizationOptions` 를 추가 하 여 지역화 공급자를 제거 합니다.

### <a name="setting-the-culture-programmatically"></a>프로그래밍 방식으로 문화권 설정

이 샘플 **Localization.StarterWeb** 프로젝트에 [GitHub](https://github.com/aspnet/entropy) 설정 하는 UI가 포함 된 `Culture`합니다. *Views/Shared/_SelectLanguagePartial.cshtml* 파일을 사용 하면 지원 되는 culture의 목록에서 culture를 지정할 수 있습니다.

[!code-HTML[주](localization/sample/Views/Shared/_SelectLanguagePartial.cshtml)]

*Views/Shared/_SelectLanguagePartial.cshtml* 파일에 추가 되는 `footer` 모든 보기에 사용할 수 있도록 하는 레이아웃 파일의 섹션:

[!code-HTML[주](localization/sample/Views/Shared/_Layout.cshtml?range=48-61&highlight=10)]

`SetLanguage` 메서드는 문화권 쿠키를 설정 합니다.

[!code-csharp[주](localization/sample/Controllers/HomeController.cs?range=57-67)]

연결할 수 없습니다는 *_SelectLanguagePartial.cshtml* 이 프로젝트에 대 한 샘플 코드에 있습니다. **Localization.StarterWeb** 프로젝트에 [GitHub](https://github.com/aspnet/entropy) 전달 하는 코드는 `RequestLocalizationOptions` 는 Razor를 통해 부분에 [종속성 주입](dependency-injection.md) 컨테이너입니다.

## <a name="globalization-and-localization-terms"></a>전역화 및 지역화 용어

응용 프로그램 지역화 프로세스에는 또한 일반적으로 현대 소프트웨어 개발에 사용 되는 관련 된 문자 집합에 대 한 기본적인 이해 및 관련 된 문제에 대 한 이해가 필요 합니다. 숫자 (코드)로 텍스트를 저장 하는 모든 컴퓨터, 있지만 다른 시스템 다른 번호를 사용 하 여 동일한 텍스트를 저장 합니다. 지역화 프로세스를 특정 문화권/로캘을 대 한 응용 프로그램 사용자 인터페이스 (UI)를 번역 가리킵니다.

[지역화 가능성](https://msdn.microsoft.com/en-us/library/aa292135(v=vs.71).aspx) 는 세계화 된 응용 프로그램 지역화 될 준비가 되었는지 확인 하는 것에 대 한 중간 프로세스입니다.

[RFC 4646](https://www.ietf.org/rfc/rfc4646.txt) 문화권 이름에 대 한 형식 "<languagecode2>-<country egioncode2="">" 여기서 <languagecode2> 는 언어 코드 및 <country egioncode2="">하위 문화권 코드입니다.</country> </country> 예를 들어 `es-CL` 스페인어 (칠레)에 대 한 `en-US` 영어 (미국) 및 `en-AU` 영어 (오스트레일리아). [RFC 4646](https://www.ietf.org/rfc/rfc4646.txt) 두 문자로 소문자 문화권 코드 언어와 연관 된 ISO 639 및는 ISO 3166 국가 또는 지역와 관련 된 두 문자로 대문자 하위 문화권 코드의 조합입니다.  참조 [언어 문화권 이름](https://msdn.microsoft.com/en-us/library/ee825488(v=cs.20).aspx)합니다.

국제화는 종종 "여기서 I18N" 약어로 라고 합니다. 약어 첫 번째 및 마지막 문자 걸리고 "I"와 "N" 마지막 서로 하므로 18 문자 수는 첫째 문자의 수입니다. 동일한 (G11N) 전역화 및 지역화 (L10N)에 적용 됩니다.

조건:

* 세계화 (G11N): 서로 다른 언어와 지역을 지원 응용 프로그램을 만드는 프로세스입니다.
* 지역화 (L10N): 지정 된 언어 및 지역에 대 한 앱을 사용자 지정 프로세스입니다.
* 국제화 (여기서 I18N): 전역화 및 지역화를 모두 설명합니다.
* 언어 및 지역을 필요에 따라 문화권: 않습니다.
* 중립 문화권:에 지정된 된 언어는 있지만 지역 하지 문화권입니다. (예를 들어 "en", "es")
* 특정 문화권: 지정 된 언어 및 지역에는 문화권입니다. (예를 들어 "EN-US", "EN-GB", "es CL")
* 로캘:는 로캘이 culture와 동일 합니다.

## <a name="additional-resources"></a>추가 리소스

* [Localization.StarterWeb 프로젝트](https://github.com/aspnet/entropy) 문서에 사용 합니다.
* [Visual Studio의 리소스 파일](https://msdn.microsoft.com/en-us/library/xbx3z216(v=vs.110).aspx#VSResFiles)
* [.Resx 파일의 리소스](https://msdn.microsoft.com/en-us/library/xbx3z216(v=vs.110).aspx#ResourcesFiles)
