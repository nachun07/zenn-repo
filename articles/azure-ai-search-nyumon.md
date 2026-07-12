---
title: "Azure AI Search入門 〜検索の基本からベクトル検索・RAG連携まで徹底解説〜"
emoji: "🔍"
type: "tech"
topics: ["azure", "azureaisearch", "search", "openai", "rag"]
published: false
---

# はじめに

Microsoft Azureが提供する検索サービス「**Azure AI Search**」（旧称：Azure Cognitive Search）について、基礎概念からハンズオンでの構築手順、そして昨今注目されている生成AI（RAG: Retrieval Augmented Generation）との連携方法まで、まとめて解説します。

「全文検索エンジンを自前で構築するのは大変そう」「社内文書やFAQを賢く検索できる仕組みを作りたい」「ChatGPTのようなAIに自社データを参照させたい（RAG）」といったニーズを持つ方に向けて、Azure AI Searchの全体像を掴んでもらうことをゴールにしています。

対象読者は以下のような方を想定しています。

- Azureをある程度触ったことがあるエンジニア
- 検索機能を自作するか、マネージドサービスを使うか検討している方
- Azure OpenAI ServiceとあわせてRAGシステムを構築したい方
- Elasticsearchなど他の検索エンジンとの違いを知りたい方

それでは早速見ていきましょう。

# Azure AI Searchとは何か

Azure AI Searchは、Microsoftが提供するフルマネージドの検索サービスです。もともとは「Azure Search」という名前でスタートし、その後AIによる文書の解析機能（AIエンリッチメント）が強化されたことで「Azure Cognitive Search」に改称、そして2023年以降のベクトル検索やセマンティック検索機能の強化に伴い、現在の「Azure AI Search」という名称に落ち着いています。

このサービスの本質は、**構造化データと非構造化データの両方を高速に検索できるインデックス（索引）を、インフラ管理なしで構築・運用できる**という点にあります。ElasticsearchやApache Solrのような全文検索エンジンをイメージするとわかりやすいですが、それらをゼロから構築・スケーリング・運用するのは決して簡単ではありません。Azure AI Searchはその複雑さをマネージドサービスとして肩代わりしてくれます。

## Azure AI Searchが解決する課題

一般的なアプリケーション開発において、検索機能の実装には以下のような課題が付きまといます。

1. **転置インデックスの構築・維持コスト**: 全文検索を高速に行うには専用のデータ構造（転置インデックス）が必要ですが、これを自前で実装するのは非常に手間がかかります。
2. **表記ゆれ・言語処理への対応**: 日本語であれば形態素解析、英語であればステミングなど、言語ごとの前処理が必要です。
3. **スケーラビリティ**: データ量やクエリ数が増えた際にスケールアウトできる設計が求められます。
4. **非構造化データの扱い**: PDFや画像、Officeファイルなどからテキストを抽出し、検索可能な形に変換する処理が必要です。
5. **意味的な検索（セマンティック検索）**: キーワードの完全一致だけでなく、「意味が近いもの」を検索したいというニーズが増えています。

Azure AI Searchは、これらすべてに対してマネージドな解決策を提供します。特に4番目と5番目については、AIを活用した独自の強みを持っています。

# Azure AI Searchの主な特徴

## 1. フルマネージドな全文検索エンジン

インフラのプロビジョニングやパッチ適用、スケーリングといった運用作業をAzureが肩代わりしてくれます。開発者はインデックスのスキーマ設計とデータ投入に集中できます。

## 2. AIエンリッチメント（Cognitive Skills）

Azure AI Search最大の特徴の一つが「AIエンリッチメントパイプライン」です。これはAzure AI services（旧Cognitive Services）の各種AI機能（OCR、画像解析、翻訳、固有表現抽出など）を検索インデックス構築の過程に組み込める仕組みです。

例えば、大量のスキャンされたPDF文書があるとします。通常であればテキスト情報がなく検索できませんが、AzureのOCRスキルを使えば、画像からテキストを抽出し、それをそのまま検索可能なフィールドとしてインデックスに格納できます。

主なスキルの例:

