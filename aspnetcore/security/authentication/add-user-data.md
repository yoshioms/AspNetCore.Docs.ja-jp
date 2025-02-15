---
title: ASP.NET Core プロジェクトにおける Identity へのカスタムユーザーデータの追加、ダウンロードおよび削除
author: rick-anderson
description: ASP.NET Core プロジェクトにおいて Identity にカスタム ユーザーデータを追加する方法について説明します。 GDPR ごとのデータを削除します。
ms.author: riande
ms.date: 06/18/2019
ms.custom: mvc, seodec18
uid: security/authentication/add-user-data
ms.openlocfilehash: f5a47ffd2e068414268ed9037d4376bfd21ba1bb
ms.sourcegitcommit: 215954a638d24124f791024c66fd4fb9109fd380
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/18/2019
ms.locfileid: "71080809"
---
# <a name="add-download-and-delete-custom-user-data-to-identity-in-an-aspnet-core-project"></a>ASP.NET Core プロジェクトにおける Identity へのカスタム ユーザーデータの追加、ダウンロードおよび削除

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

この記事では方法。

* ASP.NET Core web アプリにカスタム ユーザー データを追加します。
* カスタムユーザーデータモデルを<xref:Microsoft.AspNetCore.Identity.PersonalDataAttribute>属性で装飾して、自動的にダウンロードおよび削除できるようにします。 データをダウンロードして、削除することを行うには、満たす助けとなる[GDPR](xref:security/gdpr)要件。

プロジェクト サンプルは、Razor ページ web アプリから作成されますが、手順は ASP.NET Core MVC web アプリと同様。

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/add-user-data)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="prerequisites"></a>必須コンポーネント

[!INCLUDE [](~/includes/2.2-SDK.md)]

## <a name="create-a-razor-web-app"></a>Razor Web アプリの作成

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* Visual Studio の **[ファイル]** メニューから、 **[新規作成]**  >  **[プロジェクト]** の順に選択します。 プロジェクトに名前を**WebApp1**にする場合の名前空間と一致、[サンプルをダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/live/aspnetcore/security/authentication/add-user-data)コード。
* 選択**ASP.NET Core Web アプリケーション** > **OK**
* ドロップダウンで**ASP.NET Core 2.2**を選択します。
* 選択**Web アプリケーション**  > **OK**
* プロジェクトをビルドして実行します。

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

```dotnetcli
dotnet new webapp -o WebApp1
```

---

## <a name="run-the-identity-scaffolder"></a>Identity のスキャフォールディングの実行

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* **ソリューション エクスプ ローラー**、プロジェクトを右クリックして >**追加** > **スキャフォールディングされた新しい項目**します。
* 左側のウィンドウから、**スキャフォールディングの追加**ダイアログ ボックスで、 **Identity** > **追加**します。
* **ADD アイデンティティ**ダイアログ ボックスで、次のオプション。
  * 既存のレイアウト ファイルを選択して *~/Pages/Shared/_Layout.cshtml*
  * オーバーライドする次のファイルを選択します。
    * **アカウントまたは登録**
    * **アカウント/管理/インデックス**
  * 選択、 **+** 新たに作成するボタン**データ コンテキスト クラス**します。 型を受け入れる (**WebApp1.Models.WebApp1Context**場合は、プロジェクトの名前は**WebApp1**)。
  * 選択、 **+** 新たに作成するボタン**ユーザー クラス**します。 型を受け入れる (**WebApp1User**場合は、プロジェクトの名前は**WebApp1**) >**追加**します。
* 選択**追加**します。

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

ASP.NET Core scaffolder を以前インストールしていない場合は、今すぐインストールします。

```dotnetcli
dotnet tool install -g dotnet-aspnet-codegenerator
```

