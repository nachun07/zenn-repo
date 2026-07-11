---
title: "Azure Durable Functions入門｜非同期処理を「順番・並列・待機」でシンプルに書く方法（Python編）"
emoji: "⚡"
slug: "azure-durable-functions-nyumon"
type: "tech"
topics: ["azure", "durablefunctions", "python", "サーバーレス", "初心者"]
published: false
---

## はじめに

Azure Functionsは便利ですが、1回の呼び出しごとに完結する「ステートレス」な仕組みのため、以下のような処理を書こうとすると急に難しくなります。

- 複数の関数を「順番に」呼び出したい
- 複数の処理を「並列で」実行して、すべて終わったら次に進みたい（ファンアウト・ファンイン）
- 長時間かかる処理の進捗を管理したい
- 人の承認が来るまで処理を「待機」させたい

こうした場面で活躍するのが「**Durable Functions**」です。Azure Functionsの拡張機能として、状態を持った処理（ステートフルなワークフロー）を、キューやDBを自前で組まずに書けるようになります。

この記事では、Python版Durable Functionsのローカル環境構築から、実際に「順次処理」「並列処理（ファンアウト・ファンイン）」「外部イベント待機」「マルチエージェントオーケストレーション」を動かすところまで、実際に試した内容をもとに詳しく説明します。難しい前提知識は不要で、記事の通りに進めていけば、Pythonの非同期処理に馴染みがなくても1時間程度で一通り体験できます。

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
- 「順次処理」「並列処理」「待機」といった複雑なフローを、Pythonのジェネレーター構文（`yield`）を使ってシンプルに書ける
- リトライやタイムアウトの仕組みが標準で用意されている
- C#、JavaScript/TypeScript、Python、PowerShell、Javaに対応している
- ローカル開発〜Azureへのデプロイまで、通常のAzure Functionsとほぼ同じ手順で行える

Durable Functionsで使われる関数には、役割の異なる3種類があります。

| 種類 | 役割 |
|---|---|
| クライアント関数 | オーケストレーションを開始するきっかけとなる関数（HTTPトリガーなど） |
| オーケストレーター関数 | 処理全体の流れ（順番・並列・分岐）を定義する「指揮者」 |
| アクティビティ関数 | 実際の処理（DBアクセス、API呼び出しなど）を行う「実働部隊」 |

Python版では、オーケストレーター関数は`async def`ではなく、`def`＋`yield`を使ったジェネレーター関数として書くのが特徴です。これはリプレイの仕組み上、処理を一時停止・再開できる必要があるためです。

## 環境

この記事で想定している環境は以下の通りです。

| 項目 | 仕様 |
|---|---|
| OS | Windows / macOS / Linux（VS Codeが動く環境） |
| ランタイム | Python 3.11 |
| エディタ | Visual Studio Code |
| ツール | Azure Functions Core Tools、Azurite（ストレージエミュレーター） |
| ネットワーク | パッケージ取得時に必要（実行自体はローカル完結） |

Durable Functionsは内部で「実行履歴の保存先」としてAzure Storageを必要とします。ローカル開発では、実際にAzureのストレージアカウントを用意しなくても、エミュレーターの**Azurite**を使うことで無料かつオフライン相当で動作確認ができます。

また、この記事の例では以下のツールを使用します。

- Visual Studio Code + Python 拡張機能
- Azure Functions Core Tools
- Node.js（Azuriteのインストールに必要）
- curl（動作確認用）

Node.jsがインストールされていない場合は、公式サイト（https://nodejs.org）からLTS版をダウンロードしてインストールしてください。

## インストール手順

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

新しいフォルダを作成し、Pythonの仮想環境を用意してからFunctionsプロジェクトを初期化します。

```bash
mkdir DurableFunctionsSample
cd DurableFunctionsSample
python3 -m venv .venv
source .venv/bin/activate   # Windowsの場合は .venv\Scripts\activate
func init --worker-runtime python --model V2
```

### ステップ4：パッケージの追加

`requirements.txt`に以下を追記します。

```
azure-functions
azure-functions-durable
```

追記したら、仮想環境内でインストールします。