- **OCR**: 画像やスキャン文書からテキストを抽出
- **画像分析（Image Analysis）**: 画像からタグやキャプションを自動生成
- **エンティティ認識**: 人名・組織名・場所などの固有表現を抽出
- **キーフレーズ抽出**: 文書の要点となるキーワードを抽出
- **言語検出・翻訳**: 多言語文書への対応
- **感情分析**: テキストのポジ・ネガ判定
- **カスタムスキル**: Azure Functionsなどを使って独自の処理を組み込むことも可能

## 3. ベクトル検索（Vector Search）

2023年以降に大きく強化された機能で、テキストや画像を埋め込みベクトル（Embedding）に変換し、そのベクトル空間上での類似度検索を行えます。これにより「意味的に近い」文書を検索できるようになり、キーワードが完全一致しなくても関連性の高い結果を返せます。

Azure OpenAI Serviceの`text-embedding-ada-002`や`text-embedding-3-large`などのEmbeddingモデルと組み合わせて使うのが一般的です。

## 4. セマンティックランカー（Semantic Ranker）

検索結果を、機械学習モデルによって「意味的な関連性」の観点から再ランキングする機能です。従来のキーワードベースのスコアリング（BM25など）に加えて、Bing検索でも使われている言語理解技術を活用し、より人間の感覚に近い検索結果順位を実現します。

## 5. ハイブリッド検索

キーワード検索（全文検索）とベクトル検索を組み合わせた「ハイブリッド検索」もサポートしています。両方のスコアを組み合わせることで、キーワードの正確性とセマンティックな関連性の両方を活かした検索が可能です。RAGシステムを構築する際は、このハイブリッド検索+セマンティックランカーの組み合わせが推奨されるパターンになっています。

## 6. さまざまなデータソースとの統合

Azure Blob Storage、Azure SQL Database、Cosmos DB、Azure Table Storageなど、多様なAzureサービスをデータソースとしてインデクサー経由で自動的に取り込むことができます。

# 基本的なアーキテクチャと用語

Azure AI Searchを理解するうえで欠かせない基本概念を整理します。

## 検索サービス（Search Service）

Azure AI Searchのリソースそのものです。Azureポータル上で作成する際に、リージョンや価格レベル（SKU）を指定します。1つの検索サービスの中に複数のインデックスを作成できます。

## インデックス（Index）

検索対象となるデータの「スキーマ（構造）」を定義したものです。リレーショナルデータベースの「テーブル」に近いイメージです。インデックスにはフィールド（列）を定義し、それぞれのフィールドに対して「検索可能か」「フィルタ可能か」「ソート可能か」「ファセット可能か」といった属性を設定します。

## ドキュメント（Document）

インデックス内の1件1件のレコードです。リレーショナルデータベースでいう「行」に相当します。JSON形式で表現されます。

## データソース（Data Source）

インデックスへ投入するデータの取得元を定義するオブジェクトです。Blob Storageや Cosmos DBなどへの接続情報を保持します。

## インデクサー（Indexer）

データソースからデータを取得し、必要に応じてスキルセット（AIエンリッチメント）を適用したうえで、インデックスへ投入する処理を自動化するコンポーネントです。スケジュールを設定して定期実行することもできます。

## スキルセット（Skillset）

インデクサーの処理過程で適用するAIエンリッチメント処理（OCR、翻訳、エンティティ抽出など）を定義したパイプラインです。

## エイリアス（Alias）

インデックス名に対する別名を設定できる機能です。本番運用中にインデックスを無停止で切り替えたい場合などに有用です。

これらの関係性をまとめると、以下のようなデータフローになります。

```
データソース（Blob Storage等）
      ↓
   インデクサー（データ取得・変換をオーケストレーション）
      ↓
   スキルセット（AIエンリッチメント：OCR、翻訳、Embedding生成など）
      ↓
   インデックス（検索可能な形でデータを格納）
      ↓
   検索クエリ（アプリケーションからのリクエスト）
```

# 料金プランについて

Azure AI Searchには複数の価格レベル（Tier）が用意されています。主なものは以下の通りです。

| Tier | 特徴 |
|---|---|
| Free | 無料枠。学習用途向け。インデックス数やストレージに厳しい制限あり |
| Basic | 小規模な本番運用向け |
| Standard (S1/S2/S3) | 中〜大規模向け。パーティションやレプリカを柔軟に設定可能 |
| Storage Optimized (L1/L2) | 大容量データを扱う用途向け |

