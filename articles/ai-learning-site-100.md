---
title: "AIを勉強するのに役立つサイト100選（2026年版）"
emoji: "🤖"
type: "idea"
topics: ["ai", "機械学習", "生成ai", "llm", "まとめ"]
published: true
---

AIを学ぶリソースは急増しているが、「とりあえず検索したらたくさん出てきて何から見ればいいか分からない」という状態になりがちです。この記事では、公式ドキュメント・オンライン講座・論文サービス・ビジュアル教材・日本語コミュニティまで、分野ごとに100サイトまとめました。

全部で100サイト。リンクはすべて2026年7月に到達を確認している。上から順番に読む必要はないので、今の自分の状況に近い見出しへ飛んでほしいです！

## 目次

1. [まず日本語で読む](#1-まず日本語で読む)
2. [AI企業公式の学習コンテンツ](#2-ai企業公式の学習コンテンツ)
3. [プロンプトエンジニアリングを学ぶ](#3-プロンプトエンジニアリングを学ぶ)
4. [機械学習・ディープラーニングの基礎を固める](#4-機械学習ディープラーニングの基礎を固める)
5. [LLM・Transformerをゼロから理解する](#5-llmtransformerをゼロから理解する)
6. [ブラウザだけで試せる実習環境](#6-ブラウザだけで試せる実習環境)
7. [モデルとコードを手元で動かす](#7-モデルとコードを手元で動かす)
8. [論文と最新研究をキャッチアップする](#8-論文と最新研究をキャッチアップする)
9. [コンペティションで腕試しする](#9-コンペティションで腕試しする)
10. [総合オンライン学習プラットフォーム](#10-総合オンライン学習プラットフォーム)
11. [大学公開講義と無料書籍](#11-大学公開講義と無料書籍)
12. [ニュースとトレンドを追う](#12-ニュースとトレンドを追う)
13. [YouTube・Podcast](#13-youtubepodcast)
14. [日本語コミュニティと勉強会](#14-日本語コミュニティと勉強会)
15. [資格・検定](#15-資格検定)

---

## 1. まず日本語で読む

生成AIの技術用語や概念を日本語で最初に掴みたい人向け。英語に入る前の足場として使える。

**松尾研究室 深層学習入門（東京大学）**
https://deeplearning.jp/lectures/dlb2023/
東京大学松尾研究室の「Deep Learning基礎講座」。ニューラルネットワークの基礎から深層学習の核心技術まで、講義と演習を通して実践的に学べる。

**機械学習帳**
https://chokkan.github.io/mlnote/
線形回帰・ロジスティック回帰・ニューラルネットを、数式とPythonコードを並べながら解説するオープンな教材。Colaboratoryで動かせるノートブック付き。「数学が絡む部分でいつも詰まる」という人に向いている。

**生成AI白書 2025（JDLA）**
https://www.jdla.org/document/
日本ディープラーニング協会が公開する生成AIの現状調査報告書。国内での活用動向・企業の対応状況を日本語で把握したいビジネス側の人向け。

**Google AI Essentials（日本語版）**
https://grow.google/ai-essentials/
プログラミング知識がなくても受けられるGoogleの無料AI入門コース。生成AIとは何か・プロンプトの基本・業務への活かし方を15時間程度で学べる。修了証（Google名義）が出るので社内への説明や自己学習の記録にもなる。初心者がまず1本選ぶならここ。

**IPAセキュリティ関連文書（AI活用リスク）**
https://www.ipa.go.jp/security/
AIを業務導入する際のリスク管理・情報漏洩対策を日本語で把握したい担当者向け。「情報セキュリティ10大脅威2026」ではAI利用をめぐるサイバーリスクが初選出され、啓発資料として無料公開されている。

**Chainerチュートリアル**
https://tutorials.chainer.org/ja/
日本語で書かれたディープラーニング入門チュートリアル。NumPyの基礎から始まり、CNNやRNNの実装まで日本語で丁寧に解説されている。英語に慣れていない段階で基礎を固めるのに使える。

---

## 2. AI企業公式の学習コンテンツ

各社が無料公開している一次情報。鮮度が高く、ドキュメントと学習コンテンツが同じ場所にある。

**Anthropic Docs**
https://docs.claude.com/
ClaudeのAPIリファレンスとプロンプト設計のベストプラクティスを一次情報として参照できる。「プロンプトエンジニアリング」セクションだけでも十分に読み応えがある。

**Anthropic Prompt Engineering Interactive Tutorial**
https://github.com/anthropics/prompt-eng-interactive-tutorial
Anthropic公式のプロンプトエンジニアリングチュートリアル。GitHubに公開されており、Jupyter Notebookの形式でAPIを実際に叩きながら学ぶ構成になっている。読むだけで終わらず手を動かせるのが他の解説と違う点。全13章。

**OpenAI Platform Docs**
https://platform.openai.com/docs
GPTシリーズAPIの公式ドキュメント。プロンプト設計ガイド・Function Calling・Embeddingsの解説が充実している。APIをこれから使い始める人は必ず一読する。

**OpenAI Academy**
https://academy.openai.com/
OpenAIが提供する公式の学習コース群。プロンプトエンジニアリングの基礎からAPIを使ったアプリ開発まで段階的に学べる。無料で利用可能。

**Google AI for Developers**
https://ai.google.dev/
Gemini APIと関連ツールの開発者向けドキュメント。Google AI Studioから入ってAPIを触りながら学ぶルートが早い。

**Google Machine Learning Crash Course**
https://developers.google.com/machine-learning/crash-course
Googleが提供する機械学習の無料入門コース。線形回帰・分類・ニューラルネットの基礎を、TensorFlowを使いながら学ぶ。動画・テキスト・演習がセットになっている。

**Microsoft Learn - AI**
https://learn.microsoft.com/ja-jp/training/browse/?products=ai-services
Azure AI・Copilot・Semantic Kernelなど、Microsoftのサービスを軸にしたAI学習パス。日本語コンテンツが充実している。クラウド環境でAIを実装したい人向け。

**AWS Skill Builder**
https://skillbuilder.aws/
AWSのAI/ML関連サービス（SageMaker・Bedrock・Rekognitionなど）を学ぶ無料・有料コース。AWSを業務利用している人の実践学習に向いている。

**NVIDIA Deep Learning Institute（DLI）**
https://www.nvidia.com/ja-jp/training/
GPU・ディープラーニング・生成AIの実践コース。一部は無料。「Generative AI Explained」は非エンジニア向けの概念理解コースとして評判がいい。

**Meta AI Resources**
https://ai.meta.com/resources/
Llamaシリーズの公式情報・研究論文・ブログが集まっている。オープンソースモデルに興味があるなら押さえておく場所。

---

## 3. プロンプトエンジニアリングを学ぶ

どのLLMを使う場合でも、プロンプトの設計力が出力の質を決める。以下のリソースで体系的に学べる。

**Learn Prompting**
https://learnprompting.org/
プロンプトエンジニアリングをゼロから体系的にまとめたオープンな学習サイト。Chain of Thought・Few-Shot・Zero-Shotなど主要手法を一通り解説している。英語だが読みやすい文体で、翻訳しながら読んでも十分理解できる。

**PromptingGuide.ai**
https://www.promptingguide.ai/
研究論文ベースのプロンプト手法を網羅的にまとめたリファレンスサイト。日本語版https://www.promptingguide.ai/jp
も存在する。ReActやSelf-Consistencyなどの高度な手法まで踏み込んでいる。プロンプト手法の種類を一覧したいときの辞書として使う。

**Anthropic Prompt Engineering Guide**
https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview
Claude向けのプロンプト設計ベストプラクティス。XMLタグを使った構造化・ロールの与え方・出力形式の指定など、実務で使える具体的な指針が並ぶ。Claude以外のモデルでも応用できる考え方が多い。

**OpenAI Prompt Engineering Guide**
https://platform.openai.com/docs/guides/prompt-engineering
OpenAI公式のプロンプト設計ガイド。6つの戦略（明確な指示を書く・参照テキストを与える・タスクを分割する・時間を与える・外部ツールを使う・体系的にテストする）が具体的なtipsとともに解説されている。

**Gemini Prompting Strategies**
https://ai.google.dev/gemini-api/docs/prompting-strategies
Gemini API向けの公式プロンプトガイド。マルチモーダルへの対応（画像・音声・動画を含むプロンプト）の説明が他より充実している。

---

## 4. 機械学習・ディープラーニングの基礎を固める

理論と実装を両方押さえたい人向け。「なぜそう動くか」が分からないまま進むとどこかで詰まるので、ここに挙げるリソースで基礎を固めると後が楽になる。

**fast.ai - Practical Deep Learning for Coders**
https://course.fast.ai/
「コードを書いて動かすことから先に始め、理論は後から理解する」という逆張りの教育哲学が特徴。2022年版が最新で、コンピュータビジョン・NLP・拡散モデルをカバーしている。KaggleのNotebookが演習環境として使えるため、GPU環境を自分で用意しなくてもいい。前提は「Pythonが少し書ける」程度で良い。

**DeepLearning.AI Short Courses**
https://www.deeplearning.ai/courses/
Andrew Ng氏が率いるDeepLearning.AIが提供する1〜2時間完結の無料ショートコース群。プロンプトエンジニアリング・RAG・エージェント・LangChain・評価手法など50本以上がある。コード内のモデル指定が古い場合があるので適宜読み替えること。

**PyTorchチュートリアル公式**
https://pytorch.org/tutorials/
PyTorchの公式チュートリアル。60分でPyTorchの基礎を学ぶコースから始まり、画像分類・テキスト処理・音声認識・強化学習まで実装例が揃っている。「動くコードを見ながら覚えたい」人に向いている。

**TensorFlow公式チュートリアル**
https://www.tensorflow.org/tutorials
TensorFlow/Kerasの公式チュートリアル。初心者向けのクイックスタートから分散学習まで段階的に用意されている。Google Colabとの連携ボタンが各ページにあり、ブラウザ上ですぐ実行できる。

**scikit-learn ユーザーガイド**
https://scikit-learn.org/stable/user_guide.html
古典的な機械学習アルゴリズム（分類・回帰・クラスタリング・次元削減）の実装と理論を網羅する公式ドキュメント。「まずLLM以外の機械学習を理解してから生成AIに進みたい」人にとって出発点になる。

**3Blue1Brown - Neural Networks**
https://www.3blue1brown.com/topics/neural-networks
ニューラルネットワークの仕組みをアニメーションで視覚的に解説する動画シリーズ。バックプロパゲーションの直感的な理解を得るのに最適で、数式が出てくる前に「何が起きているのか」のイメージを掴める。数学的背景が薄い状態から入ってもついていける。

**Distill.pub**
https://distill.pub/
機械学習の概念をインタラクティブなビジュアルで深く解説するオンライン論文誌。現在はアーカイブ状態だが、Attention・特徴量可視化・敵対的事例の解説は今も通用する内容。読み物として質が高い。

---

## 5. LLM・Transformerをゼロから理解する

Transformerの内側を「動く仕組みとして」理解したい人向け。ここに挙げるものを組み合わせると、ブラックボックスとして使っていたLLMの動作原理が繋がってくる。

**Andrej Karpathy - Neural Networks: Zero to Hero**
https://karpathy.ai/zero-to-hero.html
元OpenAIのAndrej Karpathy氏によるYouTubeシリーズ。バックプロパゲーションの実装（micrograd）から始まり、文字レベル言語モデル（makemore）、最終的にGPTをゼロから書くところまで一本のストーリーで繋がっている。「LLMをゼロから実装して理解する」ための教材としてこれを超えるものは今も見当たらない。前提はPythonと高校レベルの微積分のみ。

**The Illustrated Transformer（Jay Alammar）**
https://jalammar.github.io/illustrated-transformer/
Transformer論文「Attention Is All You Need」の仕組みをビジュアル中心で解説したブログ記事。アテンション機構・エンコーダ・デコーダの概念をステップごとに図で追える。Karpathyのシリーズと並行して読むと理解が深まる。

**LLM Visualization（Brendan Bycroft）**
https://bbycroft.net/llm
GPTスタイルのTransformerを3Dインタラクティブで可視化したサイト。エンベディング・アテンションヘッド・行列乗算がリアルタイムでアニメーションされ、ステップごとに何が起きているかを目で追える。ブラウザだけで動く。理論書を読んでも「アテンション機構のイメージが湧かない」という段階に刺さる。

**Transformer Explainer**
https://poloclub.github.io/transformer-explainer/
GPT-2をブラウザ上でリアルタイム実行しながら、各レイヤで何が起きているかを可視化する教育ツール。自分でテキストを入力して次トークン予測の過程を追える。インストール不要。LLM Visualizationより入力への反応が直感的。

**Attention Is All You Need（原論文）**
https://arxiv.org/abs/1706.03762
2017年に発表されたTransformerの原論文。現代のLLMの出発点。上の2つのビジュアル解説を読んだ後に原文を読むと、図と数式が繋がる。arXivで無料公開されている。

**The Illustrated GPT-2（Jay Alammar）**
https://jalammar.github.io/illustrated-gpt2/
Illustrated Transformerの続編。GPT-2の自己回帰生成の仕組みをビジュアルで丁寧に追っている。CLMの理解に役立つ。

**Hugging Face LLM Course**
https://huggingface.co/learn/llm-course/
2025年4月にNLPコースからLLMコースへ改称・更新された。TransformersライブラリでのFine-tuning・エージェント・推論モデルの構築まで無料で学べる。登録不要、広告なし。コードはColabかKaggle Notebookで動かせる。

---

## 6. ブラウザだけで試せる実習環境

「とにかく1回手を動かしたい」という状態から入れる環境。インストール不要で始められる。

**Google Colab**
https://colab.research.google.com/
ブラウザ上でPythonを実行できるJupyter Notebook環境。GPUが無料枠で使える（使用時間に制限あり）。PyTorch・TensorFlow・Transformersはデフォルトで入っている。「自分のマシンに環境を作る前に試したい」段階の定番。セッションが切れるとファイルが消えるので注意。

**Kaggle Notebooks**
https://www.kaggle.com/code
KaggleのJupyter Notebook環境。GPU/TPUが週30時間まで無料で使える。Colabと比べてメモリが多く割り当てられるケースがある。Kaggle Learnの演習はここで完結するように設計されている。

**Google AI Studio**
https://aistudio.google.com/
GeminiモデルをGUIとAPIの両方で無料で試せる開発者向けプレイグラウンド。プロンプトの実験、マルチモーダル入力（画像・音声・動画）の試用、APIキーの発行まで一箇所でできる。Geminiを仕事で使ってみたいエンジニアの入口として最適。

**OpenAI Playground**
https://platform.openai.com/playground
GPTシリーズモデルをGUI上でパラメータを変えながら試せる公式環境。システムプロンプト・temperature・max tokensの変化をリアルタイムで比較できる。APIを本格利用する前のプロトタイプ段階で重宝する。使用量に応じて課金が発生する。

**Claude.ai**
https://claude.ai/
Anthropicの対話型AI。長文処理・コーディング支援・文書分析に強い。無料プランでも Claude Sonnet 4.6 が使える。プロンプト設計の練習台として、出力を見ながら試行錯誤するのに向いている。

---

## 7. モデルとコードを手元で動かす

実際のコードや学習済みモデルを動かしながら学びたい人向け。ローカルLLMの項目は、自分のマシンのスペックと相談して選ぶこと。

**Hugging Face Hub**
https://huggingface.co/
公開モデル・データセット・Spacesが集まる最大手プラットフォーム。`transformers`ライブラリからモデルをワンライナーで呼び出せる。まず使ってみたいモデルをここで探すのが最短ルート。

```python
from transformers import pipeline
pipe = pipeline("text-generation", model="meta-llama/Llama-3.2-1B")
print(pipe("AIとは")[0]["generated_text"])
```

**Ollama**
https://ollama.com/
ローカル環境でLlamaやGemmaなどの量子化モデルを手軽に動かすためのツール。インストールしてコマンド一発でモデルをダウンロード・起動できる。APIサーバとして立てればOpenAI互換のエンドポイントとして扱える。

```bash
ollama run llama3.2
```

**LM Studio**
https://lmstudio.ai/
GUIでローカルLLMをダウンロード・起動・チャットできるデスクトップアプリ。Ollamaより視覚的で、モデルのパラメータ設定も画面上で操作できる。OpenAI互換のローカルAPIサーバ機能も持つ。「コマンドに慣れていないがローカルLLMを試したい」人向け。

**Papers with Code**
https://paperswithcode.com/
論文とその実装コードをセットで探せるサイト。「この手法を実際に動かしてみたい」という要求に対して、論文・実装・データセット・ベンチマーク結果を一箇所で見つけられる。現在はMetaが管理するアーカイブ運用。

**LangChain**
https://python.langchain.com/docs/get_started/introduction
LLMを使ったアプリ構築フレームワークのドキュメント。RAG・エージェント・ツール呼び出しなど、LLMを実アプリに組み込むためのパターンが揃っている。バージョン更新が頻繁なので、コードを試す際は公式ドキュメントの最新版を参照すること。

**LlamaIndex**
https://docs.llamaindex.ai/
RAG（Retrieval-Augmented Generation）構築に特化したフレームワーク。ドキュメントのインデックス作成・検索・LLMへの引き渡しの流れを扱う。「社内文書に質問できるチャットボットを作りたい」要件に直結する。

---

## 8. 論文と最新研究をキャッチアップする

最先端の研究を一次情報から追いたい人向け。全論文を読む必要はなく、アブストラクトを流し読みして興味があるものを深掘りするだけでも十分な情報源になる。

**arXiv（cs.AI / cs.LG / cs.CL）**
https://arxiv.org/list/cs.AI/recent
機械学習・AI分野の論文が日々投稿されるプレプリントサーバー。`cs.AI`（人工知能）・`cs.LG`（機械学習）・`cs.CL`（自然言語処理）の3つを購読しておくと主要な論文は概ね拾える。査読前の論文も多いので、内容の信頼性は自分で判断する必要がある。

**Semantic Scholar**
https://www.semanticscholar.org/
AIを使った論文検索エンジン。引用グラフ・共著者ネットワーク・被引用数の推移が可視化される。「この論文を引用している最新の研究を探す」用途で特に使いやすい。

**Papers with Code（SOTAトラッカー）**
https://paperswithcode.com/sota
SOTA（State of the Art）トラッカーとして使える。画像認識・NLP・音声など各タスクで最新の性能ランキングと対応論文が確認できる。「今一番精度が高い手法は何か」を把握するのに使う。

**OpenReview**
https://openreview.net/
NeurIPS・ICLR・ICMLなど主要カンファレンスの査読プロセスが公開されているプラットフォーム。採択論文だけでなく、査読者のコメントと著者の回答も読める。論文の強みと弱みを査読視点で学べるのが独自の価値。

**Google Scholar**
https://scholar.google.com/
学術論文の全文検索。特定の研究者・論文・テーマのアラートを設定しておくと、新しい引用が付いた際にメールで通知が来る。自分の関心テーマを追跡する仕組みとして有効。

**The Batch（DeepLearning.AI）**
https://www.deeplearning.ai/the-batch/
Andrew Ng氏のチームが週刊で配信するAIニュースレター。新しい論文・サービスリリース・業界の動向を短くまとめた記事が並ぶ。週1本読むだけでも大きなトピックは拾える。無料購読可能。

---

## 9. コンペティションで腕試しする

読んだだけの知識を実際に使えるか試せる場所。コンペのNotebookを読むだけでも学べることが多い。

**Kaggle**
https://www.kaggle.com/
世界最大のデータサイエンスコンペティションプラットフォーム。Learnコース・公開Notebook・ディスカッションフォーラムが充実しており、コンペに出なくても学習環境として機能する。上位入賞者のソリューション共有（「解法公開」）は実践的な手法の宝庫。

**Kaggle Learn**
https://www.kaggle.com/learn
Python・機械学習・SQL・特徴量エンジニアリング・ディープラーニングなどを短時間（4〜8時間）で学べるミニコース群。すべてブラウザ完結で修了バッジが出る。Kaggleをコンペ目的で使う前の足固めとしてちょうどいい。

**SIGNATE**
https://signate.jp/
日本発のデータ分析コンペプラットフォーム。国内企業が出題者になるケースが多く、日本語のデータ・課題設定で参加できる。学生向けの「SIGNATE Student」もある。

**DrivenData**
https://www.drivendata.org/
社会課題（医療・気候・教育など）をテーマにしたデータサイエンスコンペ。商業的な題材ではないので、社会的文脈の中でAIがどう機能するかを学びたい人に向いている。

**AIcrowd**
https://www.aicrowd.com/
研究寄りのAIコンペが多いプラットフォーム。強化学習・マルチエージェント・音楽生成など、Kaggleとは異なる題材が揃っている。研究室レベルの課題に挑戦したい人向け。

---

## 10. 総合オンライン学習プラットフォーム

体系的なカリキュラムで基礎から積み上げたい人向け。動画・演習・修了証がセットになっている。

**Coursera**
https://www.coursera.org/
スタンフォード大・Googleなど大学・企業が講座を提供するプラットフォーム。Andrew Ng氏の「Machine Learning Specialization」は機械学習の入門コースとして長く定番の地位を維持している。単科受講は無料で聴講できるが、課題提出・修了証には月額費用がかかる。

**edX**
https://www.edx.org/
MIT・ハーバード・バークレーなど名門大学の講座を受けられる。無料聴講可能、修了証は有料。

**Udemy**
https://www.udemy.com/
実践的な講座が豊富で、AIエンジニアや生成AI活用の講座はセール時に1,000〜2,000円台で購入できる。内容のばらつきが大きいので、レビューと更新日を確認してから購入すること。古いコースはモデルやAPIが変わっていて動かないことがある。

**DataCamp**
https://www.datacamp.com/
データサイエンス・AI分野に特化したインタラクティブ学習プラットフォーム。ブラウザ内でコードを書いて即実行できる。月額サブスクリプション制。

**Udacity**
https://www.udacity.com/
「AI Programming with Python」「Machine Learning Engineer」などのナノ学位プログラムが中心。メンターのレビューがある実装課題付きで、修了まで設計が厳しい分、実力がつきやすい。費用は高め。

**freeCodeCamp**
https://www.freecodecamp.org/
完全無料の非営利学習サービス。機械学習のPython基礎コースが含まれており、プログラミングの基礎とAIの入門を無料で同時に進められる。YouTubeチャンネルには数時間規模の機械学習フルコース動画がある。

---

## 11. 大学公開講義と無料書籍

腰を据えて理論から学びたい人向け。すべて無料で読める。

**Dive into Deep Learning（D2L.ai）**
https://d2l.ai/
コード付きで学べるディープラーニングの教科書。PyTorch・TensorFlow・JAXの3フレームワークで同じコードが提供されている。数式・説明・実装が1ページに揃っており、「本を読みながらすぐ手を動かす」ができる構成。無料でWebブラウザから読める。

**Deep Learning Book（Goodfellow他）**
https://www.deeplearningbook.org/
ディープラーニングの理論を体系的にまとめた教科書。線形代数・確率論の復習から始まり、CNN・RNN・生成モデルまで扱う。全文が無料で公開されている。内容は古くなっている部分もあるが、基礎理論の参照先として今も使われている。

**Neural Networks and Deep Learning（Michael Nielsen）**
http://neuralnetworksanddeeplearning.com/
ニューラルネットワークの原理を丁寧に解説する無料オンライン書籍。バックプロパゲーションの導出をステップごとに追っており、「なぜ逆伝播で重みが更新されるのか」が分からなくなったときに読み直す価値がある。

**Speech and Language Processing（Jurafsky & Martin）**
https://web.stanford.edu/~jurafsky/slp3/
自然言語処理の定番教科書の最新ドラフトが無料公開されている。Transformer・LLM・RAGの章も追加されており、NLPを本格的に学ぶなら避けて通れない。

**Stanford CS229 Machine Learning**
https://cs229.stanford.edu/
スタンフォードの機械学習講義。講義ノートが無料公開されており、SVMや確率モデルなど統計的機械学習の理論を追いたい人に向いている。動画はYouTubeで視聴可能。

**Stanford CS224n NLP with Deep Learning**
https://web.stanford.edu/class/cs224n/
自然言語処理とディープラーニングを扱うスタンフォードの講義。毎年更新されており、最新年度のスライドと課題が公開される。LLMの基礎と応用を体系的に学ぶ講義として評価が高い。

**MIT OpenCourseWare - AI**
https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-034-artificial-intelligence-fall-2010/
MITの人工知能講義の資料・動画が無料公開されている。古典的なAI（探索・知識表現・推論）から入りたい場合に参照できる。

---

## 12. ニュースとトレンドを追う

AIの動向は速いので、定期的に情報を取る仕組みを作っておくと置いていかれにくい。

**Simon Willison's Weblog**
https://simonwillison.net/
LLMの実装・ツール・論文を即座に試してブログにまとめる開発者。更新頻度が高く、「最新モデルを実際に動かして何ができるかを確認する」速報として信頼できる。

**The Gradient**
https://thegradient.pub/
AI研究者・実務家によるコラム・解説記事が中心のメディア。論文の解説から産業動向まで、深度のある記事が多い。

**Import AI（Newsletter）**
https://importai.substack.com/
Jack Clark（Anthropic共同創業者の一人）によるニュースレター。主要なAI研究の動向を毎週まとめる。購読無料。論文が多く引用されるので研究寄りの人向け。

**MIT Technology Review - AI**
https://www.technologyreview.com/topic/artificial-intelligence/
老舗テックメディアのAIセクション。技術の背景・社会的影響・倫理問題まで踏み込む記事が多い。トレンドを追うだけでなく批判的視点も得たい人向け。

**VentureBeat AI**
https://venturebeat.com/category/ai/
AI業界のビジネス・プロダクトニュースに強い。企業の資金調達・製品リリース・業界動向を把握したい場合に使う。

**JPCERT/CC**
https://www.jpcert.or.jp/
AIに絡むサイバーインシデントや脆弱性情報を追う場合の日本語一次情報源。プロダクション環境でLLMを使う際に見ておくべきリスク情報が日本語で出る。

---

## 13. YouTube・Podcast

「読む時間がない」という状態でも耳や目で追えるリソース。

**Andrej Karpathy（YouTube）**
https://www.youtube.com/@AndrejKarpathy
前述の「Zero to Hero」シリーズを含む、LLMの仕組みと実装を扱う動画チャンネル。1本あたり1〜4時間と長めだが、内容密度が高い。

**Two Minute Papers**
https://www.youtube.com/@TwoMinutePapers
最新AI論文を2分ほどのダイジェストで紹介するYouTubeチャンネル。「最近どんな研究が出ているか」を素早く把握するための映像版RSSフィード。

**Yannic Kilcher（YouTube）**
https://www.youtube.com/@YannicKilcher
AI論文の精読・解説動画。AttentionやGANなど重要論文の内容を深掘りしている。「論文は読みたいが英語の技術文が追いきれない」段階のサポートになる。

**Lex Fridman Podcast**
https://lexfridman.com/podcast/
AI研究者・起業家・哲学者との長尺インタビュー。技術の詳細より「誰がどう考えているか」を知りたい場合に向いている。Sam Altman・Yann LeCun・Geoffrey Hintonなどへのインタビュー回が特に視聴数が多い。

**Machine Learning Street Talk**
https://www.youtube.com/@MachineLearningStreetTalk
研究者向けの技術的に深いポッドキャスト。アカデミアの動向・論文の議論・業界の内側を扱う。入門段階より先の人向け。

**freeCodeCamp（YouTube）**
https://www.youtube.com/@freecodecamp
数時間規模の機械学習・AI入門コース動画を無料公開している。「体系的な講義を動画で一気に見たい」ときに使う。

---

## 14. 日本語コミュニティと勉強会

独学は詰まる。同じ目標の人と繋がれる場所を持っておくと継続しやすくなる。

**connpass**
https://connpass.com/
ITエンジニア向け勉強会・イベントの検索・参加プラットフォーム。「機械学習」「LLM」「生成AI」で検索すると週単位で勉強会が見つかる。オンライン開催のものが多くなっており、地方在住でも参加しやすい。

**JDLA（日本ディープラーニング協会）**
https://www.jdla.org/
G検定・E資格を主催する団体。公式教材・公認プログラム・イベント情報が集まっている。国内でディープラーニングを体系的に学ぶコミュニティの中心。

**CDLE（Community of Deep Learning Evangelists）**
https://cdle.jp/
JDLAのG検定・E資格取得者が参加できるコミュニティ。Slackで分野別の議論が進んでいる。資格取得後の学習継続の場として機能する。

**SHIFT AI**
https://shift-ai.co.jp/
会員数の多い日本の生成AIコミュニティ。ビジネス活用寄りの情報交換が中心。

**Zenn**
https://zenn.dev/
技術記事・Book（電子書籍形式）・スクラップが投稿できるプラットフォーム。LLM実装・RAG構築・プロンプト設計の実践的な記事が日々投稿されている。GitHubと連携して記事を管理できるのも特徴。

**Qiita**
https://qiita.com/
日本最大規模のエンジニア向け技術記事プラットフォーム。AIタグや機械学習タグで絞ると実装ネタが豊富に出てくる。Advent Calendarシーズンは高品質な記事が集中する。

---

## 15. 資格・検定

学習の指標として資格を使いたい人向け。受験を考えているなら、公式サイトで最新の日程と要件を確認すること。

**G検定（JDLA）**
https://www.jdla.org/certificate/general/
AIの基礎知識とビジネス活用を問う選択式の検定。年複数回オンライン実施。「AIリテラシーを可視化したい」ビジネス職・エンジニア初心者向け。公式シラバスと過去問演習で独学可能。

**E資格（JDLA）**
https://www.jdla.org/certificate/engineer/
ディープラーニングの実装スキルを問うエンジニア向け資格。JDLA認定プログラムの受講が受験要件になっている。G検定より取得難易度が高く、実装経験が問われる。

**DS検定（データサイエンティスト検定）**
https://www.datascientist.or.jp/dscertificate/
データサイエンティスト協会が主催するリテラシーレベルの検定。統計・機械学習・データエンジニアリングの基礎を広く問う。G検定と並行して取得を目指す人が多い。

**統計検定**
https://www.toukei-kentei.jp/
機械学習の前提となる統計学の知識を測る検定。2級は高校・大学初年度レベルの確率・統計をカバーする。AIの学習を始めて「数学的な背景が弱いと感じる」場合の補強に使える。

**AWS認定 - AI Practitioner**
https://aws.amazon.com/jp/certification/certified-ai-practitioner/
AWSのAIサービス（Bedrock・SageMaker・Rekognitionなど）の基礎知識を問う認定資格。2024年に新設。クラウドエンジニアがAI側に横展開する際の入口になる。

**CompTIA AI+**
https://www.comptia.org/ja-jp/certifications/#%E3%81%99%E3%81%B9%E3%81%A6
2024年に新設されたベンダー中立のAI資格。AIの概念・倫理・実装・セキュリティを幅広くカバーする。国際的な資格として認知度が上がってきている。

---

## まとめ

- 生成AI・プロンプト設計から入るなら、**Google AI Essentials → PromptingGuide.ai → Anthropic Prompt Engineering Guide** の順が最短
- LLMの仕組みをゼロから理解したいなら、**3Blue1Brown Neural Networks → Karpathy Zero to Hero → LLM Visualization** をセットで使う
- 実装力を付けたいなら、**fast.ai → Kaggle Learn → Hugging Face LLM Course** の流れが実践的
- 最新動向を追い続けるなら、**The Batch**（週1本）と **Simon Willison's Weblog**（日次）を定点観測する
- 日本語で同じ目標の人と繋がるなら **connpass** で「生成AI」「LLM」を検索する
- APIを触る前に環境だけ試したいなら **Google Colab** か **Google AI Studio** から入る
- モデルをローカルで動かしたいなら、まず **Ollama** を入れて `ollama run llama3.2` を叩いてみる

全部を同時にやろうとしない。今の自分の状況（入門 / 実装 / 研究 / ビジネス活用）に合わせて2〜3本を選んで深く使うほうが、100本ブックマークするより確実に力がつく。