```bash
pip install -r requirements.txt
```

### ステップ5：ストレージ接続の設定確認

プロジェクト直下の`local.settings.json`が、Azuriteを見るように設定されているか確認します。

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "python"
  }
}
```

`UseDevelopmentStorage=true`と書いておくだけで、先ほど起動したAzuriteに自動的に接続してくれます。これでAzureの実アカウントを用意しなくても、ローカルだけで動作確認ができる状態になりました。

Python版のDurable Functions（V2プログラミングモデル）では、複数の関数を1つの`function_app.py`にまとめて、デコレーターで役割を指定していくスタイルが基本になります。この記事でもすべて`function_app.py`に追記していく形で進めます。

## 動作確認：まずは最小構成を動かす

いきなり複雑なパターンに入る前に、まずは1個だけアクティビティ関数を呼ぶ最小構成で、環境が正しく動いているか確認します。

`function_app.py`を以下の内容で作成します。

```python
import azure.functions as func
import azure.durable_functions as df

app = df.DFApp(http_auth_level=func.AuthLevel.ANONYMOUS)


@app.route(route="hello_start")
@app.durable_client_input(client_name="client")
async def hello_http_start(req: func.HttpRequest, client: df.DurableOrchestrationClient):
    instance_id = await client.start_new("hello_orchestrator")
    return client.create_check_status_response(req, instance_id)


@app.orchestration_trigger(context_name="context")
def hello_orchestrator(context: df.DurableOrchestrationContext):
    result = yield context.call_activity("say_hello", "Osaka")
    return result


@app.activity_trigger(input_name="name")
def say_hello(name: str) -> str:
    return f"こんにちは、{name}さん！"
```

プロジェクトを起動します。

```bash
func start
```

以下のようにローカルのエンドポイントが表示されれば起動成功です。

```
Functions:

        hello_http_start: [GET,POST] http://localhost:7071/api/hello_start

        hello_orchestrator: orchestrationTrigger

        say_hello: activityTrigger
```

別のターミナルからHTTPリクエストを送ってオーケストレーションを開始します。

```bash
curl -X POST http://localhost:7071/api/hello_start
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
  "name": "hello_orchestrator",
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

`function_app.py`に以下を追記します。

```python
@app.route(route="orders/{order_id}")
@app.durable_client_input(client_name="client")
async def order_http_start(req: func.HttpRequest, client: df.DurableOrchestrationClient):
    order_id = req.route_params.get("order_id")
    instance_id = await client.start_new("process_order_orchestrator", client_input=order_id)
    return client.create_check_status_response(req, instance_id)


@app.orchestration_trigger(context_name="context")
def process_order_orchestrator(context: df.DurableOrchestrationContext):
    order_id = context.get_input()

    validated = yield context.call_activity("validate_order", order_id)
    if not validated:
        return "注文内容が不正なため処理を中断しました"

    payment_result = yield context.call_activity("process_payment", order_id)
    shipped_result = yield context.call_activity("ship_order", payment_result)

    return f"注文処理完了: {shipped_result}"


@app.activity_trigger(input_name="order_id")
def validate_order(order_id: str) -> bool:
    return True  # 実際は在庫確認などのロジックが入る


@app.activity_trigger(input_name="order_id")
def process_payment(order_id: str) -> str:
    return f"注文{order_id}の決済完了"


@app.activity_trigger(input_name="payment_info")
def ship_order(payment_info: str) -> str:
    return f"{payment_info} → 発送手配済み"
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

`yield`でつなぐだけで、あたかも普通の同期処理のように順番に書けているのがわかります。それぞれのステップの結果は自動的に保存されるため、途中でホストが再起動しても続きから再開できます。

このパターンが向いているのは、EC注文処理のように「検証→決済→発送」のような、明確な順序依存があるワークフローです。

## パターン2：並列処理（ファンアウト・ファンイン）

複数の処理を並列で実行し、すべて完了したら結果をまとめるパターンです。「ファンアウト（Fan-out）」で処理を扇状に分散させ、「ファンイン（Fan-in）」で結果を集約する、という名前の由来通りの動きをします。

### コード例

`function_app.py`に以下を追記します。

```python
@app.route(route="images")
@app.durable_client_input(client_name="client")
async def images_http_start(req: func.HttpRequest, client: df.DurableOrchestrationClient):
    image_urls = [
        "https://example.com/img1.jpg",
        "https://example.com/img2.jpg",
        "https://example.com/img3.jpg",
    ]
    instance_id = await client.start_new("process_images_orchestrator", client_input=image_urls)
    return client.create_check_status_response(req, instance_id)