料金は「レプリカ数 × パーティション数」で構成されるユニット数に応じて課金されます。レプリカを増やすとクエリのスループットと可用性が向上し、パーティションを増やすとストレージ容量とインデックス作成のスループットが向上します。学習目的であればFreeプランで十分に機能を試すことができますが、ベクトル検索やセマンティックランカーなど一部機能は上位プランでのみ利用可能な場合があるため、公式ドキュメントの最新情報を確認することをおすすめします。

# ハンズオン：Azure AI Searchを構築してみる

ここからは実際にAzureポータルとPythonのSDKを使って、Azure AI Searchのリソースを作成し、簡単な検索を試してみます。

## ステップ1: リソースの作成

Azureポータルにログインし、「リソースの作成」から「AI Search」（もしくは「Search Service」）を検索して選択します。

設定項目は主に以下の通りです。

- **サブスクリプション**: 利用するAzureサブスクリプションを選択
- **リソースグループ**: 新規作成、または既存のものを選択
- **サービス名**: グローバルで一意な名前を指定（例: `my-search-demo`）
- **場所（リージョン）**: Japan Eastなど、利用したいリージョンを選択
- **価格レベル**: 学習用であれば`Free`を選択

作成が完了すると、検索サービスの管理画面から「URL（エンドポイント）」と「管理者キー」を確認できます。これらはSDKやREST APIから接続する際に必要になります。

## ステップ2: インデックスの作成（Python SDK）

Pythonの`azure-search-documents`ライブラリを使ってインデックスを作成してみます。まずはライブラリをインストールします。

```bash
pip install azure-search-documents azure-identity
```

次に、シンプルな書籍情報を検索できるインデックスを定義します。

```python
from azure.core.credentials import AzureKeyCredential
from azure.search.documents.indexes import SearchIndexClient
from azure.search.documents.indexes.models import (
    SearchIndex,
    SimpleField,
    SearchableField,
    SearchFieldDataType,
)

endpoint = "https://my-search-demo.search.windows.net"
admin_key = "<your-admin-key>"
index_name = "books-index"

index_client = SearchIndexClient(
    endpoint=endpoint,
    credential=AzureKeyCredential(admin_key),
)

fields = [
    SimpleField(name="id", type=SearchFieldDataType.String, key=True),
    SearchableField(name="title", type=SearchFieldDataType.String, analyzer_name="ja.microsoft"),
    SearchableField(name="author", type=SearchFieldDataType.String),
    SearchableField(name="description", type=SearchFieldDataType.String, analyzer_name="ja.microsoft"),
    SimpleField(name="price", type=SearchFieldDataType.Double, filterable=True, sortable=True),
    SimpleField(name="published_year", type=SearchFieldDataType.Int32, filterable=True, facetable=True),
]

index = SearchIndex(name=index_name, fields=fields)
index_client.create_index(index)

print(f"インデックス '{index_name}' を作成しました。")
```

ポイントは、`analyzer_name="ja.microsoft"`のように日本語向けのアナライザーを指定している点です。Azure AI Searchは日本語の形態素解析に対応したアナライザーを標準で提供しており、これを指定することで日本語の表記ゆれにもある程度対応した検索が可能になります。

## ステップ3: ドキュメントの投入

作成したインデックスに対してデータを投入します。

```python
from azure.search.documents import SearchClient

search_client = SearchClient(
    endpoint=endpoint,
    index_name=index_name,
    credential=AzureKeyCredential(admin_key),
)

documents = [
    {
        "id": "1",
        "title": "実践的なクラウド設計入門",
        "author": "山田太郎",
        "description": "Azureを中心としたクラウドアーキテクチャの設計手法を解説する書籍です。",
        "price": 3200,
        "published_year": 2023,
    },
    {
        "id": "2",
        "title": "検索エンジンの仕組みを学ぶ",
        "author": "鈴木花子",
        "description": "全文検索エンジンの内部構造とインデックス作成のアルゴリズムを丁寧に解説。",
        "price": 2800,
        "published_year": 2021,
    },
    {
        "id": "3",
        "title": "生成AIとRAGシステム構築の実務",
        "author": "佐藤次郎",
        "description": "大規模言語モデルと検索拡張生成（RAG）を組み合わせた実践的な構築事例集。",
        "price": 3600,
        "published_year": 2024,
    },
]

result = search_client.upload_documents(documents=documents)
print("投入結果:", result)
```

