---
title: "Azure Durable Functions入門｜非同期処理を「順番・並列・待機」でシンプルに書く方法"
emoji: "⚡"
slug: "azure-durable-functions-nyumon"
type: "tech"
topics: ["azure", "durablefunctions", "サーバーレス", "csharp", "初心者"]
published: true
---

## はじめに

Azure Functionsは便利ですが、1回の呼び出しごとに完結する「ステートレス」な仕組みのため、以下のような処理を書こうとすると急に難しくなります。

- 複数の関数を「順番に」呼び出したい
- 複数の処理を「並列で」実行して、すべて終わったら次に進みたい（ファンアウト・ファンイン）
- 長時間かかる処理の進捗を管理したい
- 人の承認が来るまで処理を「待機」させたい

こうした場面で活躍するのが「**Durable Functions**」です。Azure Functionsの拡張機能として、状態を持った処理（ステートフルなワークフロー）を、キューやDBを自前で組まずに書けるようになります。

この記事では、Durable Functionsのローカル環境構築から、実際に「順次処理」「並列処理（ファンアウト・ファンイン）」「外部イベント待機」を動かすところまで、実際に試した内容をもとに詳しく説明します。難しい前提知識は不要で、記事の通りに進めていけば、Azure Functionsを触ったことがある方なら1時間程度で一通り体験できます。

## Durable Functionsとは

Durable Functionsは、サーバーレス環境で**ステートフルな処理**を実現するためのAzure Functionsの拡張機能です。

通常のAzure Functionsは、リクエストが来て処理をして返す、それだけで終わりの「ステートレス」な仕組みです。前回の呼び出しの情報を覚えておくことはできません。

一方Durable Functionsを使うと、複数の関数呼び出しをまたいで状態（実行順序や途中経過、変数の値など）を保持できます。この状態管理は、開発者がDBやキューを意識してコードを書く必要がなく、フレームワーク側が自動的に面倒を見てくれます。

### 裏側の仕組み：イベントソーシングとリプレイ

Durable Functionsの動作を理解するうえで欠かせないのが「イベントソーシング」という考え方です。

オーケストレーター関数（後述）が実行されると、その実行履歴（どの関数を呼んだか、どんな結果が返ってきたか、いつタイマーをセットしたか）がAzure Storageに逐一記録されていきます。

そして、オーケストレーターが一時停止・再開するたびに、この履歴を最初から「リプレイ（再生）」することで現在の状態を復元します。例えば3つの処理を順番に呼ぶオーケストレーターがあったとして、2つ目の処理が終わった直後にホストが再起動したとします。すると次にそのインスタンスが動くときは、1つ目・2つ目の処理を（実際には再実行せず記録された結果を使って）高速にリプレイし、3つ目の処理から続きを実行します。

このリプレイの仕組みがあるからこそ、以下のような特性を持ちます。

- サーバーレス環境でも長時間（数時間〜数ヶ月単位）の処理を継続できる
- 途中でホストが落ちても、処理を最初からやり直さずに続きから再開できる
- スケールアウトしても状態が失われない

## Durable Functionsの特徴

- 複数の関数呼び出しをまたいで状態を保持できる
- 「順次処理」「並列処理」「待機」といった複雑なフローを、ほぼ同期処理のように直感的なコードで書ける
- リトライやタイムアウトの仕組みが標準で用意されている
- C#、JavaScript/TypeScript、Python、PowerShell、Javaに対応している
- ローカル開発〜Azureへのデプロイまで、通常のAzure Functionsとほぼ同じ手順で行える

Durable Functionsで使われる関数には、役割の異なる3種類があります。

| 種類 | 役割 |
|---|---|
| クライアント関数 | オーケストレーションを開始するきっかけとなる関数（HTTPトリガーなど） |
| オーケストレーター関数 | 処理全体の流れ（順番・並列・分岐）を定義する「指揮者」 |
| アクティビティ関数 | 実際の処理（DBアクセス、API呼び出しなど）を行う「実働部隊」 |

## 環境

この記事で想定している環境は以下の通りです。

