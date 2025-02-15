---
title: ASP.NET Core Web API の応答データの書式設定
author: ardalis
description: ASP.NET Core Web API で応答データを書式設定する方法について説明します。
ms.author: riande
ms.custom: H1Hack27Feb2017
ms.date: 8/22/2019
uid: web-api/advanced/formatting
ms.openlocfilehash: e503df3d81efbb2800503c0cb4ff5ae093b6e1ac
ms.sourcegitcommit: 023495344053dc59115c80538f0ece935e7490a2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/28/2019
ms.locfileid: "71592348"
---
# <a name="format-response-data-in-aspnet-core-web-api"></a>ASP.NET Core Web API の応答データの書式設定

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)、[Steve Smith](https://ardalis.com/)

ASP.NET Core MVC では、応答データの書式設定がサポートされます。 応答データは、特定の形式を使用して、またはクライアントが要求した形式に応じて書式設定できます。

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/advanced/formatting)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="format-specific-action-results"></a>書式固有アクションの結果

アクションの結果には、<xref:Microsoft.AspNetCore.Mvc.JsonResult> や <xref:Microsoft.AspNetCore.Mvc.ContentResult> のように、特定の形式に固有となる型があります。 アクションでは、クライアントの設定に関係なく、特定の形式で書式設定された結果を返すことができます。 たとえば、`JsonResult` を返すと、JSON 形式のデータが返されます。 `ContentResult` または文字列を返すと、プレーンテキスト形式の文字列データが返されます。

アクションが特定の型を返す必要はありません。 ASP.NET Core によって、すべてのオブジェクトの戻り値がサポートされます。  型が <xref:Microsoft.AspNetCore.Mvc.IActionResult> ではないオブジェクトを返すアクションからの結果は、適切な <xref:Microsoft.AspNetCore.Mvc.Formatters.IOutputFormatter> 実装を利用してシリアル化されます。 詳細については、<xref:web-api/action-return-types> を参照してください。

組み込みヘルパー メソッド <xref:Microsoft.AspNetCore.Mvc.ControllerBase.Ok*> では、JSON 形式のデータが返されます。[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_get)]

サンプル ダウンロードでは作成者の一覧が返されます。 F12 ブラウザー開発者ツールまたは [Postman](https://www.getpostman.com/tools) と前のコードを使用します。

* **content-type:** `application/json; charset=utf-8` を含む応答ヘッダーが表示されます。
* 要求ヘッダーが表示されます。 たとえば、`Accept` ヘッダーです。 `Accept` ヘッダーは前のコードでは無視されます。

プレーンテキストで書式設定されたデータを返すには、<xref:Microsoft.AspNetCore.Mvc.ContentResult.Content> と <xref:Microsoft.AspNetCore.Mvc.ContentResult.Content> ヘルパーを使用します。

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_about)]

前のコードで、返される `Content-Type` は `text/plain` です。 文字列を返すと、`text/plain` の `Content-Type` が送られます。

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_string)]

戻り値の型が複数あるアクションの場合は、`IActionResult` を返します。 たとえば、実行された操作の結果に基づいて、さまざまな HTTP 状態コードを返します。

## <a name="content-negotiation"></a>コンテンツ ネゴシエーション

クライアントにより [Accept ヘッダー](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html)が指定されると、コンテンツ ネゴシエーションが発生します。 ASP.NET Core で使用される既定の形式は [JSON](https://json.org/) です。 コンテンツ ネゴシエーションの説明を以下に示します。

* <xref:Microsoft.AspNetCore.Mvc.ObjectResult> によって実装されます。
* ヘルパー メソッドから返されるステータス コード固有のアクションの結果に組み込まれます。 アクションの結果のヘルパー メソッドは `ObjectResult` に基づいています。

モデルの型類が返されると、戻り値の型は `ObjectResult` になります。

次のアクション メソッドでは、ヘルパー メソッドの `Ok` と `NotFound` が使用されます。

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_search)]