パッケージ参照を追加[Microsoft.VisualStudio.Web.CodeGeneration.Design](https://www.nuget.org/packages/Microsoft.VisualStudio.Web.CodeGeneration.Design/)プロジェクト (.csproj) ファイル。 プロジェクト ディレクトリに、次のコマンドを実行します。

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet restore
```

Identity scaffolder オプションを一覧表示するには、次のコマンドを実行します。

```dotnetcli
dotnet aspnet-codegenerator identity -h
```

プロジェクト フォルダーでは、Identity scaffolder を実行します。

```dotnetcli
dotnet aspnet-codegenerator identity -u WebApp1User -fi Account.Register;Account.Manage.Index
```

---

指示に従って、[移行、UseAuthentication、およびレイアウト](xref:security/authentication/scaffold-identity#efm)次の手順を実行します。

* 移行を作成し、データベースを更新します。
* `UseAuthentication` に `Startup.Configure` を追加します。
* 追加`<partial name="_LoginPartial" />`レイアウト ファイルにします。
* アプリをテストします。
  * ユーザーを登録する
  * 新しいユーザー名を選択します (次に、**ログアウト**リンク)。 ウィンドウを拡大またはユーザー名とその他のリンクを表示するナビゲーション バーのアイコンを選択する必要があります。
  * 選択、**個人データ**タブ。
  * 選択、**ダウンロード**ボタンをクリックし、調査、 *PersonalData.json*ファイル。
  * テスト、**削除**ボタンで、ユーザーのログオンを削除します。

## <a name="add-custom-user-data-to-the-identity-db"></a>Identity DB へのカスタム ユーザーデータの追加

更新プログラム、`IdentityUser`カスタム プロパティを持つクラスを派生します。 ファイルの名前は WebApp1 プロジェクトの名前を付けた場合*Areas/Identity/Data/WebApp1User.cs*します。 次のコード ファイルを更新します。

[!code-csharp[Main](add-user-data/sample-2.2/Areas/Identity/Data/WebApp1User.cs)]

修飾されたプロパティ、 [PersonalData](/dotnet/api/microsoft.aspnetcore.identity.personaldataattribute?view=aspnetcore-2.1)属性には。

* 削除されたときに、 *Areas/Identity/Pages/Account/Manage/DeletePersonalData.cshtml* Razor ページを呼び出して`UserManager.Delete`します。
* によってデータのダウンロードに含まれる、 *Areas/Identity/Pages/Account/Manage/DownloadPersonalData.cshtml* Razor ページ。

### <a name="update-the-accountmanageindexcshtml-page"></a>Account/Manage/Index.cshtml ページの更新

更新プログラム、`InputModel`で*Areas/Identity/Pages/Account/Manage/Index.cshtml.cs*次のようにコードを強調表示されます。

[!code-csharp[Main](add-user-data/sample-2.2/Areas/Identity/Pages/Account/Manage/Index.cshtml.cs?name=snippet&highlight=28-36,63-64,98-106,119)]

更新プログラム、 *Areas/Identity/Pages/Account/Manage/Index.cshtml*を次の強調表示されているマークアップ。

[!code-html[Main](add-user-data/sample-2.2/Areas/Identity/Pages/Account/Manage/Index.cshtml?highlight=35-42)]

### <a name="update-the-accountregistercshtml-page"></a>Account/Register.cshtml ページの更新

更新プログラム、`InputModel`で*Areas/Identity/Pages/Account/Register.cshtml.cs*次のようにコードを強調表示されます。

[!code-csharp[Main](add-user-data/sample-2.2/Areas/Identity/Pages/Account/Register.cshtml.cs?name=snippet&highlight=28-36,67,66)]

更新プログラム、 *Areas/Identity/Pages/Account/Register.cshtml*を次の強調表示されているマークアップ。

[!code-html[Main](add-user-data/sample-2.2/Areas/Identity/Pages/Account/Register.cshtml?highlight=16-25)]

プロジェクトをビルドします。

### <a name="add-a-migration-for-the-custom-user-data"></a>カスタム ユーザー データのマイグレーションの追加

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Visual Studio で**パッケージ マネージャー コンソール**:

```powershell
Add-Migration CustomUserData
Update-Database
```

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

```dotnetcli
dotnet ef migrations add CustomUserData
dotnet ef database update
```

---

## <a name="test-create-view-download-delete-custom-user-data"></a>カスタム ユーザー データの作成、表示、ダウンロード、削除のテスト

アプリをテストします。

* 新しいユーザーを登録します。
* カスタムのユーザー データを表示、`/Identity/Account/Manage`ページ。
* ダウンロードしてから、ユーザーの個人データを表示、`/Identity/Account/Manage/PersonalData`ページ。