| 項目 | 仕様 |
|---|---|
| OS | Windows / macOS / Linux（VS Codeが動く環境） |
| ランタイム | .NET 8（分離ワーカーモデル） |
| エディタ | Visual Studio Code |
| ツール | Azure Functions Core Tools、Azurite（ストレージエミュレーター） |
| ネットワーク | パッケージ取得時に必要（実行自体はローカル完結） |

Durable Functionsは内部で「実行履歴の保存先」としてAzure Storageを必要とします。ローカル開発では、実際にAzureのストレージアカウントを用意しなくても、エミュレーターの**Azurite**を使うことで無料かつオフライン相当で動作確認ができます。

また、この記事の例では以下のツールを使用します。

- Visual Studio Code + C# 拡張機能
- Azure Functions Core Tools
- Node.js（Azuriteのインストールに必要）
- curl（動作確認用）

Node.jsがインストールされていない場合は、公式サイト（https://nodejs.org）からLTS版をダウンロードしてインストールしてください。

## インストール手順

Durable Functionsの開発環境を整える方法は、大きく分けて「Azure Functions Core Toolsでプロジェクトを作る方法」と「Visual Studioのテンプレートから作る方法」の2つがあります。CLIに慣れている方には前者がおすすめです。

### ステップ1：Azure Functions Core Toolsのインストール

ターミナルを開いて、以下を実行します（npm経由の例）。

```bash
npm install -g azure-functions-core-tools@4 --unsafe-perm true
```

インストールが完了したら、バージョンを確認します。

```bash
func --version
```

以下のようにバージョン情報が表示されれば成功です。

```
4.x.xxxx
```

### ステップ2：Azuriteのインストール

ローカルのストレージエミュレーターであるAzuriteをインストールします。

```bash
npm install -g azurite
```

インストールが完了したら、任意のフォルダで起動しておきます。

```bash
mkdir azurite_data
azurite --silent --location ./azurite_data
```

以下のような出力が表示されれば、エミュレーターがバックグラウンドで起動しています。

```
Azurite Blob service is starting at http://127.0.0.1:10000
Azurite Blob service is successfully listening at http://127.0.0.1:10000
Azurite Queue service is starting at http://127.0.0.1:10001
Azurite Table service is starting at http://127.0.0.1:10002
```

このターミナルは動かしっぱなしにして、別のターミナルで以降の作業を進めます。

### ステップ3：プロジェクトの作成

新しいフォルダを作成し、Functionsプロジェクトを初期化します。

```bash
mkdir DurableFunctionsSample
cd DurableFunctionsSample
func init --worker-runtime dotnet-isolated
```

続けて、Durable Functionsの拡張パッケージを追加します。

```bash
dotnet add package Microsoft.Azure.Functions.Worker.Extensions.DurableTask
dotnet add package Microsoft.Azure.Functions.Worker.Extensions.DurableTask.AzureManaged
```

### ステップ4：ストレージ接続の設定確認

プロジェクト直下の`local.settings.json`が、Azuriteを見るように設定されているか確認します。

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet-isolated"
  }
}
```

`UseDevelopmentStorage=true`と書いておくだけで、先ほど起動したAzuriteに自動的に接続してくれます。これでAzureの実アカウントを用意しなくても、ローカルだけで動作確認ができる状態になりました。

## 動作確認：まずは最小構成を動かす

いきなり複雑なパターンに入る前に、まずは1個だけアクティビティ関数を呼ぶ最小構成で、環境が正しく動いているか確認します。

`Function1.cs`を以下の内容で作成します。

```csharp
using Microsoft.Azure.Functions.Worker;
using Microsoft.Azure.Functions.Worker.Http;
using Microsoft.DurableTask;
using Microsoft.DurableTask.Client;
using Microsoft.Extensions.Logging;

public static class HelloOrchestration
{
    [Function(nameof(HelloOrchestrator))]
    public static async Task<string> HelloOrchestrator(
        [OrchestrationTrigger] TaskOrchestrationContext context)
    {
        var result = await context.CallActivityAsync<string>(nameof(SayHello), "Osaka");
        return result;
    }

