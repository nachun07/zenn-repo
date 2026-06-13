---
title: "Ollamaで月額0円のローカルLLM環境を構築してみた"
emoji: "🦙"
type: "tech"
topics:
  - ollama
  - llm
  - ai
  - macos
  - localai
published: false
---

## はじめに

ChatGPTは便利ですが、有料版は月額3000円必要です。また、無料版でも機能制限があります。加えて、ローカル環境で動かしたい、プライベートなデータを扱いたい、インターネット接続がない環境で使いたい、という場面もあります。

こうした場面で活躍するのが「ローカルLLM」です。自分のパソコンの中だけでAIモデルを動かす方法があります。その中でも、**Ollamaはセットアップが簡単**で、初心者向けです。

この記事では、MacにOllamaをインストールして、実際にAIモデルを動かすまでの流れを、実際に試した経験をもとに説明します。難しい知識は不要で、記事の通りに進めていけば、誰でも30分程度で環境が整います。

## ローカルLLMとは

LLMは「Large Language Model」の略で、大規模言語モデルを意味します。ChatGPTもLLMの一種です。

通常、ChatGPTを使うときは、入力内容がOpenAIのサーバーに送られて、そこで処理されます。一方、**ローカルLLMは、自分のパソコンの中だけでこの処理を完結させる方法**です。

### ローカルLLMのポイント

- インターネット接続がなくても動く
- 入力したテキストや会話内容がサーバーに送られない
- 実行速度はパソコンのスペックに左右される
- 月額費用がかからない

## Ollamaとは

**Ollamaは、ローカルLLMを簡単に実行するためのツール**です。複雑な設定が不要で、インストールしてコマンドを実行するだけで、LLMが動きます。

### Ollamaの特徴

- インストールが簡単（Homebrewで一発）
- 多くのオープンソースLLMに対応している
- CLIと、APIの両方で使える
- macOS、Linux、Windowsで動作する
- ダウンロードしたモデルは自動で管理される

Ollamaを通じて使えるモデルには、Meta社の「Llama3」、Alibaba社の「Qwen」、Mistral社の「Mistral」などがあります。これらはすべてオープンソースで、無料です。

## 環境

この記事で想定している環境は以下の通りです。

| 項目 | 仕様 |
| --- | --- |
| OS | macOS（Intel/Apple Silicon） |
| メモリ | 8GB以上推奨（4GBでも動作する場合がある） |
| ディスク空き | 最低10GB（モデルによって異なる） |
| ネットワーク | モデルダウンロード時に必要 |

特にメモリが重要です。Llama3などのモデルは、実行時にメモリを多く消費するため、できれば16GB以上が理想的です。

また、この記事の例では以下のツールを使用します。

- ターミナル（macOS標準）
- Homebrew（Macのパッケージマネージャー）
- Python3（API利用時）

Homebrewがインストールされていない場合は、以下を実行してください。

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## インストール手順

Ollamaのインストール方法は2つあります。GUIアプリケーションとしてインストールする方法と、ターミナルから Homebrewでインストールする方法です。初心者向けには、アプリケーションをダウンロードする方法をお勧めします。

### 方法1：公式サイトからアプリケーションをダウンロード（推奨）

#### ステップ1：公式サイトにアクセス

以下のURLにアクセスします。

```
https://ollama.ai
```

または検索エンジンで「Ollama」と検索して、公式サイトを開きます。

#### ステップ2：ダウンロード

公式サイトのダウンロードページに、「Download for macOS」ボタンがあります。クリックするだけでダウンロードが始まります。

Apple Silicon（M1/M2/M3チップ）とIntel Macの判別は自動的に行われるため、手動で選択する必要はありません。

![インストール画面](/images/ollama/Ollama-Download.png)

#### ステップ3：インストール

ダウンロードが完了すると、`Ollama.dmg`というファイルがダウンロードフォルダに保存されます。

このファイルをダブルクリックするとインストーラーが開きます。

インストール画面には、Ollamaのアイコンと「Applications」フォルダが表示されます。**Ollamaのアイコンを「Applications」フォルダへドラッグ&ドロップします**。

![アプリケーションドラッグ&ドロップ画面](/images/ollama/Ollama-dmg.png)

これだけでインストール完了です。

完了後、ダウンロードフォルダの`Ollama.dmg`は削除して問題ありません。

#### ステップ4：Ollamaアプリケーションを起動

Finderを開いて、左サイドバーから「アプリケーション」を選択します。

一覧の中から「Ollama」を探して、ダブルクリックで起動します。

初回起動時は以下のメッセージが表示される場合があります。