## ステップ4: 検索クエリの実行

投入したデータに対して検索を実行してみます。

```python
results = search_client.search(search_text="クラウド 設計")

for result in results:
    print(f"タイトル: {result['title']}")
    print(f"著者: {result['author']}")
    print(f"説明: {result['description']}")
    print("---")
```

フィルタやファセット（絞り込み用の集計）を組み合わせることも可能です。

```python
results = search_client.search(
    search_text="AI",
    filter="published_year ge 2022",
    facets=["published_year"],
    order_by=["price desc"],
)

for result in results:
    print(result["title"], result["price"], result["published_year"])
```

このように、SQLライクなフィルタ構文（OData形式）を使って条件を絞り込んだり、ソート順を指定したりできる点も大きな特徴です。

# ベクトル検索を試してみる

続いて、Azure AI Search最大の注目機能である「ベクトル検索」を試してみます。ベクトル検索を行うには、あらかじめテキストをEmbedding（埋め込みベクトル）に変換しておく必要があります。ここではAzure OpenAI Serviceの`text-embedding-3-large`モデルを利用する例を示します。

## ベクトルフィールドを含むインデックス定義

```python
from azure.search.documents.indexes.models import (
    SearchIndex,
    SimpleField,
    SearchableField,
    SearchFieldDataType,
    SearchField,
    VectorSearch,
    VectorSearchProfile,
    HnswAlgorithmConfiguration,
)

fields = [
    SimpleField(name="id", type=SearchFieldDataType.String, key=True),
    SearchableField(name="content", type=SearchFieldDataType.String, analyzer_name="ja.microsoft"),
    SearchField(
        name="content_vector",
        type=SearchFieldDataType.Collection(SearchFieldDataType.Single),
        searchable=True,
        vector_search_dimensions=3072,  # text-embedding-3-largeの次元数
        vector_search_profile_name="my-vector-profile",
    ),
]

vector_search = VectorSearch(
    algorithms=[HnswAlgorithmConfiguration(name="my-hnsw-config")],
    profiles=[
        VectorSearchProfile(
            name="my-vector-profile",
            algorithm_configuration_name="my-hnsw-config",
        )
    ],
)

index = SearchIndex(name="vector-index", fields=fields, vector_search=vector_search)
index_client.create_index(index)
```

ここで指定している`HnswAlgorithmConfiguration`は、近似最近傍探索（ANN）のアルゴリズムであるHNSW（Hierarchical Navigable Small World）を使うことを意味します。ベクトル同士の類似度を高速に計算するための業界標準的な手法です。

## Embeddingを生成してドキュメントを投入

```python
from openai import AzureOpenAI

client = AzureOpenAI(
    azure_endpoint="https://<your-openai-resource>.openai.azure.com/",
    api_key="<your-openai-key>",
    api_version="2024-02-01",
)

def get_embedding(text: str):
    response = client.embeddings.create(
        input=text,
        model="text-embedding-3-large",
    )
    return response.data[0].embedding

texts = [
    "Azure AI Searchはフルマネージドな検索サービスです。",
    "ベクトル検索により意味的な類似検索が可能になります。",
    "RAGは検索結果を用いて生成AIの回答精度を高める手法です。",
]

documents = []
for i, text in enumerate(texts):
    documents.append({
        "id": str(i),
        "content": text,
        "content_vector": get_embedding(text),
    })

search_client = SearchClient(
    endpoint=endpoint,
    index_name="vector-index",
    credential=AzureKeyCredential(admin_key),
)
search_client.upload_documents(documents=documents)
```

## ベクトル検索の実行

```python
from azure.search.documents.models import VectorizedQuery

query_text = "検索拡張生成について教えて"
query_vector = get_embedding(query_text)

vector_query = VectorizedQuery(
    vector=query_vector,
    k_nearest_neighbors=3,
    fields="content_vector",
)

results = search_client.search(
    search_text=None,
    vector_queries=[vector_query],
)

for result in results:
    print(result["content"])
```