    [Function(nameof(SayHello))]
    public static string SayHello([ActivityTrigger] string name, FunctionContext executionContext)
    {
        return $"こんにちは、{name}さん！";
    }

    [Function("HelloHttpStart")]
    public static async Task<HttpResponseData> HttpStart(
        [HttpTrigger(AuthorizationLevel.Anonymous, "post")] HttpRequestData req,
        [DurableClient] DurableTaskClient client,
        FunctionContext executionContext)
    {
        string instanceId = await client.ScheduleNewOrchestrationInstanceAsync(
            nameof(HelloOrchestrator));

        return await client.CreateCheckStatusResponseAsync(req, instanceId);
    }
}
```

プロジェクトを起動します。

```bash
func start
```

以下のようにローカルのエンドポイントが表示されれば起動成功です。

```
Functions:

        HelloHttpStart: [POST] http://localhost:7071/api/HelloHttpStart

        HelloOrchestrator: orchestrationTrigger

        SayHello: activityTrigger
```

別のターミナルからHTTPリクエストを送ってオーケストレーションを開始します。

```bash
curl -X POST http://localhost:7071/api/HelloHttpStart
```

以下のようなレスポンスが返ってきます。

```json
{
  "id": "abc123...",
  "statusQueryGetUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123...",
  "sendEventPostUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123.../raiseEvent/{eventName}",
  "terminatePostUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123.../terminate"
}
```

この`statusQueryGetUri`にアクセスすると、処理の状態を確認できます。

```bash
curl "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123..."
```

```json
{
  "name": "HelloOrchestrator",
  "instanceId": "abc123...",
  "runtimeStatus": "Completed",
  "output": "こんにちは、Osakaさん！",
  "createdTime": "2026-07-11T04:00:00Z",
  "lastUpdatedTime": "2026-07-11T04:00:01Z"
}
```

`runtimeStatus`が`Completed`になっていて、`output`にアクティビティ関数の結果が返ってきていれば、環境構築は完璧です。ここまでできれば、あとは本題のパターンを試していくだけです。

## パターン1：順次処理（Function Chaining）

最もシンプルなパターンです。複数の処理を順番に実行し、前の処理の結果を次の処理に渡していきます。

### コード例

`SequentialOrchestration.cs`を作成します。

```csharp
using Microsoft.Azure.Functions.Worker;
using Microsoft.Azure.Functions.Worker.Http;
using Microsoft.DurableTask;
using Microsoft.DurableTask.Client;

public static class SequentialOrchestration
{
    [Function(nameof(ProcessOrderOrchestrator))]
    public static async Task<string> ProcessOrderOrchestrator(
        [OrchestrationTrigger] TaskOrchestrationContext context)
    {
        var orderId = context.GetInput<string>();

        var validated = await context.CallActivityAsync<bool>(nameof(ValidateOrder), orderId);
        if (!validated)
        {
            return "注文内容が不正なため処理を中断しました";
        }

        var paymentResult = await context.CallActivityAsync<string>(nameof(ProcessPayment), orderId);
        var shippedResult = await context.CallActivityAsync<string>(nameof(ShipOrder), paymentResult);

        return $"注文処理完了: {shippedResult}";
    }

    [Function(nameof(ValidateOrder))]
    public static bool ValidateOrder([ActivityTrigger] string orderId)
    {
        return true; // 実際は在庫確認などのロジックが入る
    }

    [Function(nameof(ProcessPayment))]
    public static string ProcessPayment([ActivityTrigger] string orderId)
    {
        return $"注文{orderId}の決済完了";
    }

    [Function(nameof(ShipOrder))]
    public static string ShipOrder([ActivityTrigger] string paymentInfo)
    {
        return $"{paymentInfo} → 発送手配済み";
    }