```
"Ollama"は、App Storeからダウンロードされていません。開いてもよろしいですか？
```

「開く」をクリックして進めます。

起動すると、メニューバー（画面上部）に🦙アイコンが表示されます。これでOllamaがバックグラウンドで動作しています。

メニューバーのOllamaアイコンをクリックすると、メニューが表示され、ステータスを確認できます。

### 方法2：Homebrewでインストール（ターミナル利用者向け）

ターミナルでコマンドを実行するのが得意な方は、こちらの方法も便利です。

#### ステップ1：Homebrewを使ったインストール

ターミナルを開いて、以下のコマンドを実行します。

```bash
brew install ollama
```

これで自動的にダウンロードとインストールが完了します。

#### ステップ2：バックグラウンドサービスとして起動

Ollamaをバックグラウンドサービスとして登録します。

```bash
brew services start ollama
```

起動状況を確認するには：

```bash
brew services list
```

以下のように表示されます。

```
NAME    STATUS      USER FILE
ollama  started     username ~/Library/LaunchAgents/homebrew.mxcl.ollama.plist
```

`started`と表示されていれば、Ollamaが起動しています。

#### ステップ3：インストール確認

バージョンを確認して、インストールが成功したか調べます。

```bash
ollama --version
```

バージョン情報が表示されれば、インストール成功です。

```
ollama version is x.x.x
```

### どちらの方法を選べばいい？

| 方法 | メリット | デメリット |
| --- | --- | --- |
| **アプリケーション** | GUIで簡単、メニューバーから操作可能 | ターミナルを使わない |
| **Homebrew** | ターミナルで自動管理、スクリプト化可能 | 基本的なコマンド知識が必要 |

初心者向けには、方法1（アプリケーション）をお勧めします。Ollamaが起動しているか確認も簡単で、いつでもメニューバーから制御できます。

### 動作確認

どちらの方法でインストールした場合でも、Ollamaが正常に動作しているか確認しましょう。

ターミナルを開いて、以下のコマンドを実行します。

```bash
curl http://localhost:11434/api/tags
```

以下のような応答が返ってきたら、Ollamaのシステムがバックグラウンドで正常に起動しています。

```json
{"models":[]}
```

`{"models":[]}`（空っぽ）と表示されて驚くかもしれませんが、これは**「まだパソコン内にAIモデルを一つもダウンロードしていない状態」を意味する正常な応答**です。

無事にOllamaとの通信ができていることが確認できました。これで準備完了です。次のステップに進んで、AIの頭脳となるモデルをダウンロードしましょう。

## モデル実行方法

Ollamaアプリケーションが起動している状態で、実際にLLMモデルを実行します。メニューバーのOllamaアイコンが表示されていることを確認してから進めてください。

### ステップ1：モデルをダウンロード

Ollamaで利用可能なモデルは、公式サイトで確認できます。ここでは、Meta社の「Llama3」を例に説明します。

ターミナルを開いて、以下を実行します。

```bash
ollama pull llama3
```

初回実行時は、モデル（約4GB）のダウンロードが始まります。インターネット速度によって異なりますが、数分から十数分かかる場合があります。

ダウンロード完了時には、以下のように表示されます。

```
pulling manifest 
pulling 6a0746a1ec1a: 100% ▕████████████████████████████████████ 4.7GB                         
pulling 4fa551d4f938: 100% ▕████████████████████████████████████ 12 KB                         
pulling 8ab4849b038c: 100% ▕████████████████████████████████████▏254 B                         
pulling 577073ffcc6c: 100% ▕████████████████████████████████████ 110 B                         
pulling 3f8eb4da87fa: 100% ▕████████████████████████████████████ 485 B                         
verifying sha256 digest 
writing manifest 
success
```

### ステップ2：モデルの一覧確認

ダウンロード済みのモデルを確認するには、以下を実行します。

```bash
ollama list
```

以下のように表示されます。

```
NAME            ID              SIZE      MODIFIED
llama3:latest   6a0746a1ec1a    4.7 GB    2 hours ago
```

### ステップ3：モデルの実行

ダウンロードしたモデルを実際に実行します。

```bash
ollama run llama3
```

初回実行時は、モデルのセットアップに数秒かかります。その後、プロンプトが表示されます。

```
>>> 
```

ここでテキストを入力できます。入力すると、モデルが考えて回答を返します。

例えば、日本語で質問を入力してみてください。モデルが日本語で応答します。

複数回の会話も可能です。そのまま続けて入力できます。

対話を終了するには、Ctrl+Dを押すか、以下を入力します。

```
>>> /bye
```

これで対話が終了します。

## アプリのチャット画面からモデルを実行する