別の形式が要求され、その要求された形式をサーバーが返せる場合を除き、JSON で書式設定された応答が返されます。 [Fiddler](https://www.telerik.com/fiddler) や [Postman](https://www.getpostman.com/tools) のようなツールは、`Accept` ヘッダーを設定して戻り値の形式を指定できます。 `Accept` に含まれる型がサーバーでサポートされるときは、その型が返されます。

既定では、ASP.NET Core では JSON のみがサポートされます。 既定値を変更しないアプリの場合は、クライアントの要求に関係なく、JSON で書式設定された応答が常に返されます。 フォーマッタの追加方法は次のセクションで説明します。

コントローラー アクションは POCO (単純な従来の CLR オブジェクト) を返すことができます。 POCO が返されると、ランタイムはそのオブジェクトをラップする `ObjectResult` を自動的に作成します。 クライアントは、書式設定されたシリアル化オブジェクトを取得します。 返されるオブジェクトが `null` の場合は、`204 No Content` という応答が返されます。

オブジェクトの型を返す:

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_alias)]

前のコードでは、有効な作成者エイリアスの要求に対して、`200 OK` という応答と作成者のデータが返されます。 無効なエイリアスの要求に対しては、`204 No Content` という応答が返されます。

### <a name="the-accept-header"></a>Accept ヘッダー

コンテンツ "*ネゴシエーション*" は、`Accept` ヘッダーが要求に含まれるときに発生します。 要求に Accept ヘッダーが含まれるとき、ASP.NET Core は次のように処理します。

* Accept ヘッダーのメディアの種類を優先順で列挙します。
* 指定された形式のいずれかで応答を生成できるフォーマッタを見つけようとします。

クライアントの要求を満たすことができるフォーマッタが見つからない場合は、ASP.NET Core は次のようにします。

* <xref:Microsoft.AspNetCore.Mvc.MvcOptions> が設定されていた場合は、`406 Not Acceptable` を返します。それ以外の場合は次のようにします。
* 応答を生成できる最初のフォーマッタを見つけようとします。

要求された形式に対応するフォーマッタが構成されていない場合、オブジェクトを書式設定できる最初のフォーマッタが使用されます。 要求に `Accept` ヘッダーが表示されない場合は、次のようになります。

* オブジェクトを処理できる最初のフォーマッタが使用されて、応答をシリアル化します。
* ネゴシエーションは発生しません。 サーバーが返す形式を決定します。

Accept ヘッダーに `*/*` が含まれる場合、`RespectBrowserAcceptHeader` が <xref:Microsoft.AspNetCore.Mvc.MvcOptions> で true に設定されていない限り、ヘッダーが無視されます。

### <a name="browsers-and-content-negotiation"></a>ブラウザーとコンテンツ ネゴシエーション

一般的な API クライアントとは異なり、Web ブラウザーは `Accept` ヘッダーを提供します。 Web ブラウザーでは、ワイルドカードを含む多くの形式が指定されます。 既定では、要求がブラウザーから送信されていることをフレームワークが検出すると、次のようになります。

* `Accept` ヘッダーは無視されます。
* 特に構成されていない限り、コンテンツは JSON で返されます。

このため、API を使用するときにブラウザー間でさらに一貫性の高いエクスペリエンスが提供されます。

ブラウザーの Accept ヘッダーを優先するようにアプリを構成するには、<xref:Microsoft.AspNetCore.Mvc.MvcOptions.RespectBrowserAcceptHeader> を `true` に設定します。

::: moniker range=">= aspnetcore-3.0"
[!code-csharp[](./formatting/3.0sample/StartupRespectBrowserAcceptHeader.cs?name=snippet)]
::: moniker-end
::: moniker range="< aspnetcore-3.0"
[!code-csharp[](./formatting/sample/StartupRespectBrowserAcceptHeader.cs?name=snippet)]
::: moniker-end

### <a name="configure-formatters"></a>フォーマッタの構成

追加の形式をサポートする必要があるアプリは、適切な NuGet パッケージを追加し、サポートを構成できます。 入力と出力で別々のフォーマッタがあります。 入力フォーマッタは、[モデル バインド](xref:mvc/models/model-binding)によって使用されます。 出力フォーマッタは、応答の書式設定に使用されます。 カスタム フォーマッタの作成の詳細については、[カスタム フォーマッタ](xref:web-api/advanced/custom-formatters)に関するページを参照してください。

::: moniker range=">= aspnetcore-3.0"

### <a name="add-xml-format-support"></a>XML 形式のサポートを追加する