    [Function("OrderHttpStart")]
    public static async Task<HttpResponseData> HttpStart(
        [HttpTrigger(AuthorizationLevel.Anonymous, "post", Route = "orders/{orderId}")] HttpRequestData req,
        [DurableClient] DurableTaskClient client,
        string orderId)
    {
        string instanceId = await client.ScheduleNewOrchestrationInstanceAsync(
            nameof(ProcessOrderOrchestrator), orderId);

        return await client.CreateCheckStatusResponseAsync(req, instanceId);
    }
}
```

### 動かしてみる

```bash
curl -X POST http://localhost:7071/api/orders/1001
```

ステータス確認用のURLにアクセスすると、以下のように結果が返ってきます。

```json
{
  "runtimeStatus": "Completed",
  "output": "注文処理完了: 注文1001の決済完了 → 発送手配済み"
}
```

`await`でつなぐだけで、あたかも普通の同期処理のように順番に書けているのがわかります。それぞれのステップの結果は自動的に保存されるため、途中でホストが再起動しても続きから再開できます。

このパターンが向いているのは、EC注文処理のように「検証→決済→発送」のような、明確な順序依存があるワークフローです。

## パターン2：並列処理（ファンアウト・ファンイン）

複数の処理を並列で実行し、すべて完了したら結果をまとめるパターンです。「ファンアウト（Fan-out）」で処理を扇状に分散させ、「ファンイン（Fan-in）」で結果を集約する、という名前の由来通りの動きをします。

### コード例

`FanOutFanInOrchestration.cs`を作成します。

```csharp
using Microsoft.Azure.Functions.Worker;
using Microsoft.Azure.Functions.Worker.Http;
using Microsoft.DurableTask;
using Microsoft.DurableTask.Client;

public static class FanOutFanInOrchestration
{
    [Function(nameof(ProcessImagesOrchestrator))]
    public static async Task<List<string>> ProcessImagesOrchestrator(
        [OrchestrationTrigger] TaskOrchestrationContext context)
    {
        var imageUrls = context.GetInput<List<string>>();

        // ファンアウト：並列でリサイズ処理を投げる
        var tasks = new List<Task<string>>();
        foreach (var url in imageUrls)
        {
            tasks.Add(context.CallActivityAsync<string>(nameof(ResizeImage), url));
        }

        // ファンイン：すべての完了を待って結果をまとめる
        var results = await Task.WhenAll(tasks);

        await context.CallActivityAsync(nameof(SaveThumbnailList), results.ToList());

        return results.ToList();
    }

    [Function(nameof(ResizeImage))]
    public static string ResizeImage([ActivityTrigger] string imageUrl)
    {
        // 実際はここで画像処理ライブラリなどを使ってリサイズする
        return $"{imageUrl} -> リサイズ完了";
    }

    [Function(nameof(SaveThumbnailList))]
    public static void SaveThumbnailList([ActivityTrigger] List<string> results)
    {
        // 実際はDBやストレージへの保存処理が入る
    }

    [Function("ImagesHttpStart")]
    public static async Task<HttpResponseData> HttpStart(
        [HttpTrigger(AuthorizationLevel.Anonymous, "post", Route = "images")] HttpRequestData req,
        [DurableClient] DurableTaskClient client)
    {
        var imageUrls = new List<string>
        {
            "https://example.com/img1.jpg",
            "https://example.com/img2.jpg",
            "https://example.com/img3.jpg"
        };

        string instanceId = await client.ScheduleNewOrchestrationInstanceAsync(
            nameof(ProcessImagesOrchestrator), imageUrls);

        return await client.CreateCheckStatusResponseAsync(req, instanceId);
    }
}
```

### 動かしてみる

```bash
curl -X POST http://localhost:7071/api/images
```

ステータス確認用URLにアクセスすると、以下のように3件すべてが並列処理された結果が返ってきます。

```json
{
  "runtimeStatus": "Completed",
  "output": [
    "https://example.com/img1.jpg -> リサイズ完了",
    "https://example.com/img2.jpg -> リサイズ完了",
    "https://example.com/img3.jpg -> リサイズ完了"
  ]
}
```

もし`foreach`の中で1つずつ`await`してしまうと、意図せず直列実行になってしまいます。並列にしたい場合は、まず`Task`のリストを作ってから最後に`Task.WhenAll`でまとめて待つのがポイントです。

```csharp
// NG: 直列実行になってしまう
foreach (var url in imageUrls)
{
    var result = await context.CallActivityAsync<string>(nameof(ResizeImage), url);
}

