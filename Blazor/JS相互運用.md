# Blazor の JavaScript相互運用

[ASP.NET Core Blazor JavaScript の相互運用性 (JS 相互運用) | Microsoft Learn](https://learn.microsoft.com/ja-jp/aspnet/core/blazor/javascript-interoperability/?view=aspnetcore-9.0)

概要

1. Visual StudioでBlazor Web Appプロジェクトを作成する
2. `wwwroot/javascript/interop.js` や `Components/Pages/XXX.razor.js` にファイルを置く
3. ~~`App.razor` にスクリプト読み込み `<script>` を追加する~~ 要らない？
4. `JS.InvokeAsync<IJSObjectReference>("import", "./javascript/interop.js");` でモジュールを読み込む
5. モジュールに対して `InvokeVoidAsync()` で JavaScript 関数を呼び出す

`wwwroot/javascript/interop.js`

```javascript
export function showAlert(message) {
    console.log(message)
    alert(message);
}
```

`App.razor`

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <base href="/" />
    <link rel="stylesheet" href="bootstrap/bootstrap.min.css" />
    <link rel="stylesheet" href="app.css" />
    <link rel="stylesheet" href="blazor-js-test.styles.css" />
    <link rel="icon" type="image/png" href="favicon.png" />
    <HeadOutlet />
</head>

<body>
    <Routes />
    <script src="_framework/blazor.web.js"></script>
    <script src="interop.js" type="module"></script>
</body>

</html>
```

`Components/Pages/XXX.razor`

```cs
@page "/"
@rendermode InteractiveServer
@inject IJSRuntime JS

<PageTitle>JSテスト</PageTitle>

<h1>JSテスト</h1>

<p>
    <button @onclick="ShowAlertAsync">Show alert1</button>
    <button @onclick="ShowAlert2Async">Show alert2</button>
</p>


@code {
    private IJSObjectReference? module;
    private IJSObjectReference? module2;

    private async Task ShowAlertAsync(MouseEventArgs e)
    {
        module = module ?? await JS.InvokeAsync<IJSObjectReference>("import", "./javascript/interop.js");
        await module.InvokeVoidAsync("showAlert", "Hello from Blazor!");
    }

    private async Task ShowAlert2Async(MouseEventArgs e)
    {
        module2 = module2 ?? await JS.InvokeAsync<IJSObjectReference>("import", "./Components/Pages/Home.razor.js");
        await module2.InvokeVoidAsync("showAlert", "Hello from Blazor!");
    }
}
```

![alt text](images/js_interop_sample.png)

検索したりCopilotに聞いたりすると、 `App.razor` や `index.html` や `_Host.cshtml` に `<script>` 要素の追加をしていたが、今回試した限りではこれなしでも JavaScript 関数を呼び出せた。.NETのバージョン違い？わからない。

考えられる理由としては、`import` 関数で動的にJavaScriptファイルを読み込んでいる扱いになっている？DevToolsの様子を見ると、ボタンを押したときに初めてツリーにファイルが表示される。

## Web Speech APIの利用

`Toolbelt.Blazor.SpeechRecognition` パッケージを利用するとBlazorアプリでWeb Speech APIによる文字起こしを追加できる。

- [Blazor WebAssembly で作った Web アプリ "snow catch" ゲームを、🎙️ボイスコマンド (音声認識) で操作できるようにする #Blazor - Qiita](https://qiita.com/jsakamoto/items/9378a345a96113319102)
- [NuGet Gallery | Toolbelt.Blazor.SpeechRecognition 1.0.0](https://www.nuget.org/packages/Toolbelt.Blazor.SpeechRecognition)

だいたい以下のようにして実行できた。

```cs
@page "/web-speech-recognition-package"
@rendermode InteractiveServer
@using Toolbelt.Blazor.SpeechRecognition
@inject SpeechRecognition SpeechRecognition

<h3>WebSpeechRecognitionPackage</h3>

<div style="width: 72px; height: 72px;">
    <button @onclick="StartStopListening">再生</button>
</div>

@foreach (var msg in messages) {
    <p>@msg</p>
}

@code {
    ...
    private List<string> messages = new List<string>();
    private int currentResultIndex = -1;

    protected override void OnInitialized()
    {
        base.OnInitialized();
        SpeechRecognition.Lang = "ja-JP";
        // 中間の結果を返す
        SpeechRecognition.InterimResults = true;
        // 継続的に結果を返す
        SpeechRecognition.Continuous = true;
        SpeechRecognition.Result += OnSpeechRecognized;
        SpeechRecognition.End += OnEndSpeechRecognition;
    }

    private void OnSpeechRecognized(object? sender, SpeechRecognitionEventArgs e)
    {
        var result = e?.Results?[e.ResultIndex];
        var item = result?.Items?.LastOrDefault();
        if (item == null) return;
        if (currentResultIndex == e.ResultIndex)
        {
            messages[currentResultIndex] = $"{e.ResultIndex}:{item.Transcript}";
        }
        else
        {
            currentResultIndex = e.ResultIndex;
            messages.Add($"{e.ResultIndex}:{item.Transcript}");
        }
        StateHasChanged();
    }

    ...
}
```

ただ、JS相互運用の挙動を考えると、サーバーサイドレンダリングで実行するのは効率が悪い(らしい)。  

1. Client:ボタンクリック
2. →Server:C#でイベントハンドラ処理、JSコード呼び出し
3. →Client:JSコード実行して文字取得、サーバーに処理を返す
4. →Server:取得した文字をレンダリング
5. →Client:文字差分を更新
6. →3→4→5を必要なだけ繰り返す

リアルタイム文字起こしにすると、チャンクを受け取るごとにサーバークライアント間で通信が発生する。  
パフォーマンスに影響するため、リアルタイム性が必要な場面ではWebAssemplyを利用したクライアントサイドレンダリングを検討したほうがいい。

Blazor Web Appテンプレートを利用すると、一部のコンポーネントのみをWebAssemplyとできるので、これを試したい。
