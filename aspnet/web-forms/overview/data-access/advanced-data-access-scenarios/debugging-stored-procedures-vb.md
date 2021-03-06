---
uid: web-forms/overview/data-access/advanced-data-access-scenarios/debugging-stored-procedures-vb
title: "저장된 프로시저 (VB) 디버깅 | Microsoft Docs"
author: rick-anderson
description: "Visual Studio Professional 및 Team System 버전 수 있도록 중단점을 설정 하 고 SQL Server 내에서 저장된 프로시저에 단계 저장 디버깅을 만드는 중..."
ms.author: aspnetcontent
manager: wpickett
ms.date: 08/03/2007
ms.topic: article
ms.assetid: 9ed8ccb5-5f31-4eb4-976d-cabf4b45ca09
ms.technology: dotnet-webforms
ms.prod: .net-framework
msc.legacyurl: /web-forms/overview/data-access/advanced-data-access-scenarios/debugging-stored-procedures-vb
msc.type: authoredcontent
ms.openlocfilehash: ad09847d828d02019a72e3022d035a8fbe921568
ms.sourcegitcommit: 060879fcf3f73d2366b5c811986f8695fff65db8
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/24/2018
---
<a name="debugging-stored-procedures-vb"></a>저장된 프로시저 (VB) 디버깅
====================
으로 [Scott Mitchell](https://twitter.com/ScottOnWriting)

[코드를 다운로드](http://download.microsoft.com/download/3/9/f/39f92b37-e92e-4ab3-909e-b4ef23d01aa3/ASPNET_Data_Tutorial_74_VB.zip) 또는 [PDF 다운로드](debugging-stored-procedures-vb/_static/datatutorial74vb1.pdf)

> Visual Studio Professional 및 Team System 버전 응용 프로그램 코드를 디버깅 하는 것 만큼 쉽게 저장된 프로시저를 디버깅 하기 중단점을 설정 하 고 SQL Server 내에서 저장된 프로시저에 단계 수 있습니다. 이 자습서에서는 데이터베이스를 직접 디버깅 및 저장된 프로시저의 응용 프로그램 디버깅에 대해 설명 합니다.


## <a name="introduction"></a>소개

Visual Studio는 풍부한 디버깅 환경을 제공 합니다. 몇 가지 키 입력 또는 마우스 클릭 만으로 것 s 중단점을 사용 하는 프로그램의 실행을 중지 하 여 해당 상태 및 제어 흐름을 검사할 수 있습니다. 응용 프로그램 코드를 디버깅 하와 함께 Visual Studio는 SQL Server 내에서 저장된 프로시저를 디버깅 지원 합니다. ASP.NET 코드 숨김 클래스 또는 비즈니스 논리 계층 클래스의 코드 내에서 중단점을 설정 하는 것 처럼 하므로 너무은 내에 배치할 수 저장된 프로시저입니다.

이 자습서를 한 단계씩 실행 저장된 프로시저 내 Visual Studio 서버 탐색기에서도 적중 하는 중단점을 설정 하는 저장된 프로시저 호출 될 때 실행 중인 ASP.NET 응용 프로그램에서 하는 방법에 대해 살펴보겠습니다.

> [!NOTE]
> 그러나 저장된 프로시저 수만 한 단계씩 코드 실행 하 고 팀 시스템 및 Professional 버전의 Visual Studio를 통해 디버깅 합니다. 표준 버전의 Visual Studio 또는 Visual Web Developer를 사용 하는 저장된 프로시저를 디버깅 하는 데 필요한 단계를 살펴봅니다 있지만 컴퓨터에 이러한 단계를 복제할 수 없습니다와 함께 읽기 시작 됩니다.


## <a name="sql-server-debugging-concepts"></a>SQL Server 디버깅 개념

Microsoft SQL Server 2005와의 통합을 제공 하도록 디자인 된는 [공용 언어 런타임 (CLR)](https://msdn.microsoft.com/netframework/aa497266.aspx), 하는 모든.NET 어셈블리에서 사용 하는 런타임입니다. 따라서 SQL Server 2005 관리 되는 데이터베이스 개체를 지원합니다. 즉, Visual Basic 클래스의 메서드로 저장된 프로시저 및 사용자 정의 함수 (Udf)와 같은 데이터베이스 개체를 만들 수 있습니다. 이렇게 하면 이러한 저장된 프로시저 및 사용자 지정 클래스와.NET Framework의 기능을 활용 하는 Udf 수 있습니다. 물론, SQL Server 2005는 또한 T-SQL 데이터베이스 개체에 대 한 지원을 제공합니다.

SQL Server 2005는 T-SQL와 관리 되는 데이터베이스 개체를 모두 디버깅 지원 합니다. 그러나 이러한 개체는 팀 시스템 및 Visual Studio 2005 Professional edition을 통해만 디버그할 수 있습니다. 이 자습서에서는 디버깅 T-SQL 데이터베이스 개체를 살펴보겠습니다. 이후의 자습서는 관리 되는 데이터베이스 개체 디버깅 살펴봅니다.

[개요의 T-SQL 및 CLR 디버깅 SQL Server 2005에서](https://blogs.msdn.com/sqlclr/archive/2006/06/29/651644.aspx) 에서 블로그 항목은 [SQL Server 2005 CLR 통합 팀](https://blogs.msdn.com/sqlclr/default.aspx) Visual Studio에서 SQL Server 2005 개체를 디버깅 하는 세 가지 방법으로 강조 표시 합니다.

- **데이터베이스 디버깅 (DDD) 직접** -서버 탐색기 저장된 프로시저 및 Udf 등 T-SQL 데이터베이스 개체를 한 단계씩 실행할 수 있습니다. 1 단계에서에서 DDD를 살펴보겠습니다.
- **응용 프로그램 디버깅** -데이터베이스 개체 내에서 중단점을 설정 하 고 다음 ASP.NET 응용 프로그램을 실행할 수 있습니다. 데이터베이스 개체를 실행 하면 중단점이 적중 됩니다 및 제어 하 고 디버거를 통해 설정 합니다. 응용 프로그램 디버깅 म 없습니다 단계적으로 데이터베이스 개체 응용 프로그램 코드에서 참고 합니다. 에서는 명시적으로 설정 해야 중단점 이러한 저장된 프로시저 또는 Udf의 디버거를 중지 하도록 하겠습니다. 응용 프로그램 디버깅 2 단계부터이 검사 됩니다.
- **SQL Server 프로젝트에서 디버깅** -팀 시스템 및 Visual Studio Professional edition에는 관리 되는 데이터베이스 개체를 만드는 데 주로 사용 하는 SQL Server 프로젝트 유형을 포함 합니다. SQL Server 프로젝트를 사용 하 여 살펴보겠습니다 및 다음 자습서에서 해당 콘텐츠를 디버깅 합니다.

Visual Studio는 로컬 및 원격 SQL Server 인스턴스에서 저장된 프로시저를 디버깅할 수 있습니다. 로컬 SQL Server 인스턴스는 Visual Studio와 동일한 컴퓨터에 설치 되어입니다. 사용 중인 SQL Server 데이터베이스 개발 컴퓨터에 없는 경우, 원격 인스턴스를 간주 됩니다. 이러한 자습서에서는 사용 되 고 로컬 SQL Server 인스턴스. 원격 SQL server 인스턴스에서 저장된 프로시저를 디버깅 합니다. 로컬 인스턴스에 저장된 프로시저를 디버깅 하는 경우 보다 더 많은 구성 단계가 필요 합니다.

로컬 SQL Server 인스턴스를 사용 하는 경우 단계 1부터 시작 하 고 끝에이 자습서를 통해 작업 수 있습니다. 그러나 원격 SQL Server 인스턴스를 사용 하는 경우 할 수 있습니다를 디버깅할 때 되어 있는지 확인 하는 첫 번째 필요가 원격 인스턴스에서 SQL Server 로그인을 갖고 있는 Windows 사용자 계정으로 개발 컴퓨터에 기록 됩니다. Moveover,이 데이터베이스 로그인 및 실행 중인 ASP.NET 응용 프로그램에서 데이터베이스에 연결 하는 데 사용 되는 데이터베이스 로그인을 모두의 구성원 이어야 합니다.는 `sysadmin` 역할입니다. 원격 인스턴스에 섹션에 맞춰져 있으며 원격 인스턴스를 디버깅 하려면 Visual Studio 및 SQL Server 구성에 대 한 자세한 내용은이 자습서의 끝에서 T-SQL 디버깅 데이터베이스 개체를 참조 하십시오.

마지막으로, T-SQL 데이터베이스 개체에 대 한 지원을 디버깅 아닌지 디버깅.NET 응용 프로그램에 대 한 지원으로 다양 한 기능을 이해 합니다. 예를 들어, 중단점 조건 및 필터 지원 되지 않으며, 디버깅 창의 하위 집합을 사용할 수 있는 편집 하며 계속 하기를 사용할 수 없습니다, 쓸모와 같은 직접 실행 창 렌더링 됩니다. 참조 [디버거 명령 및 기능에 대 한 제한](https://msdn.microsoft.com/library/ms165035(VS.80).aspx) 자세한 정보에 대 한 합니다.

## <a name="step-1-directly-stepping-into-a-stored-procedure"></a>저장된 프로시저를 직접 한 단계씩 실행 1 단계:

Visual Studio에서는 쉽게 데이터베이스 개체를 직접 디버그할 수 있습니다. 한 단계씩 코드 실행을 직접 데이터베이스 디버깅 (DDD) 기능을 사용 하는 방법에 대해 s는 `Products_SelectByCategoryID` Northwind 데이터베이스에서 저장 프로시저입니다. 이름에서 알 수 있듯이 `Products_SelectByCategoryID` 특정 범주의;에 대 한 제품 정보를 반환에서 만든는 [를 사용 하 여 기존 저장 프로시저 형식화 된 데이터 집합의 Tableadapter에 대 한](using-existing-stored-procedures-for-the-typed-dataset-s-tableadapters-vb.md) 자습서입니다. 서버 탐색기로 이동 하 여 시작 하 고 Northwind 데이터베이스 노드를 확장 합니다. 그 다음, Stored Procedures 폴더에 드릴 다운을 마우스 오른쪽 단추로 클릭는 `Products_SelectByCategoryID` 저장 프로시저를 하 고 상황에 맞는 메뉴에서 저장 프로시저 한 단계씩 코드 실행 옵션을 선택 합니다. 그러면 디버거를 시작 됩니다.

이후는 `Products_SelectByCategoryID` 저장된 프로시저에 필요한는 `@CategoryID` 입력된 매개 변수 라는 요청이 값을 제공 해야 합니다. 음료에 대 한 정보를 반환 합니다: 1을 입력 합니다.


![에 대 한 값 1을 사용 하 여 @CategoryID 매개 변수](debugging-stored-procedures-vb/_static/image1.png)

**그림 1**:에 대 한 값 1을 사용 하 여 `@CategoryID` 매개 변수


에 대 한 값을 입력 한 후의 `@CategoryID` 매개 변수를 저장된 프로시저 실행 됩니다. 그러나 완료 될 때까지 실행 하는 대신 디버거가 첫 번째 문에서 실행을 중단 합니다. 저장된 프로시저에서 현재 위치를 나타내는 여백에 노란색 화살표를 note 합니다. 확인 및 조사식 창 또는 저장된 프로시저에서 매개 변수 이름 위로 이동 하 여 매개 변수 값을 편집할 수 있습니다.


[![디버거는 저장 프로시저의 첫 번째 문에서 중지](debugging-stored-procedures-vb/_static/image3.png)](debugging-stored-procedures-vb/_static/image2.png)

**그림 2**: 디버거가 저장 프로시저의 첫 번째 문에서 중지 ([전체 크기 이미지를 보려면 클릭](debugging-stored-procedures-vb/_static/image4.png))


한 번에 문 하나씩 저장된 프로시저를 단계별로 실행 되도록 프로시저 단위 실행 도구 모음 단추를 클릭 하거나 F10 키를 누릅니다. `Products_SelectByCategoryID` 저장된 프로시저에는 단일 `SELECT` 문을, F10 도달 단일 문을 프로시저 단위로 실행 되 고 저장된 프로시저의 실행을 완료 합니다. 저장된 프로시저가 완료 된 후 출력 창에 해당 출력 나타나고 디버거를 종료 합니다.

> [!NOTE]
> T-SQL 디버깅; 문 수준에서 발생 단계씩 실행할 수 없습니다는 `SELECT` 문.


## <a name="step-2-configuring-the-website-for-application-debugging"></a>응용 프로그램 디버깅에 대 한 웹 사이트를 구성 하는 2 단계:

서버 탐색기에서 직접 저장된 프로시저를 디버깅 하는 것은 유용, 대부분의 시나리오에서 우리에 더 관심이 ASP.NET 응용 프로그램에서 호출 될 때 저장된 프로시저를 디버깅 합니다. Visual Studio 내에서 저장된 프로시저에 중단점을 추가 하 고 ASP.NET 응용 프로그램을 디버깅를 시작할 수 있습니다. 중단점을 포함 하는 저장된 프로시저는 응용 프로그램에서 호출 되 면 실행이 중단점에서 중단 됩니다 및 수 및 변경 된 저장된 프로시저 s 매개 변수 값을 보고 1 단계에서에서 했던 것 처럼 해당 문을 단계별로 실행 합니다.

응용 프로그램에서 호출한 저장된 프로시저를 디버깅을 시작할 수 있습니다, 전에 SQL Server 디버거와 통합 하는 ASP.NET 웹 응용 프로그램에 지시 해야 합니다. 솔루션 탐색기에서 웹 사이트 이름을 마우스 오른쪽 단추로 클릭 하 여 시작 (`ASPNET_Data_Tutorial_74_VB`). 상황에 맞는 메뉴에서 속성 페이지 옵션을 선택, 왼쪽에서 시작 옵션 항목을 선택 하 고 디버거 섹션에서 SQL Server 확인란 (그림 3 참조).


[![응용 프로그램의 속성 페이지에서 SQL Server 확인란](debugging-stored-procedures-vb/_static/image6.png)](debugging-stored-procedures-vb/_static/image5.png)

**그림 3**: 응용 프로그램의 속성 페이지에서에서 SQL Server 확인란 ([전체 크기 이미지를 보려면 클릭](debugging-stored-procedures-vb/_static/image7.png))


또한 연결 풀링을 사용 하지 않도록 설정 하는 응용 프로그램에서 사용 되는 데이터베이스 연결 문자열을 업데이트 해야 합니다. 데이터베이스에 대 한 연결 종료 되 면 해당 `SqlConnection` 개체의 사용 가능한 연결 풀에 배치 됩니다. 데이터베이스에 연결을 설정할 때 사용 가능한 연결 보다는 개체가이 풀에서 검색할 수 있습니다 필요를 만들고 새 연결을 설정 합니다. 이 풀링을 연결 개체의 성능 향상 이며 기본적으로 사용 됩니다. 그러나 디버깅할 때 디버깅 인프라에는 풀에서 만든 연결으로 작업 하는 경우 올바르게 다시 설정 하지 때문에 연결 풀링을 해제 하려고 합니다.

사용할 수 없는 연결 풀링, 하려면 업데이트는 `NORTHWNDConnectionString` 에 `Web.config` 설정이 포함 되도록 `Pooling=false` 합니다.


[!code-xml[Main](debugging-stored-procedures-vb/samples/sample1.xml)]

> [!NOTE]
> 완료 했으면 ASP.NET 응용 프로그램을 통해 SQL Server 디버깅 해야 제거 하 여 연결 풀링 reinstate는 `Pooling` 연결 문자열에서 설정 (또는로 설정 하 여 `Pooling=true` ).


이 시점에서 ASP.NET 응용 프로그램에 웹 응용 프로그램을 통해 호출 되 면 SQL Server 데이터베이스 개체를 디버깅 하려면 Visual Studio를 허용 하도록 구성한 합니다. 이제 하는 저장된 프로시저에 중단점을 추가 하 고 디버깅을 시작!

## <a name="step-3-adding-a-breakpoint-and-debugging"></a>중단점을 추가 하 고 디버깅 하는 3 단계:

열기는 `Products_SelectByCategoryID` 의 시작 부분에 중단점을 설정 하 고 저장 프로시저는 `SELECT` 적절 한 위치에서 여백을 클릭 하 여 또는의 시작 부분에 커서를 배치 하 여 문에서 `SELECT` 문과 f9 합니다. 그림 4에서 알 수 있듯이, 중단점 여백에 빨간색 원이으로 나타납니다.


[![Products_SelectByCategoryID에 중단점을 설정할 저장 프로시저](debugging-stored-procedures-vb/_static/image9.png)](debugging-stored-procedures-vb/_static/image8.png)

**그림 4**:에 중단점을 설정는 `Products_SelectByCategoryID` 저장 프로시저 ([전체 크기 이미지를 보려면 클릭](debugging-stored-procedures-vb/_static/image10.png))


클라이언트 응용 프로그램을 통해 디버그 해야 하는 SQL 데이터베이스 개체에 대 한 순서로 데이터베이스 응용 프로그램 디버깅을 지원 하도록 구성 되어야 합니다. 처음에 중단점을 설정 하면이 설정은, 자동으로 전환 해야 하지만 검토 하는 것이 좋습니다. 마우스 오른쪽 단추로 클릭는 `NORTHWND.MDF` 서버 탐색기에서 노드. 상황에 맞는 메뉴에는 확인 된 이라는 메뉴 항목이 응용 프로그램 디버깅 포함 되어야 합니다.


![응용 프로그램 디버깅 옵션을 사용할 수 있는지 확인 하십시오.](debugging-stored-procedures-vb/_static/image11.png)

**그림 5**: 응용 프로그램 디버깅 옵션을 사용할 수 있는지 확인 하십시오.


중단점 설정 및 사용할 수 있는 응용 프로그램 디버깅 옵션 사용 하 여 ASP.NET 응용 프로그램에서 호출 될 때 저장된 프로시저 디버깅 준비가 됩니다. 디버그 메뉴에 이동 하 여 디버거를 시작 하 고 도구 모음에서 재생 아이콘 f 5를 클릭 하 여 또는 녹색을 클릭 하 여 디버깅 시작을 선택 합니다. 디버거 시작 되 고 웹 사이트를 시작 합니다.

`Products_SelectByCategoryID` 저장된 프로시저에서 만든는 [를 사용 하 여 기존 저장 프로시저 형식화 된 데이터 집합의 Tableadapter에 대 한](using-existing-stored-procedures-for-the-typed-dataset-s-tableadapters-vb.md) 자습서입니다. 해당 웹 페이지 (`~/AdvancedDAL/ExistingSprocs.aspx`)이 저장된 프로시저에서 반환 된 결과 표시 하는 gridview입니다. 브라우저를 통해이 페이지를 방문 합니다. 페이지에서의 중단점에 도달할 때는 `Products_SelectByCategoryID` 도달 저장된 프로시저 및 Visual Studio로 제어를 반환 합니다. 와 마찬가지로 1 단계에서에서 저장된 프로시저의 문 및 보기를 단계별로 실행 하 고 수정할 수 매개 변수 값입니다.


[![ExistingSprocs.aspx 페이지에는 처음에 Beverages 표시 됩니다.](debugging-stored-procedures-vb/_static/image13.png)](debugging-stored-procedures-vb/_static/image12.png)

**그림 6**:는 `ExistingSprocs.aspx` 페이지에는 처음에 Beverages 표시 됩니다 ([전체 크기 이미지를 보려면 클릭](debugging-stored-procedures-vb/_static/image14.png))


[![저장 프로시저의 중단점에 도달 했습니다.](debugging-stored-procedures-vb/_static/image16.png)](debugging-stored-procedures-vb/_static/image15.png)

**그림 7**: 저장 프로시저의 s 중단점에 도달 했습니다 ([전체 크기 이미지를 보려면 클릭](debugging-stored-procedures-vb/_static/image17.png))


그림 7에서는의 값에서 조사식 창으로는 `@CategoryID` 매개 변수는 1입니다. 때문에 이것이 `ExistingSprocs.aspx` 페이지는 음료 범주에 제품 나타냅니다는 `CategoryID` 값이 1입니다. 드롭다운 목록에서 다른 범주를 선택 합니다. 이렇게 포스트백을 발생 되 고 다시 실행은 `Products_SelectByCategoryID` 저장 프로시저입니다. 다시, 중단점이 적중 되는 `@CategoryID` s 선택한 드롭 다운 목록 항목을 반영 하는 매개 변수의 값 `CategoryID`합니다.


[![드롭 다운 목록에서 다른 범주를 선택 합니다.](debugging-stored-procedures-vb/_static/image19.png)](debugging-stored-procedures-vb/_static/image18.png)

**그림 8**: 드롭 다운 목록에서 다른 범주를 선택 ([전체 크기 이미지를 보려면 클릭](debugging-stored-procedures-vb/_static/image20.png))


[![@CategoryID 웹 페이지에서 선택한 범주를 반영 하는 매개 변수](debugging-stored-procedures-vb/_static/image22.png)](debugging-stored-procedures-vb/_static/image21.png)

**그림 9**:는 `@CategoryID` 매개 변수는 웹 페이지에서 범주 선택 반영 ([전체 크기 이미지를 보려면 클릭](debugging-stored-procedures-vb/_static/image23.png))


> [!NOTE]
> 경우에 설정한 중단점은 `Products_SelectByCategoryID` 을 방문할 때 저장된 프로시저 적중 되지 않습니다는 `ExistingSprocs.aspx` 페이지에서 SQL Server 확인란은 체크 인 s 속성 페이지는 ASP.NET 응용 프로그램의 디버거 섹션, 연결 풀링 되었음을 있는지 확인 사용 안 함을 선택 하 고 응용 프로그램 디버깅 옵션의 데이터베이스를 사용할 수 있음을 합니다. 여전히 다시 문제가 Visual Studio를 다시 시작 하 고 다시 시도 하십시오.


## <a name="debugging-t-sql-database-objects-on-remote-instances"></a>원격 인스턴스에서 T-SQL 데이터베이스 개체 디버깅

Visual Studio와 동일한 컴퓨터에 SQL Server 데이터베이스 인스턴스가 있으면 Visual Studio를 통해 데이터베이스 개체를 디버깅 하는 것은 매우 간단 합니다. 그러나, 서로 다른 컴퓨터에서 SQL Server 및 Visual Studio 상주 하는 경우 몇 가지 주의 구성이 제대로 작동 하는 모든를 얻을 수 필수입니다. 우리는 문제에 직면 하는 두 가지 핵심 작업 가지가 있습니다.

- ADO.NET 통해 데이터베이스에 연결 하는 데 사용 되는 로그인에 속해 있는지 확인은 `sysadmin` 역할입니다.
- Visual Studio에서 개발 컴퓨터에서 사용 하는 Windows 사용자 계정에 속하는 올바른 SQL Server 로그인 계정 인지 확인는 `sysadmin` 역할입니다.

첫 번째 단계는 비교적 간단 합니다. 먼저, ASP.NET 응용 프로그램에서 데이터베이스에 연결 하 고 다음 SQL Server Management Studio에서 해당 로그인 계정을 추가 하는 데 사용 하는 사용자 계정 식별는 `sysadmin` 역할입니다.

두 번째 작업은 응용 프로그램을 디버깅 하는 데 Windows 사용자 계정을 원격 데이터베이스에서 유효한 로그인 하도록 요구 합니다. 그러나 가능성에는 Windows 계정으로 사용자의 워크스테이션에 로그온 하는 경우 SQL Server에서 유효한 로그인 하지 않습니다. SQL Server에 특정 로그인 계정을 추가 하는 대신 더 적합할 SQL Server 디버깅 계정으로 Windows 사용자 계정을 일부를 지정 하는 것입니다. 그런 다음 원격 SQL Server 인스턴스의 데이터베이스 개체를 디버깅 하려면 Visual Studio는 Windows 로그인 계정 s 자격 증명을 사용 하 여 실행 합니다.

예제는 작업을 명확 하 게 하는 데 도움이 됩니다. Windows 계정인은 `SQLDebug` Windows 도메인 내에서. 이 계정은의 구성원으로 유효한 로그인과 원격 SQL Server 인스턴스에 추가할 해야는 `sysadmin` 역할입니다. 그런 다음 Visual Studio에서 원격 SQL Server 인스턴스를 디버깅 하려면 해야 Visual Studio와 실행의 `SQLDebug` 사용자입니다. 로그 아웃 우리의 워크스테이션으로 다시 로그인 하 여 수행할 수 있습니다 `SQLDebug`, 고유한 자격 증명을 사용 하 여이 워크스테이션에 로그인을 사용 하 여 간단 하지만 Visual Studio를 실행 될 수 `runas.exe` 으로 Visual Studio를 시작 하는 `SQLDebug` 사용자입니다. `runas.exe`특정 응용 프로그램을에서 다른 사용자 계정 가장 하 여 실행할 수 있습니다. 으로 Visual Studio를 시작 `SQLDebug`, 명령줄에서 다음 문을 입력할 수 있습니다.


[!code-console[Main](debugging-stored-procedures-vb/samples/sample2.cmd)]

이 프로세스에 대 한 자세한 내용은 참조 하십시오. [William 오른쪽 Vaughn](http://betav.com/BLOG/billva/) s *Hitchhiker s Visual Studio 및 SQL Server, 일곱 번째 버전 가이드* 으로 [방법: SQL Server 사용 권한 설정 디버깅을 위해](https://msdn.microsoft.com/library/w1bhybwz(VS.80).aspx)합니다.

> [!NOTE]
> 개발 컴퓨터에서 Windows XP 서비스 팩 2를 실행 중인 경우 인터넷 연결 방화벽 원격 디버깅을 허용 하도록 구성 해야 합니다. [방법에: SQL Server 2005 디버깅 사용](https://msdn.microsoft.com/library/s0fk6z6e(VS.80).aspx) 문서 정보는이 두 단계: (a)에서 Visual Studio 호스트 컴퓨터를 추가 해야 `Devenv.exe` 열어야 예외 목록 및 TCP 135 포트를 열기 및 (b)에서 원격 컴퓨터 (SQL) TCP 135 포트 사용 되 고 추가 `sqlservr.exe` 예외 목록에 있습니다. 도메인 정책에 따라 IPSec을 통해 수행 해야 하는 네트워크 통신을 필요한 경우 UDP 4500 및 UDP 500 포트를 열어야 합니다.


## <a name="summary"></a>요약

.NET 응용 프로그램 코드에 대 한 디버깅 지원에 제공할 뿐 아니라 Visual Studio 또한 다양 한 디버깅 SQL Server 2005에 대 한 옵션을 제공 합니다. 이 자습서에서는 이러한 옵션 중에서 찾았습니다: 데이터베이스를 직접 디버깅 및 응용 프로그램 디버깅 합니다. T-SQL 데이터베이스 개체를 직접 디버깅 서버 탐색기를 통해 개체를 배치한 다음에서 마우스 오른쪽 단추로 클릭 하 고 한 단계씩 코드 실행 선택 합니다. 디버거를 시작 하 고 데이터베이스 개체를 이때 개체의 문 및 보기를 단계별로 실행 하 고 수정할 수 매개 변수 값의 첫 번째 문에서 중지 합니다. 1 단계에서에서 한 단계씩 코드 실행이 접근 방식 사용은 `Products_SelectByCategoryID` 저장 프로시저입니다.

응용 프로그램 디버깅 중단점을을 데이터베이스 개체 내에서 직접 설정할 수 있습니다. 중단점을 포함 하는 데이터베이스 개체 (예: ASP.NET 웹 응용 프로그램)는 클라이언트 응용 프로그램에서 호출 되 면 디버거가 수행 하는 대로 프로그램 중단 합니다. 응용 프로그램 디버깅 것 보다 명확 하 게 보여 주기 때문에 어떤 응용 프로그램 작업 호출 해야 할 특정 데이터베이스 개체를 사용 하면 유용 합니다. 그러나 좀 더 구성 및 데이터베이스를 직접 디버깅할 보다 설치가 필요합니다.

SQL Server 프로젝트를 통해 데이터베이스 개체를 디버깅할 수도 있습니다. SQL Server 프로젝트를 사용 하는 방법에 및 다음 자습서의 관리 되는 데이터베이스 개체 작성 하 고 디버깅 사용 하는 방법입니다.

만족도 매우 프로그래밍!

## <a name="about-the-author"></a>작성자 정보

[Scott Mitchell](http://www.4guysfromrolla.com/ScottMitchell.shtml), 7 ASP/ASP.NET 서적과의 창립자의 작성자 [4GuysFromRolla.com](http://www.4guysfromrolla.com), 1998 이후 Microsoft 웹 기술과 함께 작동 합니다. Scott 독립 컨설턴트, 강사, 기술 및 작성기 작동합니다. 그의 최신 서적은 [ *Sam 업무량이 직접 ASP.NET 2.0 24 시간 동안에서*](https://www.amazon.com/exec/obidos/ASIN/0672327384/4guysfromrollaco)합니다. 에 연결할 수 그 [ mitchell@4GuysFromRolla.com.](mailto:mitchell@4GuysFromRolla.com) 에서 찾을 수 있는 그의 블로그를 통해 또는 [http://ScottOnWriting.NET](http://ScottOnWriting.NET)합니다.

>[!div class="step-by-step"]
[이전](protecting-connection-strings-and-other-configuration-information-vb.md)
[다음](creating-stored-procedures-and-user-defined-functions-with-managed-code-vb.md)