// OK: 並列実行になる
var tasks = imageUrls.Select(url => context.CallActivityAsync<string>(nameof(ResizeImage), url));
var results = await Task.WhenAll(tasks);
```

画像のリサイズや、複数店舗の在庫を一括チェックするといった、それぞれが独立していて並列化できる処理に向いています。並列数が非常に多い場合（数千件など）は、一度に全部投げるのではなく、バッチに分割して段階的に処理する方が安全です。同時実行数が多すぎるとストレージのスロットリングに引っかかることがあります。

## パターン3：外部イベント待機（Human Interaction）

人間の承認など、外部からのイベントを待つパターンです。承認フローや、経費申請・休暇申請のワークフローなど、人が絡む業務プロセスの自動化に向いています。

### コード例

`ApprovalOrchestration.cs`を作成します。

```csharp
using Microsoft.Azure.Functions.Worker;
using Microsoft.Azure.Functions.Worker.Http;
using Microsoft.DurableTask;
using Microsoft.DurableTask.Client;

public static class ApprovalOrchestration
{
    [Function(nameof(ApprovalOrchestrator))]
    public static async Task<string> ApprovalOrchestrator(
        [OrchestrationTrigger] TaskOrchestrationContext context)
    {
        await context.CallActivityAsync(nameof(RequestApproval), null);

        using var cts = new CancellationTokenSource();
        var timeoutTask = context.CreateTimer(
            context.CurrentUtcDateTime.AddMinutes(5), cts.Token);
        var approvalTask = context.WaitForExternalEvent<bool>("ApprovalEvent");

        var winner = await Task.WhenAny(approvalTask, timeoutTask);

        if (winner == approvalTask && approvalTask.Result)
        {
            cts.Cancel();
            return "承認されました";
        }

        return "タイムアウトまたは却下されました";
    }

    [Function(nameof(RequestApproval))]
    public static void RequestApproval([ActivityTrigger] object input)
    {
        // 実際はメール送信やSlack通知などが入る
    }

    [Function("ApprovalHttpStart")]
    public static async Task<HttpResponseData> HttpStart(
        [HttpTrigger(AuthorizationLevel.Anonymous, "post", Route = "approvals")] HttpRequestData req,
        [DurableClient] DurableTaskClient client)
    {
        string instanceId = await client.ScheduleNewOrchestrationInstanceAsync(
            nameof(ApprovalOrchestrator));

        return await client.CreateCheckStatusResponseAsync(req, instanceId);
    }

    [Function("ApproveRequest")]
    public static async Task<HttpResponseData> ApproveRequest(
        [HttpTrigger(AuthorizationLevel.Anonymous, "post", Route = "approvals/{instanceId}/approve")] HttpRequestData req,
        [DurableClient] DurableTaskClient client,
        string instanceId)
    {
        await client.RaiseEventAsync(instanceId, "ApprovalEvent", true);

        var response = req.CreateResponse(System.Net.HttpStatusCode.OK);
        await response.WriteStringAsync("承認イベントを送信しました");
        return response;
    }
}
```

### 動かしてみる

まずオーケストレーションを開始します。

```bash
curl -X POST http://localhost:7071/api/approvals
```

レスポンスの`id`（インスタンスID）を控えておきます。この時点でステータスを確認すると、承認待ちで止まっているのがわかります。

```json
{
  "runtimeStatus": "Running"
}
```

5分以内に、控えておいたインスタンスIDを使って承認イベントを送ります。

```bash
curl -X POST http://localhost:7071/api/approvals/abc123.../approve
```

もう一度ステータスを確認すると、処理が先に進んでいるのがわかります。

```json
{
  "runtimeStatus": "Completed",
  "output": "承認されました"
}
```

もし5分以内にイベントを送らなかった場合は、タイムアウトして以下のような結果になります。

```json
{
  "runtimeStatus": "Completed",
  "output": "タイムアウトまたは却下されました"
}
```

タイマーとイベント待機を`Task.WhenAny`で組み合わせることで、「一定時間以内に承認がなければタイムアウト」といった処理も自然に書けます。

## オーケストレーター関数を書くときの注意点

先ほど触れた「リプレイ」の仕組みがあるため、オーケストレーター関数にはいくつか守るべき制約があります。これを破ると、リプレイのたびに結果が変わってしまい、意図しない動作やエラーにつながります。

### 非決定的なコードを書かない

`DateTime.Now`や`Guid.NewGuid()`、乱数などをオーケストレーター関数内で直接使ってはいけません。リプレイのたびに違う値が生成されてしまうためです。代わりに、Durable Functionsが提供する決定的なAPIを使います。

```csharp
// NG: リプレイのたびに違う値になる
var now = DateTime.Now;
var id = Guid.NewGuid();