このように、クエリ文字列そのものではなく「クエリのベクトル表現」に基づいて類似度の高いドキュメントを取得できます。これにより「RAG」というキーワードを含まない文章でも、意味的に関連する「検索拡張生成」に関する文書をヒットさせることが可能になります。

## ハイブリッド検索の実装

実務では、キーワード検索とベクトル検索を組み合わせたハイブリッド検索を使うケースが多いです。実装はシンプルで、`search_text`と`vector_queries`を同時に指定するだけです。

```python
results = search_client.search(
    search_text=query_text,
    vector_queries=[vector_query],
    top=5,
)
```

これにより、キーワードの完全一致による精度と、ベクトル検索による意味的な網羅性の両方を活かした検索結果が得られます。多くのベンチマークで、単純なキーワード検索やベクトル検索単体よりもハイブリッド検索の方が検索精度（再現率・適合率）が高くなる傾向が報告されています。

# セマンティックランカーの活用

ハイブリッド検索の結果に対して、さらに意味的な観点から並び替えを行う「セマンティックランカー」を有効にすることもできます。これはインデックス作成時に「セマンティック構成（Semantic Configuration）」を定義し、検索クエリ実行時に`query_type="semantic"`を指定することで利用できます。

```python
results = search_client.search(
    search_text=query_text,
    vector_queries=[vector_query],
    query_type="semantic",
    semantic_configuration_name="my-semantic-config",
    top=5,
)

for result in results:
    print(result["content"], result["@search.reranker_score"])
```

セマンティックランカーを使うと、通常のBM25スコアに加えて`@search.reranker_score`という指標が返されるようになり、これを使ってより「人間が読んで自然な関連度順」に近い並び替えが可能になります。

# RAG（検索拡張生成）との連携

近年、Azure AI Searchが特に注目されている理由の一つが、Azure OpenAI ServiceとあわせたRAG（Retrieval Augmented Generation）システムの構築基盤として利用できる点です。

## RAGとは何か

大規模言語モデル（LLM）は膨大な知識を持っていますが、以下のような弱点があります。

- 学習データの時点までの情報しか知らない（最新情報に弱い）
- 企業内の非公開情報（社内ドキュメントなど）は当然知らない
- 誤った情報をもっともらしく生成してしまう「ハルシネーション」が発生することがある

RAGはこれらの課題に対処するためのアーキテクチャパターンで、ユーザーの質問に対して、まず検索エンジンで関連する文書を取得し、その文書の内容をプロンプトに含めたうえでLLMに回答を生成させる、という流れを取ります。

```
ユーザーの質問
     ↓
Azure AI Searchで関連文書を検索（ベクトル検索 / ハイブリッド検索）
     ↓
検索結果の文書をコンテキストとしてプロンプトに埋め込む
     ↓
Azure OpenAI ServiceのLLM（GPT-4oなど）が回答を生成
     ↓
根拠に基づいた回答をユーザーに返却
```

このアーキテクチャにおいて、Azure AI Searchは「検索」の部分、つまり社内文書やナレッジベースから関連情報を高精度に取得する役割を担います。

## RAGの簡易実装例

以下は、Azure AI Searchで取得した検索結果をコンテキストとして、Azure OpenAIに回答を生成させる簡易的な実装例です。

```python
def rag_answer(question: str):
    # 1. 質問をベクトル化
    question_vector = get_embedding(question)

    # 2. Azure AI Searchでハイブリッド検索を実行
    vector_query = VectorizedQuery(
        vector=question_vector,
        k_nearest_neighbors=3,
        fields="content_vector",
    )
    search_results = search_client.search(
        search_text=question,
        vector_queries=[vector_query],
        query_type="semantic",
        semantic_configuration_name="my-semantic-config",
        top=3,
    )

    context_chunks = [r["content"] for r in search_results]
    context_text = "\n---\n".join(context_chunks)

    # 3. プロンプトを構築してLLMに問い合わせ
    system_prompt = (
        "あなたは社内ドキュメントに基づいて質問に答えるアシスタントです。"
        "以下のコンテキストに含まれる情報のみを根拠として回答してください。"
        "コンテキストに答えがない場合は「わかりません」と答えてください。"
    )

    chat_response = client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": f"コンテキスト:\n{context_text}\n\n質問: {question}"},
        ],
    )

    return chat_response.choices[0].message.content


answer = rag_answer("Azure AI Searchでベクトル検索を有効にする方法は？")
print(answer)
```