@app.orchestration_trigger(context_name="context")
def process_images_orchestrator(context: df.DurableOrchestrationContext):
    image_urls = context.get_input()

    # ファンアウト：並列でリサイズ処理を投げる
    tasks = [context.call_activity("resize_image", url) for url in image_urls]

    # ファンイン：すべての完了を待って結果をまとめる
    results = yield context.task_all(tasks)

    yield context.call_activity("save_thumbnail_list", results)

    return results


@app.activity_trigger(input_name="image_url")
def resize_image(image_url: str) -> str:
    # 実際はここで画像処理ライブラリなどを使ってリサイズする
    return f"{image_url} -> リサイズ完了"


@app.activity_trigger(input_name="results")
def save_thumbnail_list(results: list) -> None:
    # 実際はDBやストレージへの保存処理が入る
    pass
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

もしリスト内包表記を使わず、ループの中で1つずつ`yield`してしまうと、意図せず直列実行になってしまいます。並列にしたい場合は、まず`call_activity`のタスクのリストを作ってから、最後に`context.task_all`でまとめて待つのがポイントです。

```python
# NG: 直列実行になってしまう
results = []
for url in image_urls:
    result = yield context.call_activity("resize_image", url)
    results.append(result)

# OK: 並列実行になる
tasks = [context.call_activity("resize_image", url) for url in image_urls]
results = yield context.task_all(tasks)
```

画像のリサイズや、複数店舗の在庫を一括チェックするといった、それぞれが独立していて並列化できる処理に向いています。並列数が非常に多い場合（数千件など）は、一度に全部投げるのではなく、バッチに分割して段階的に処理する方が安全です。同時実行数が多すぎるとストレージのスロットリングに引っかかることがあります。

## パターン3：外部イベント待機（Human Interaction）

人間の承認など、外部からのイベントを待つパターンです。承認フローや、経費申請・休暇申請のワークフローなど、人が絡む業務プロセスの自動化に向いています。

### コード例

`function_app.py`に以下を追記します。