// OK: 決定的な値が返る
var now = context.CurrentUtcDateTime;
var id = context.NewGuid();
```

### I/Oを直接行わない

DBアクセスやHTTP通信、ファイル操作などは、オーケストレーター関数内で直接行わず、必ずアクティビティ関数に委譲します。オーケストレーターはあくまで「流れ」を定義するだけの場所です。

### `Task.Delay`ではなく`context.CreateTimer`を使う

通常の`Task.Delay`を使うと、リプレイのたびに実際に待機してしまったり、正しく永続化されなかったりします。必ず`context.CreateTimer`を使いましょう（パターン3のコード参照）。

### 無限ループに注意する

長時間稼働するオーケストレーションは、実行履歴がどんどん蓄積されていきます。履歴が大きくなりすぎるとパフォーマンスが劣化するため、`context.ContinueAsNew()`を使って定期的に履歴をリセットすることが推奨されています。

## リトライとエラーハンドリング

アクティビティ関数の呼び出しには、リトライポリシーを簡単に設定できます。

```csharp
var retryOptions = new TaskOptions(
    new TaskRetryOptions(
        new RetryPolicy(
            maxNumberOfAttempts: 3,
            firstRetryInterval: TimeSpan.FromSeconds(5))));

var result = await context.CallActivityAsync<string>(
    nameof(ProcessPayment), orderId, retryOptions);