Ollamaアプリを起動すると、ChatGPTのようなシンプルなチャット画面（GUI）が表示されます。ターミナル（黒い画面）にコマンドを打ち込む必要はなく、この画面から簡単にAIと会話を始めることができます。

### ステップ1：使用するモデルを選択する

画面中央にある入力欄（Send a message）の右側を見ると、現在選択されているAIモデルが表示されています。

![Ollamaアプリ画面](/images/ollama/Ollama-app.png)

ここをクリックすると、パソコンにダウンロードされているモデルの一覧が表示されます。今回は先ほどダウンロードした**「llama3」を選択**してください。

### ステップ2：メッセージを送信する

モデルが「llama3」になっていることを確認したら、入力欄に質問（例：「こんにちは！自己紹介をしてください。そして日本語で回答してください。」など）を入力します。

入力後、右端の「↑（送信）」ボタンを押すか、Enterキーを押すだけでローカルLLMとのチャットがスタートします。

## 別のモデルを試す

ほかのモデルも同じ方法で使えます。例えば、Alibaba社の「Qwen」は、日本語の処理に強いとされています。

```bash
ollama pull qwen:latest
ollama run qwen:latest
```

モデルのサイズは様々です。メモリが少ない場合は、より小さなモデルを選ぶことをお勧めします。例えば、「neural-chat」は軽量です。

```bash
ollama pull neural-chat:7b
ollama run neural-chat:7b
```

## CLIでの使用例

Ollamaはコマンドラインでも利用できます。アプリケーションがメニューバーで起動している状態で、ターミナルからコマンドを実行します。スクリプトから呼び出す際に便利です。

### 基本的な使い方

以下のコマンドで、対話なしで一度だけ質問に答えます。

```bash
ollama run llama3 "日本の首都はどこですか"
```

モデルが回答を返して、すぐにコマンドが終了します。

```
日本の首都は東京です。東京は日本の中心的な都市であり、政治、経済、文化の中心地として機能しています。
```

### ファイルから入力する

テキストファイルに入力を保存して、それをモデルに渡すこともできます。

まず、ファイルを作成します。

```bash
cat > question.txt << 'EOF'
以下のテキストを日本語で要約してください。

Artificial Intelligence (AI) is the simulation of human intelligence processes by computer systems. AI has applications in various fields including healthcare, finance, education, and entertainment. Machine learning, a subset of AI, enables systems to learn and improve from experience without being explicitly programmed.
EOF
```

このファイルをモデルに渡します。

```bash
cat question.txt | ollama run llama3
```

以下のような要約が返されます。

```
このテキストは、人工知能(AI)について説明しています。AIはコンピュータシステムが人間の知能プロセスをシミュレートしたものであり、医療、金融、教育、エンターテインメントなど様々な分野で応用されています。AIの部分集合である機械学習は、システムが明示的にプログラムされなくても、経験から学習し改善できるようにします。
```

### 複数の質問を連続実行する

以下のスクリプトを作成して、複数の質問を連続実行します。

```bash
cat > questions.txt << 'EOF'
質問1：Pythonとは何ですか
質問2：JavaScriptの主な用途は
質問3：ローカルLLMのメリットは
EOF
```

ターミナルで以下を実行します。

```bash
while IFS= read -r question; do
  echo "質問: $question"
  echo "$question" | ollama run llama3
  echo "---"
done < questions.txt
```

各質問に対してモデルが回答を返します。

## API（Python）での利用

Pythonからもモデルを利用できます。これにより、Pythonアプリケーション内にAI機能を組み込めます。

### 前提：Ollamaアプリケーションの起動

APIを使う場合、Ollamaアプリケーションは常に起動していないといけません。メニューバーのOllamaアイコンが表示されていることを確認してください。

### Python環境の準備

Pythonで必要なライブラリをインストールします。Ollamaとの通信には、`requests`ライブラリを使用します。

```bash
pip install requests
```
※ Macの環境によっては、直接 pip install するとエラーが出る場合があります。その場合は、プロジェクト用のフォルダを作成し、仮想環境（venv）を立ち上げてから実行してください。
### 基本的な使い方

新しいファイル `ollama_test.py` を作成します。

```python
import requests
import json

def ask_ollama(prompt, model="llama3"):
    """
    Ollamaに質問して回答を得る
    """
    url = "http://localhost:11434/api/generate"
    
    payload = {
        "model": model,
        "prompt": prompt,
        "stream": False
    }
    
    response = requests.post(url, json=payload)
    
    if response.status_code == 200:
        result = response.json()
        return result["response"]
    else:
        return f"エラー: {response.status_code}"

# 実行例
if __name__ == "__main__":
    question = "Pythonプログラミング言語について、100語程度で説明してください。"
    answer = ask_ollama(question)
    print(f"質問: {question}")
    print(f"回答: {answer}")
```