```python
from datetime import timedelta


@app.route(route="approvals")
@app.durable_client_input(client_name="client")
async def approval_http_start(req: func.HttpRequest, client: df.DurableOrchestrationClient):
    instance_id = await client.start_new("approval_orchestrator")
    return client.create_check_status_response(req, instance_id)


@app.route(route="approvals/{instance_id}/approve")
@app.durable_client_input(client_name="client")
async def approve_request(req: func.HttpRequest, client: df.DurableOrchestrationClient):
    instance_id = req.route_params.get("instance_id")
    await client.raise_event(instance_id, "ApprovalEvent", True)
    return func.HttpResponse("承認イベントを送信しました", status_code=200)


@app.orchestration_trigger(context_name="context")
def approval_orchestrator(context: df.DurableOrchestrationContext):
    yield context.call_activity("request_approval", None)

    timeout_task = context.create_timer(context.current_utc_datetime + timedelta(minutes=5))
    approval_task = context.wait_for_external_event("ApprovalEvent")

    winner = yield context.task_any([approval_task, timeout_task])

    if winner == approval_task and approval_task.result:
        timeout_task.cancel()
        return "承認されました"

    return "タイムアウトまたは却下されました"


@app.activity_trigger(input_name="input")
def request_approval(input) -> None:
    # 実際はメール送信やSlack通知などが入る
    pass
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

タイマーとイベント待機を`context.task_any`で組み合わせることで、「一定時間以内に承認がなければタイムアウト」といった処理も自然に書けます。

## パターン4：マルチエージェントオーケストレーション

近年、Durable Functionsは複数のAIエージェント（LLM呼び出し）を組み合わせる「マルチエージェントオーケストレーション」の基盤としても使われるようになっています。プロンプトチェイニングによる関数連鎖や、ファンアウト・ファンインによる並列化といった代表的なエージェントパターンを、これまで紹介してきたDurable Functionsのプログラミングモデルでそのまま実装できます。

### なぜDurable FunctionsがAIエージェントに向いているのか

LLM呼び出しは、通常のAPI呼び出しに比べて時間がかかり、レート制限やタイムアウトで失敗することも珍しくありません。複数のLLM呼び出しやツール実行、エージェント間のハンドオフを含む複雑なワークフローになるほど、途中で失敗する可能性は高くなります。

自前でリトライやチェックポイントの仕組みを組み込んでいない場合、30分かかった処理が1つのエージェント呼び出しの失敗によって最初からやり直しになってしまう、といった事態が起こり得ます。Durable Functionsを使うと、各エージェント呼び出しやツール呼び出しが自動的にチェックポイントされる耐久性のある操作として実行されるため、失敗した箇所から処理を再開でき、それまでに消費したLLMのトークンや処理時間を無駄にしません。

具体的には、以下のような特性がマルチエージェント構成と相性が良いポイントです。

- **チェックポイントによる再開性**: 個々のエージェント呼び出しをアクティビティ関数として実装すれば、途中で失敗しても最初からやり直す必要がない
- **ファンアウト・ファンインで並列化**: 複数の専門エージェントに同時に問い合わせて、結果をまとめて集約できる
- **外部イベント待機との組み合わせ**: エージェントの提案に対する人間の承認を挟む「Human-in-the-loop」も自然に実現できる
- **決定的なオーケストレーション**: どのエージェントをどの順番で呼んだかが実行履歴として残るため、デバッグや監査がしやすい

### コード例：複数の専門エージェントを並列に呼び出す

ここでは、1つのメインエージェントが調査を行った後、その結果を複数の翻訳エージェントに同時に渡して、並列に多言語へ翻訳するシンプルな例を紹介します。実際のLLM呼び出し部分は、Azure OpenAIなど任意のクライアントに置き換えて考えてください。

`function_app.py`に以下を追記します。

```python
@app.route(route="agent_workflow")
@app.durable_client_input(client_name="client")
async def agent_workflow_http_start(req: func.HttpRequest, client: df.DurableOrchestrationClient):
    question = req.params.get("question", "Durable Functionsとは何ですか？")
    instance_id = await client.start_new("agent_orchestration_workflow", client_input=question)
    return client.create_check_status_response(req, instance_id)


@app.orchestration_trigger(context_name="context")
def agent_orchestration_workflow(context: df.DurableOrchestrationContext):
    question = context.get_input()

    # 1. まずメインエージェントに調査・回答を行わせる（順次処理）
    answer = yield context.call_activity("call_research_agent", question)

    # 2. その結果を複数の翻訳エージェントに並列で渡す（ファンアウト）
    target_languages = ["en", "zh", "ko"]
    translation_tasks = [
        context.call_activity("call_translation_agent", {"text": answer, "lang": lang})
        for lang in target_languages
    ]

    # 3. すべての翻訳結果を集約する（ファンイン）
    translations = yield context.task_all(translation_tasks)

    return {
        "question": question,
        "answer": answer,
        "translations": dict(zip(target_languages, translations)),
    }


@app.activity_trigger(input_name="question")
def call_research_agent(question: str) -> str:
    # 実際はここでAzure OpenAIなどのLLMクライアントを呼び出す
    # response = openai_client.chat.completions.create(...)
    return f"「{question}」についての調査結果（ダミー応答）"


@app.activity_trigger(input_name="params")
def call_translation_agent(params: dict) -> str:
    text = params["text"]
    lang = params["lang"]
    # 実際はここで翻訳専用エージェント（LLM）を呼び出す
    return f"[{lang}] {text}"