このように、Azure AI Searchの検索結果をそのままLLMのプロンプトに組み込むことで、社内独自のデータに基づいた回答生成を実現できます。

## Azure AI Foundryとの統合

Azure OpenAI Service単体だけでなく、Azure AI Foundry（旧Azure AI Studio）上でも、Azure AI Searchを「データソース」として簡単に接続できる機能が提供されています。GUI上でインデックスを指定するだけで、チャット機能に「自社データを使う」というオプションを追加できるため、コードを書かずにRAGのプロトタイプを試すことも可能です。本格的な実装に入る前に、まずはこの機能で挙動を確認してみるのもおすすめです。

# チャンク分割（Chunking）の重要性

RAGを実装するうえで避けて通れないのが「チャンク分割」の設計です。長大な文書をそのままEmbeddingに変換すると、ベクトルが薄まってしまい検索精度が落ちてしまいます。そのため、文書を適切なサイズ（例えば500〜1000トークン程度）に分割してからEmbedding化するのが一般的です。

チャンク分割の方針としては、以下のような選択肢があります。

- **固定長分割**: 一定の文字数・トークン数で機械的に分割する。実装は簡単だが文脈が途中で切れるリスクがある
- **オーバーラップ分割**: チャンクの境界に重複部分を持たせることで、文脈の断絶を緩和する
- **意味的分割**: 見出しや段落など、文書の構造的な区切りに沿って分割する
- **再帰的分割**: 大きな単位（章）→小さな単位（段落・文）と階層的に分割していく

Azure AI Searchでは、インデクサーの「テキスト分割スキル（Split Skill）」を使うことで、こうしたチャンク分割処理をパイプラインに組み込むことも可能です。また、Azure AI Foundryの「データの取り込み（Add your data）」機能を使うと、チャンク分割からEmbedding生成、インデックス作成までをGUI上で自動化できます。

# セキュリティと運用面のポイント

## アクセス制御

Azure AI Searchでは、APIキーによる認証に加えて、Azure Active Directory（Microsoft Entra ID）を使ったロールベースアクセス制御（RBAC）もサポートしています。管理者キー（フルアクセス）とクエリキー（読み取り専用）を使い分けることで、フロントエンドアプリケーションに強い権限を持つキーを埋め込んでしまうリスクを避けられます。

## プライベートエンドポイント

企業のセキュリティ要件によっては、インターネットを経由せずAzure仮想ネットワーク内で閉じた通信を行いたいケースがあります。Azure AI Searchはプライベートエンドポイントに対応しており、VNet内からのみアクセス可能な構成を組むことができます。

## ドキュメントレベルのアクセス制御

マルチテナントのSaaSサービスなどでは、ユーザーごとに閲覧可能なドキュメントを制限したいケースがあります。Azure AI Searchでは、フィルタ機能を使って「特定のユーザーIDやグループIDに紐づくドキュメントのみを返す」といったセキュリティトリミングを実装できます。

## モニタリング

Azure Monitorと連携することで、クエリのレイテンシ、スロットリングの発生状況、インデクサーの実行結果などを監視できます。本番運用では、インデクサーのエラー通知やクエリのパフォーマンス低下をいち早く検知できる体制を整えておくことが重要です。

# 他の検索ソリューションとの比較

Azure AI Searchを検討する際、よく比較対象になるソリューションについても簡単に触れておきます。

## Elasticsearch / OpenSearch

オープンソースの全文検索エンジンであり、非常に高い柔軟性とカスタマイズ性を持ちます。自前でクラスタを構築・運用する必要があり、その分の運用コストがかかりますが、細かいチューニングや独自のプラグイン開発などが可能です。Azure AI Searchは、その運用の手間をマネージドサービスとして肩代わりする代わりに、内部実装の細かい制御はある程度制限される、というトレードオフになります。

## Azure Cosmos DB for MongoDB vCore / PostgreSQL拡張のベクトル検索