実行します。

```bash
python ollama_test.py
```

以下のような出力が得られます。

```
質問: Pythonプログラミング言語について、100語程度で説明してください。
回答: Pythonは、1991年にGuido van Rossum氏によって開発された、シンプルで読みやすい構文を持つ高級プログラミング言語です。データ分析、Web開発、機械学習など、多様な分野で広く利用されています。初心者にも学習しやすく、豊富なライブラリが利用可能です。世界中の開発者に愛用されており、技術分野での重要な言語となっています。
```

### ストリーミング対応版

長い回答の場合、ストリーミング機能を使うと、回答が返されるに従い少しずつ表示できます。

```python
import requests
import json

def ask_ollama_stream(prompt, model="llama3"):
    """
    Ollamaに質問して、ストリーミングで回答を得る
    """
    url = "http://localhost:11434/api/generate"
    
    payload = {
        "model": model,
        "prompt": prompt,
        "stream": True
    }
    
    response = requests.post(url, json=payload, stream=True)
    
    if response.status_code == 200:
        for line in response.iter_lines():
            if line:
                chunk = json.loads(line)
                print(chunk["response"], end="", flush=True)
        print()  # 最後に改行
    else:
        print(f"エラー: {response.status_code}")

# 実行例
if __name__ == "__main__":
    question = "日本の歴史を簡潔に説明してください。"
    ask_ollama_stream(question)
```

実行します。

```bash
python ollama_test_stream.py
```

回答がリアルタイムで表示されます。

### より実践的な例：会話の履歴管理

複数のやり取りを保持する例です。チャット履歴を管理する場合は、`/api/chat`エンドポイントを使用するのが推奨されます。

```python
import requests
import json

class OllamaChat:
    def __init__(self, model="llama3"):
        self.model = model
        self.url = "http://localhost:11434/api/chat"
        self.messages = []
    
    def send(self, user_message):
        """
        ユーザーメッセージを送信して、モデルからの回答を得る
        """
        # メッセージをチャット形式で追加
        self.messages.append({
            "role": "user",
            "content": user_message
        })
        
        payload = {
            "model": self.model,
            "messages": self.messages,
            "stream": False
        }
        
        response = requests.post(self.url, json=payload)
        
        if response.status_code == 200:
            result = response.json()
            assistant_message = result["message"]["content"]
            
            # モデルのレスポンスを履歴に追加
            self.messages.append({
                "role": "assistant",
                "content": assistant_message
            })
            
            return assistant_message
        else:
            return f"エラー: {response.status_code}"
    
    def reset(self):
        """
        会話履歴をリセット
        """
        self.messages = []

# 使用例
if __name__ == "__main__":
    chat = OllamaChat()
    
    print("=== Ollama チャットボット ===")
    print("質問を入力してください（'exit'で終了）")
    print()
    
    while True:
        user_input = input("あなた: ").strip()
        
        if user_input.lower() == "exit":
            print("チャットを終了します。")
            break
        
        if not user_input:
            continue
        
        response = chat.send(user_input)
        print(f"アシスタント: {response}\n")
```

実行して、対話を進めます。

```bash
python ollama_chat.py
```

```
=== Ollama チャットボット ===
質問を入力してください（'exit'で終了）

あなた: こんにちは。あなたは何ですか。
アシスタント: こんにちは。私はAI言語モデルで、様々な質問に答えたり、情報を提供したりすることができます。ローカルのOllamaを通じて実行されているLlama3モデルです。

あなた: Pythonについて教えてください。
アシスタント: Pythonはシンプルで読みやすい構文を持つプログラミング言語です。データ分析、Web開発、機械学習など...

あなた: exit
チャットを終了します。
```

## ChatGPTとの比較

Ollamaを使ったローカルLLMとChatGPTの主な違いをまとめます。

| 項目 | ローカルLLM(Ollama) | ChatGPT（無料版） | ChatGPT Plus（有料版） |
| --- | --- | --- | --- |
| 費用 | 無料 | 無料 | 月額3000円 |
| 機能制限 | なし | あり（使用回数制限） | 少ない |
| プライバシー | データがローカルに留まる | データがサーバーに送られる | データがサーバーに送られる |
| インターネット | 不要（起動後） | 必須 | 必須 |
| 実行速度 | パソコンのスペック依存 | 速い | 速い（優先処理） |
| 回答の質 | モデルによる（やや劣る） | 高い | より高い |
| カスタマイズ | ファインチューニングで可能 | 不可 | API利用のみ |
| 日本語対応 | モデルによる | 優秀 | 優秀 |
| セットアップ | 数十分 | 登録後すぐ | 登録＋決済後すぐ |