```

### 動かしてみる

```bash
curl -X POST "http://localhost:7071/api/agent_workflow?question=Durable+Functionsとは"
```

ステータス確認用URLにアクセスすると、メインエージェントの回答と、3言語分の翻訳結果がまとまって返ってきます。

```json
{
  "runtimeStatus": "Completed",
  "output": {
    "question": "Durable Functionsとは",
    "answer": "「Durable Functionsとは」についての調査結果（ダミー応答）",
    "translations": {
      "en": "[en] 「Durable Functionsとは」についての調査結果（ダミー応答）",
      "zh": "[zh] 「Durable Functionsとは」についての調査結果（ダミー応答）",
      "ko": "[ko] 「Durable Functionsとは」についての調査結果（ダミー応答）"
    }
  }
}
```

この例のように、「メインエージェントの実行（順次処理）→複数の専門エージェントへの並列委譲（ファンアウト・ファンイン）→結果の集約」という流れを、これまで紹介したパターンの組み合わせだけで表現できます。エージェントの数が増えても、`target_languages`のリストに追加するだけで並列度を増やせる点も、通常のファンアウト・ファンインパターンと同じ感覚です。

### Human-in-the-loopとの組み合わせ

マルチエージェント構成では、エージェントが生成した計画やアウトプットを、実行前に人間がレビューしたいケースもよくあります。これは人間の承認を待つDurable Functionsのパターンと組み合わせることで、承認待ちの間に障害が発生してもオーケストレーションが失われず、承認が得られた時点から処理を再開できます。実装は、パターン3で紹介した`wait_for_external_event`を、エージェントによる計画立案のアクティビティ関数の後に挟むだけです。

```python
@app.orchestration_trigger(context_name="context")
def travel_plan_orchestrator(context: df.DurableOrchestrationContext):
    request = context.get_input()

    # エージェントに旅行プランを提案させる
    plan = yield context.call_activity("call_planner_agent", request)

    # 人間の承認を待つ
    approved = yield context.wait_for_external_event("PlanApproved")

    if not approved:
        return "プランが却下されました"

    # 承認後、予約エージェントに処理を委譲する
    booking_result = yield context.call_activity("call_booking_agent", plan)
    return booking_result
```

### 本格的に使う場合は専用の拡張機能もある

ここで紹介したのは、各エージェント呼び出しを単純なアクティビティ関数として実装する、最もシンプルな構成です。より本格的にマルチエージェントシステムを構築する場合、Microsoftは**Microsoft Agent Framework向けのDurable拡張機能**を提供しています。この拡張機能は、エージェントのセッション永続化、オーケストレーションやワークフローの進捗のチェックポイント、失敗からの自動復旧、分散ホストへのスケーリングを、エージェント側のロジックを変更せずに実現します。

エージェントごとに会話履歴を保持する永続セッション、複数の専門エージェントを決定的なワークフローの中で自動チェックポイント付きで調整するマルチエージェントオーケストレーション、人間の承認や長時間の待機に対応する長期実行ワークフローといった機能が用意されており、Azure Functions上でのサーバーレスホスティングにも対応しています。今回のようにまず自前のアクティビティ関数でパターンを理解したうえで、本番運用ではこうした専用拡張機能への移行を検討するとよいでしょう。

## オーケストレーター関数を書くときの注意点

先ほど触れた「リプレイ」の仕組みがあるため、オーケストレーター関数にはいくつか守るべき制約があります。これを破ると、リプレイのたびに結果が変わってしまい、意図しない動作やエラーにつながります。

### 非決定的なコードを書かない

`datetime.now()`や`uuid.uuid4()`、`random`モジュールなどをオーケストレーター関数内で直接使ってはいけません。リプレイのたびに違う値が生成されてしまうためです。代わりに、Durable Functionsが提供する決定的なAPIを使います。

```python
# NG: リプレイのたびに違う値になる
import datetime
import uuid
now = datetime.datetime.now()
id = uuid.uuid4()