近年は、データベース側にもベクトル検索機能が組み込まれるケースが増えています。既存のデータベースにデータがある場合、そのままベクトル検索を追加できるという利点がありますが、AIエンリッチメントパイプラインやセマンティックランカーといった高度な検索機能については、Azure AI Searchの方が専門特化している分、機能が充実しています。

## 選定のポイント

- 既にAzureエコシステムを利用しており、運用負荷を抑えたい → Azure AI Search
- 非常に高いカスタマイズ性や、マルチクラウド・オンプレミスでの運用が必要 → Elasticsearch等の自己運用型
- 既存のデータベースにシンプルなベクトル検索だけ追加したい → データベース組み込みのベクトル検索機能

# よくあるユースケース

最後に、Azure AI Searchが実際にどのような場面で使われているか、代表的なユースケースを紹介します。

## 1. 社内ナレッジベース検索

社内Wiki、規程集、過去の議事録などをインデックス化し、「あの規程どこだっけ？」を即座に検索できるようにする用途です。AIエンリッチメントでPDFのOCRやキーフレーズ抽出を組み合わせることで、非構造化文書もまとめて検索対象にできます。

## 2. ECサイトの商品検索

商品名や説明文だけでなく、画像解析による自動タグ付けを活用し、「赤いスニーカー」といった曖昧な検索にも対応できる商品検索を実現できます。ファセット機能を使えば、カテゴリや価格帯での絞り込みUIも簡単に実装可能です。

## 3. カスタマーサポートのFAQ検索・チャットボット

過去の問い合わせ履歴やFAQをベクトル化しておき、ユーザーの質問に対して意味的に近いFAQを提示する、あるいはRAGを使ってチャットボットが根拠のある回答を生成する、といった用途で活用されています。

## 4. 法務・コンプライアンス文書の検索

契約書や法令文書など、専門用語が多く長大な文書に対して、キーフレーズ抽出やエンティティ認識を組み合わせることで、必要な条項や関連する過去の契約を素早く見つけ出す用途にも利用されています。

## 5. 医療・研究分野での文献検索

論文や臨床データなど専門性の高い文書に対して、ベクトル検索による類似文献の発見や、セマンティックランカーによる関連度の高い論文の絞り込みなどに活用されるケースも増えています。

# まとめ

本記事では、Azure AI Searchについて以下の内容を解説しました。

- Azure AI Searchはフルマネージドな検索サービスであり、インフラ管理の手間を大幅に削減できる
- AIエンリッチメントパイプラインにより、OCRや画像解析、エンティティ抽出などを組み込んだ高度な検索インデックスを構築できる
- ベクトル検索・ハイブリッド検索・セマンティックランカーにより、キーワードの完全一致だけでなく意味的な検索が可能
- Python SDKを使えば、インデックス作成からデータ投入、検索クエリの実行までスムーズに実装できる
- Azure OpenAI ServiceやAzure AI Foundryと組み合わせることで、RAGシステムの検索基盤として活用できる
- セキュリティ面でもRBACやプライベートエンドポイント、ドキュメントレベルのアクセス制御など、エンタープライズ用途に耐えうる機能が揃っている

検索機能というと地味な印象を持たれがちですが、生成AIブームの中でRAGの重要性が高まったことで、Azure AI Searchはあらためて大きな注目を集めているサービスです。まずはFreeプランで小さなインデックスを作り、ベクトル検索やRAGの挙動を実際に触って体感してみることをおすすめします。

最後まで読んでいただきありがとうございました。この記事が、Azure AI Searchを使い始めるきっかけになれば幸いです。

もし本記事が役に立った、参考になったという方がいらっしゃいましたら、Zennのサポート機能で応援していただけると励みになります。今後も実際に手を動かして得た知見をわかりやすい形でアウトプットしていきますので、よろしければお願いします🙏

# 参考リンク

- [Azure AI Search 公式ドキュメント](https://learn.microsoft.com/azure/search/)
- [Azure AI Search とは](https://learn.microsoft.com/azure/search/search-what-is-azure-search)
- [ベクトル検索の概要](https://learn.microsoft.com/azure/search/vector-search-overview)
- [Azure OpenAI Service ドキュメント](https://learn.microsoft.com/azure/ai-services/openai/)