```

これにより、外部APIの一時的な障害などに対して、自動的にリトライしてくれるようになります。

また`try-catch`で例外をハンドリングして、途中のステップが失敗した際に前のステップを取り消す「補償トランザクション（Sagaパターン）」を書くこともできます。

```csharp
try
{
    await context.CallActivityAsync(nameof(ProcessPayment), orderId);
}
catch (TaskFailedException)
{
    await context.CallActivityAsync(nameof(CancelOrder), orderId);
    return "決済に失敗したため注文をキャンセルしました";
}
```

## 通常のAzure Functionsとの比較

順次処理・並列処理・待機処理を「自前」で実装する場合と、Durable Functionsを使う場合の違いをまとめます。

| 項目 | 自前実装（キュー+DB） | Durable Functions |
|---|---|---|
| 実行順序の制御 | キューの順序やロックで自前管理 | `await`を並べるだけ |
| 並列処理と集約 | 完了カウンタなどを自前でDB管理 | `Task.WhenAll`で完結 |
| 途中経過の永続化 | 状態テーブルを自前設計 | 自動でイベント履歴に保存 |
| 再起動時の復旧 | 自前でリカバリロジックが必要 | リプレイで自動復旧 |
| タイムアウト・待機 | タイマーやスケジューラを自前構築 | `CreateTimer`で標準対応 |
| 学習コスト | 低い（普通の非同期処理の知識で足りる） | オーケストレーターの制約を覚える必要あり |
| ローカル開発 | 特に制約なし | Azuriteのセットアップが必要 |

## メリット・デメリット

### Durable Functionsのメリット

**複雑なフローをシンプルなコードで表現できる**
順次処理・並列処理・待機処理といった複雑な非同期フローを、`await`と`Task.WhenAll`、`Task.WhenAny`を組み合わせるだけで、あたかも同期処理のように書けます。キューやステートストアを自前で設計する必要がありません。

**状態管理を意識しなくていい**
実行履歴が自動的に保存されるため、途中でホストが再起動してもゼロからやり直しになりません。長時間実行される処理でも安心して書けます。

**リトライやタイムアウトが標準機能として使える**
`RetryPolicy`や`CreateTimer`など、実務でよく必要になる機能が最初から用意されています。

**通常のAzure Functionsの知識がそのまま活きる**
アクティビティ関数やクライアント関数は、通常のAzure Functionsとほぼ同じ書き方です。トリガーの種類も普段と同じものが使えます。

### Durable Functionsのデメリット

**オーケストレーター関数特有の制約がある**
「非決定的なコードを書けない」「I/Oを直接行えない」といったルールがあり、慣れるまでは戸惑うポイントです。

**ローカル開発にAzuriteが必要**
通常のAzure Functionsよりも、ローカル環境のセットアップに一手間かかります。

**デバッグがやや複雑**
リプレイの仕組みがあるため、オーケストレーター関数内にブレークポイントを置いても、リプレイのたびに同じ箇所で複数回止まることがあります。ログの読み方に慣れが必要です。

**課金体系の理解が必要**
Azureにデプロイする場合、通常のAzure Functionsの実行時間課金に加えて、Storageアカウントの操作回数に応じた課金も発生します。頻繁にタイマーを使うパターンなどでは、コスト設計に注意が必要です。

## 実際に使ってみて

実際にローカル環境（Azurite + Functions Core Tools）で、上記3つのパターンをすべて動かしてみました。

順次処理は最も直感的で、普段async/awaitを書いている感覚のまま実装できました。特に迷うポイントはなく、すぐに動作確認まで進められました。

ファンアウト・ファンインは、最初`foreach`の中でうっかり`await`してしまい、並列になっていないことに気づかず少し時間を溶かしました。`Task.WhenAll`を使う書き方に慣れれば、そのあとは問題なく書けるようになりました。

外部イベント待機は、`curl`でイベントを手動送信するテストがやや手間でしたが、実際のプロダクトでは承認ボタンを押した先でこのAPIを叩く形になるとイメージすると、業務フローへの応用がしやすいと感じました。

全体を通して、オーケストレーター関数のルール（非決定的な処理を書かない、I/Oを直接行わない）さえ最初に押さえてしまえば、あとはコードの見通しの良さに驚くはずです。特にファンアウト・ファンインは、自前実装だと状態管理だけでかなりのコード量になる処理が、数行で書けてしまうのが体感できました。

一方で、ローカルでのデバッグ体験は通常のAzure Functionsよりも一段階複雑で、Azuriteの起動を忘れて「なぜかストレージに接続できない」というエラーに何度か遭遇しました。作業を始める前にAzuriteが起動しているか確認する癖をつけておくと、無駄なつまずきを減らせると思います。

## まとめ

本記事で説明した内容をまとめます。

- 環境構築：Functions Core Tools + Azuriteでローカル完結の開発環境が作れる
- 順次処理：`await`を並べるだけで、複数のアクティビティを順番に実行できる
- 並列処理（ファンアウト・ファンイン）：`Task.WhenAll`で並列実行と結果集約がシンプルに書ける
- 外部イベント待機：`WaitForExternalEvent`と`CreateTimer`の組み合わせで、承認待ちのようなフローも書ける
- 注意点：オーケストレーター関数には「非決定的な処理禁止」「I/O禁止」などの制約がある

Durable Functionsを使うことで、複雑な非同期処理のフローを、自前でキューやステートストアを組まずに、シンプルなコードで実現できます。最初はオーケストレーター関数特有のルールに戸惑うかもしれませんが、一度慣れてしまえば非常に強力な武器になってくれるはずです。

次回は実際にAzureにデプロイして、Application Insightsと組み合わせて監視するところまで試してみたいと思います。

## 参考リンク

- [Durable Functions の概要 - Microsoft Learn](https://learn.microsoft.com/ja-jp/azure/azure-functions/durable/durable-functions-overview)