# OK: 決定的な値が返る
now = context.current_utc_datetime
id = context.new_uuid()
```

### I/Oを直接行わない

DBアクセスやHTTP通信、ファイル操作などは、オーケストレーター関数内で直接行わず、必ずアクティビティ関数に委譲します。オーケストレーターはあくまで「流れ」を定義するだけの場所です。

### `time.sleep`ではなく`context.create_timer`を使う

通常の`time.sleep`を使うと、リプレイのたびに実際に待機してしまったり、正しく永続化されなかったりします。必ず`context.create_timer`を使いましょう（パターン3のコード参照）。

### async defではなくdef + yieldで書く

オーケストレーター関数は通常のPythonの`async def`関数ではなく、`def`＋`yield`を使ったジェネレーター関数として書きます。これは、Durable Task Frameworkがオーケストレーターの実行を細かく制御し、リプレイを実現するための仕組みです。`async/await`はクライアント関数（HTTPトリガーなど）側で使います。

### 無限ループに注意する

長時間稼働するオーケストレーションは、実行履歴がどんどん蓄積されていきます。履歴が大きくなりすぎるとパフォーマンスが劣化するため、`context.continue_as_new()`を使って定期的に履歴をリセットすることが推奨されています。

## リトライとエラーハンドリング

アクティビティ関数の呼び出しには、リトライポリシーを簡単に設定できます。

```python
from azure.durable_functions import RetryOptions

retry_options = RetryOptions(first_retry_interval_in_milliseconds=5000, max_number_of_attempts=3)

result = yield context.call_activity_with_retry("process_payment", retry_options, order_id)
```

これにより、外部APIの一時的な障害などに対して、自動的にリトライしてくれるようになります。

また`try-except`で例外をハンドリングして、途中のステップが失敗した際に前のステップを取り消す「補償トランザクション（Sagaパターン）」を書くこともできます。

```python
try:
    yield context.call_activity("process_payment", order_id)
except Exception:
    yield context.call_activity("cancel_order", order_id)
    return "決済に失敗したため注文をキャンセルしました"