<xref:System.Xml.Serialization.XmlSerializer> を使用して実装された XML フォーマッタは、<xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcBuilderExtensions.AddXmlSerializerFormatters*> を呼び出すことで構成できます。

[!code-csharp[](./formatting/3.0sample/Startup.cs?name=snippet)]

前のコードは、`XmlSerializer` を使用して結果をシリアル化します。

前のコードを使用する場合、コントローラー メソッドは要求の `Accept` ヘッダーに基づいて適切な形式を返す必要があります。

### <a name="configure-systemtextjson-based-formatters"></a>System.Text.Json ベースのフォーマッタを構成する

`System.Text.Json` ベースのフォーマッタの機能は、`Microsoft.AspNetCore.Mvc.JsonOptions.SerializerOptions` を使用して構成することができます。

```csharp
services.AddControllers().AddJsonOptions(options =>
{
    // Use the default property (Pascal) casing.
    options.SerializerOptions.PropertyNamingPolicy = null;

    // Configure a custom converter.
    options.SerializerOptions.Converters.Add(new MyCustomJsonConverter());
});
```

出力のシリアル化オプションは、アクションごとに `JsonResult` を使用して構成することができます。 次に例を示します。

```csharp
public IActionResult Get()
{
    return Json(model, new JsonSerializerOptions
    {
        options.WriteIndented = true,
    });
}
```

### <a name="add-newtonsoftjson-based-json-format-support"></a>Newtonsoft.Json ベースの JSON 形式のサポートを追加する

ASP.NET Core 3.0 より前、既定では、`Newtonsoft.Json` パッケージを使用して実装された JSON フォーマッタが使用されていました。 ASP.NET Core 3.0 以降、既定の JSON フォーマッタは `System.Text.Json` に基づいています。 `Newtonsoft.Json` ベースのフォーマッタと機能のサポートは、[Microsoft.AspNetCore.Mvc.NewtonsoftJson](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.NewtonsoftJson/) NuGet パッケージをインストールして `Startup.ConfigureServices` で構成することで利用できます。

[!code-csharp[](./formatting/3.0sample/StartupNewtonsoftJson.cs?name=snippet)]

一部の機能は `System.Text.Json` ベースのフォーマッタでうまく動作せず、`Newtonsoft.Json` ベースのフォーマッタの参照が必要となる場合があります。 アプリが以下の場合には、`Newtonsoft.Json` ベースのフォーマッタの使用を続けます。

* `Newtonsoft.Json` 属性を使用する。 たとえば、`[JsonProperty]` または `[JsonIgnore]` のようにします。
* シリアル化の設定をカスタマイズする。
* `Newtonsoft.Json` で提供される機能に依存する。
* `Microsoft.AspNetCore.Mvc.JsonResult.SerializerSettings` を構成する。 ASP.NET Core 3.0 より前は、`JsonResult.SerializerSettings`が `Newtonsoft.Json` に固有の `JsonSerializerSettings` のインスタンスを受け入れます。
* [OpenAPI](<xref:tutorials/web-api-help-pages-using-swagger>) ドキュメントを生成する。

`Newtonsoft.Json` ベースのフォーマッタの機能は、`Microsoft.AspNetCore.Mvc.MvcNewtonsoftJsonOptions.SerializerSettings` を使用して構成することができます。

```csharp
services.AddControllers().AddNewtonsoftJson(options =>
{
    // Use the default property (Pascal) casing
    options.SerializerSettings.ContractResolver = new DefaultContractResolver();

    // Configure a custom converter
    options.SerializerOptions.Converters.Add(new MyCustomJsonConverter());
});
```

出力のシリアル化オプションは、アクションごとに `JsonResult` を使用して構成することができます。 次に例を示します。

```csharp
public IActionResult Get()
{
    return Json(model, new JsonSerializerSettings
    {
        options.Formatting = Formatting.Indented,
    });
}
```

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

### <a name="add-xml-format-support"></a>XML 形式のサポートを追加する

XML の書式設定には、[Microsoft.AspNetCore.Mvc.Formatters.Xml](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Formatters.Xml/) NuGet パッケージが必要です。