## メリット・デメリット

### ローカルLLMのメリット

1. **完全無料で無制限利用**
   ローカルLLMは0円で、使用回数に制限がありません。ChatGPTは無料版でも使用回数に制限がありますし、有料版は月額20ドル必要です。開発やビジネス用途で、頻繁に使う場合はローカルLLMが経済的です。

2. **完全なプライベート処理**
   データが自分のパソコン内に留まり、サーバーに送られません。機密情報、個人情報、企業秘密を扱う際に安心です。

3. **カスタマイズとファインチューニングが可能**
   オープンソースモデルなので、自分のデータで再学習させるなど、カスタマイズが自由です。ChatGPT APIではこうしたカスタマイズは制限されています。

4. **インターネット接続が不要**
   一度モデルをダウンロードすれば、オフラインでも使用できます。オフィスネットワークが制限されている環境でも動作します。

5. **API化が容易**
   Pythonやほかのプログラムから簡単に統合できます。

### ローカルLLMのデメリット

1. **回答の品質**
   ChatGPTと比較して、回答の質がやや劣る場合があります。特に複雑な推論や、最新の情報が必要な場合です。

2. **パソコンのスペック依存**
   実行速度やメモリ使用量がパソコンのスペックに大きく左右されます。低スペックマシンでは動作が遅い可能性があります。

3. **モデルのサイズ**
   高性能なモデルほど、ダウンロードサイズが大きく、メモリを消費します。

4. **セットアップの手間**
   初回セットアップに数十分から数時間かかる場合があります。

5. **日本語対応の差**
   モデルによって日本語対応の品質が異なります。Llama3は基本的な日本語は理解していますが、複雑な文脈や専門用語にはやや弱い傾向があります。より自然な日本語対話が必要な場合は、QwenやELYZAなどの日本語特化モデルの利用をお勧めします。

6. **常時起動の必要性**
   APIとして使う場合、Ollamaを常に起動しておく必要があります。

## 実際に使ってみて

実際にMacBook Pro（M5チップ）で試してみました。

Llama3は4.7GBのダウンロードで、ダウンロード自体は5分程度で終わりました。セットアップは簡単で、インストール後すぐに使える状態になります。

対話モード、CLIコマンド、APIの3つの方法すべてが問題なく動作しました。特にPythonのAPIでは、Ollamaをバックグラウンドで実行しておくだけで、簡単にプログラムに組み込めました。

日本語の対応は、Llama3は基本的な日本語は理解していますが、複雑な文脈や専門用語にはやや弱い傾向があります。より自然な日本語対話が必要な場合は、QwenやELYZAなどの日本語特化モデルの利用をお勧めします。

回答速度は、ChatGPTほど高速ではありませんが、待てないほど遅いわけではありません。数秒から数十秒で回答が返ってきます。

何度も試した結果、個人的には以下の使い方がお勧めです。

- プライベートなデータを扱う場合や、使用回数が多い場合はローカルLLM
- 高い精度が重要な場合、最新情報が必要な場合、または試験的に使う場合はChatGPT無料版
- 毎日業務で使うなど、本格的に利用する場合はChatGPT Plusと使い分け
- 開発時のプロトタイピングはローカルLLM（コスト0だから試しやすい）

## まとめ

Ollamaを使うことで、完全にプライベートで無制限にローカルLLM環境を構築できます。インストールは簡単で、Macユーザーであれば30分程度で環境が整います。

本記事で説明した内容をまとめます。

1. **インストール**：Homebrewで簡単にインストール可能
2. **モデル実行**：`ollama pull`と`ollama run`ですぐに使用可能
3. **CLI利用**：コマンドラインから直接、またはスクリプトから呼び出し可能
4. **API利用**：Pythonなどのプログラムから容易に統合可能
5. **カスタマイズ**：複数のオープンソースモデルから選択可能

メリットは、プライベート性、費用、カスタマイズの自由度です。デメリットは、回答の質と実行速度がパソコンのスペック依存であることです。

ローカルLLMとChatGPTは競合ではなく、補完する関係です。用途に応じて使い分けることで、より効率的にAI技術を活用できます。

最初は試し的に使ってみて、自分のワークフローに合っていればそのまま本格導入するといいでしょう。記事の手順に従えば、誰でも簡単に始められます。

ぜひ一度、ローカルLLM環境を構築してみてください。