```

## 通常のAzure Functionsとの比較

順次処理・並列処理・待機処理を「自前」で実装する場合と、Durable Functionsを使う場合の違いをまとめます。

| 項目 | 自前実装（キュー+DB） | Durable Functions |
|---|---|---|
| 実行順序の制御 | キューの順序やロックで自前管理 | `yield`を並べるだけ |
| 並列処理と集約 | 完了カウンタなどを自前でDB管理 | `context.task_all`で完結 |
| 途中経過の永続化 | 状態テーブルを自前設計 | 自動でイベント履歴に保存 |
| 再起動時の復旧 | 自前でリカバリロジックが必要 | リプレイで自動復旧 |
| タイムアウト・待機 | タイマーやスケジューラを自前構築 | `create_timer`で標準対応 |
| 学習コスト | 低い（普通の非同期処理の知識で足りる） | オーケストレーターの制約や`yield`構文を覚える必要あり |
| ローカル開発 | 特に制約なし | Azuriteのセットアップが必要 |

## メリット・デメリット

### Durable Functionsのメリット

**複雑なフローをシンプルなコードで表現できる**
順次処理・並列処理・待機処理といった複雑な非同期フローを、`yield`と`context.task_all`、`context.task_any`を組み合わせるだけで、あたかも同期処理のように書けます。キューやステートストアを自前で設計する必要がありません。

**状態管理を意識しなくていい**
実行履歴が自動的に保存されるため、途中でホストが再起動してもゼロからやり直しになりません。長時間実行される処理でも安心して書けます。

**リトライやタイムアウトが標準機能として使える**
`RetryOptions`や`create_timer`など、実務でよく必要になる機能が最初から用意されています。

**通常のAzure Functionsの知識がそのまま活きる**
アクティビティ関数やクライアント関数は、通常のPython版Azure Functionsとほぼ同じ書き方です。トリガーの種類も普段と同じものが使えます。

### Durable Functionsのデメリット

**オーケストレーター関数特有の制約がある**
「非決定的なコードを書けない」「I/Oを直接行えない」「`async def`ではなく`yield`を使う」といったルールがあり、慣れるまでは戸惑うポイントです。特にPythonに慣れているほど、`async/await`ではなく`yield`を使う独特の書き方に最初は違和感を覚えるかもしれません。

**ローカル開発にAzuriteが必要**
通常のAzure Functionsよりも、ローカル環境のセットアップに一手間かかります。

**デバッグがやや複雑**
リプレイの仕組みがあるため、オーケストレーター関数内にブレークポイントを置いても、リプレイのたびに同じ箇所で複数回止まることがあります。ログの読み方に慣れが必要です。

**課金体系の理解が必要**
Azureにデプロイする場合、通常のAzure Functionsの実行時間課金に加えて、Storageアカウントの操作回数に応じた課金も発生します。頻繁にタイマーを使うパターンなどでは、コスト設計に注意が必要です。

## 実際に使ってみて

実際にローカル環境（Azurite + Functions Core Tools + Python）で、上記4つのパターンをすべて動かしてみました。

順次処理は最も直感的で、`yield`を使うことさえ覚えてしまえば、普段の同期処理を書いている感覚のまま実装できました。特に迷うポイントはなく、すぐに動作確認まで進められました。

ファンアウト・ファンインは、最初リスト内包表記を使わずループの中で`yield`してしまい、並列になっていないことに気づかず少し時間を溶かしました。`context.task_all`にタスクのリストをまとめて渡す書き方に慣れれば、そのあとは問題なく書けるようになりました。

外部イベント待機は、`curl`でイベントを手動送信するテストがやや手間でしたが、実際のプロダクトでは承認ボタンを押した先でこのAPIを叩く形になるとイメージすると、業務フローへの応用がしやすいと感じました。

マルチエージェントオーケストレーションは、既存の順次処理・ファンアウト/ファンイン・外部イベント待機のパターンをそのまま組み合わせられる点が最大の発見でした。「エージェント呼び出し＝アクティビティ関数」と割り切れば、新しい概念をほとんど覚える必要がなく、既存のワークフロー資産をそのままLLM時代の構成に転用できると実感しました。

全体を通して、Pythonの通常の非同期処理（`async/await`）に慣れていると、オーケストレーター関数だけ`def`＋`yield`で書くことに最初は戸惑いましたが、「オーケストレーターの中だけは特別なルールがある」と割り切ってしまえば、あとはコードの見通しの良さに驚くはずです。特にファンアウト・ファンインは、自前実装だと状態管理だけでかなりのコード量になる処理が、数行で書けてしまうのが体感できました。

一方で、ローカルでのデバッグ体験は通常のAzure Functionsよりも一段階複雑で、Azuriteの起動を忘れて「なぜかストレージに接続できない」というエラーに何度か遭遇しました。作業を始める前にAzuriteが起動しているか確認する癖をつけておくと、無駄なつまずきを減らせると思います。

## まとめ

本記事で説明した内容をまとめます。

- 環境構築：Functions Core Tools + Azurite + Pythonの仮想環境でローカル完結の開発環境が作れる
- 順次処理：`yield`を並べるだけで、複数のアクティビティを順番に実行できる
- 並列処理（ファンアウト・ファンイン）：`context.task_all`で並列実行と結果集約がシンプルに書ける
- 外部イベント待機：`wait_for_external_event`と`create_timer`の組み合わせで、承認待ちのようなフローも書ける
- マルチエージェントオーケストレーション：エージェント呼び出しをアクティビティ関数として実装すれば、順次処理・ファンアウト/ファンイン・外部イベント待機の各パターンをそのまま組み合わせて、耐久性のあるマルチエージェントワークフローを構築できる
- 注意点：オーケストレーター関数には「非決定的な処理禁止」「I/O禁止」「`async def`ではなく`yield`」などの制約がある

Durable Functionsを使うことで、複雑な非同期処理のフローを、自前でキューやステートストアを組まずに、シンプルなコードで実現できます。Pythonの通常の非同期処理とは書き方が異なる部分に最初は戸惑うかもしれませんが、一度慣れてしまえば非常に強力な武器になってくれるはずです。マルチエージェント構成のように新しいユースケースが増えても、既存のパターンの組み合わせで対応できるのも大きな魅力です。

次回は実際にAzureにデプロイして、Application Insightsと組み合わせて監視するところまで試してみたいと思います。

## 参考リンク

- [Durable Functions の概要 - Microsoft Learn](https://learn.microsoft.com/ja-jp/azure/azure-functions/durable/durable-functions-overview)