<xref:System.Xml.Serialization.XmlSerializer> を使用して実装された XML フォーマッタは、<xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcBuilderExtensions.AddXmlSerializerFormatters*> を呼び出すことで構成できます。

[!code-csharp[](./formatting/sample/Startup.cs?name=snippet)]

前のコードは、`XmlSerializer` を使用して結果をシリアル化します。

前のコードを使用する場合、コントローラー メソッドは要求の `Accept` ヘッダーに基づいて適切な形式を返す必要があります。

::: moniker-end

### <a name="specify-a-format"></a>形式を指定する

応答の形式を制限するには、[`[Produces]`](xref:Microsoft.AspNetCore.Mvc.ProducesAttribute) フィルターを適用します。 ほとんどの[フィルター](xref:mvc/controllers/filters)と同様に、`[Produces]` をアクション、コントローラー、グローバル スコープに適用できます。

[!code-csharp[](./formatting/3.0sample/Controllers/WeatherForecastController.cs?name=snippet)]

前の [`[Produces]`](xref:Microsoft.AspNetCore.Mvc.ProducesAttribute) フィルターでは以下の処理が行われます。

* コントローラー内のすべてのアクションが、JSON で書式設定された応答を返すように強制します。
* 他のフォーマッタが構成されていて、クライアントが別の形式を指定した場合でも、JSON が返されます。

詳細については、[フィルター](xref:mvc/controllers/filters)に関するページを参照してください。

### <a name="special-case-formatters"></a>特殊なケースのフォーマッタ

一部の特殊なケースが組み込みのフォーマッタで実装されます。 既定では、戻り値の型 `string` は *text/plain* として書式設定されます (`Accept` ヘッダー経由で要求された場合は *text/html*)。 この動作は <xref:Microsoft.AspNetCore.Mvc.Formatters.TextOutputFormatter> を削除することで削除できます。 フォーマッタは `Configure` メソッドで削除します。 戻り値の型としてモデル オブジェクトをともなうアクションは、`null` を返すとき、`204 No Content` を返します。 この動作は <xref:Microsoft.AspNetCore.Mvc.Formatters.HttpNoContentOutputFormatter> を削除することで削除できます。 次のコードでは、`TextOutputFormatter` と `HttpNoContentOutputFormatter` が削除されます。

::: moniker range=">= aspnetcore-3.0"
[!code-csharp[](./formatting/3.0sample/StartupTextOutputFormatter.cs?name=snippet)]
::: moniker-end
::: moniker range="< aspnetcore-3.0"
[!code-csharp[](./formatting/sample/StartupTextOutputFormatter.cs?name=snippet)]
::: moniker-end

`TextOutputFormatter` がないと、戻り値の型 `string` は `406 Not Acceptable` を返します。 XML フォーマッタが存在するときは、`TextOutputFormatter` が削除された場合、戻り値の型 `string` が書式設定されます。

`HttpNoContentOutputFormatter` がない場合、構成されているフォーマッタを利用し、null オブジェクトが書式設定されます。 次に例を示します。

* JSON フォーマッタは、本文が `null` の応答を返します。
* XML フォーマッタは、属性 `xsi:nil="true"` が設定された空の XML 要素を返します。

## <a name="response-format-url-mappings"></a>応答形式の URL マッピング

クライアントは、URL の一部として特定の形式を要求できます。次に例を示します。

* クエリ文字列またはパスの一部。
* .xml または .json など形式固有のファイル拡張子の使用。

要求パスからのマッピングは、API で使用されるルートに指定する必要があります。 次に例を示します。

[!code-csharp[](./formatting/sample/Controllers/ProductsController.cs?name=snippet)]

前のルートを使用すると、要求された形式をオプションのファイル拡張子として指定できます。 [`[FormatFilter]`](xref:Microsoft.AspNetCore.Mvc.FormatFilterAttribute) 属性は `RouteData` に形式値がないか確認し、応答が作成されると、応答形式を適切なフォーマッタにマッピングします。

|           ルート        |             フォーマッタ              |
|------------------------|------------------------------------|
|   `/api/products/5`    |    既定の出力フォーマッタ    |
| `/api/products/5.json` | JSON フォーマッタ (構成される場合) |
| `/api/products/5.xml`  | XML フォーマッタ (構成される場合)  |
