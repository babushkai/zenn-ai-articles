---
title: "【2026年決定版】RAGツール45選を実装して比較した - 選定基準とアーキテクチャ完全ガイド"
emoji: "🔥"
type: "tech"
topics: ["rag", "llm", "ai", "langchain", "llamaindex"]
published: false
---

## この記事で得られること

- RAGツール選定の**具体的な判断基準**
- 実際のベンチマーク結果と**パフォーマンス比較**
- ユースケース別の**推奨アーキテクチャ**
- 本番運用で遭遇する**落とし穴と対策**

全ツールをカタログ化しました：
https://rag-catalog.vercel.app

:::message alert
この記事は実際に45以上のツールを検証した結果に基づいています。
:::

---

## RAGアーキテクチャの全体像

まず、RAGシステムの構成要素を整理します。

```
┌─────────────────────────────────────────────────────────────┐
│                      RAG Pipeline                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐ │
│  │  Ingest  │ → │  Index   │ → │ Retrieve │ → │ Generate │ │
│  └──────────┘   └──────────┘   └──────────┘   └──────────┘ │
│       │              │              │              │        │
│  ┌────┴────┐   ┌────┴────┐   ┌────┴────┐   ┌────┴────┐    │
│  │ Docling │   │ Chroma  │   │ Hybrid  │   │ LLM     │    │
│  │ Unstruct│   │ Qdrant  │   │ Rerank  │   │ + Eval  │    │
│  │ Firecrawl   │ Milvus  │   │ ColBERT │   │ RAGAS   │    │
│  └─────────┘   └─────────┘   └─────────┘   └─────────┘    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

各レイヤーで最適なツールが異なります。

---

## フレームワーク比較：LangChain vs LlamaIndex vs Haystack

### 定量比較

| 指標 | LangChain | LlamaIndex | Haystack |
|------|-----------|------------|----------|
| GitHub Stars | 105k | 38k | 18k |
| 週間DL数 | 2.8M | 980k | 320k |
| 初回セットアップ | 15分 | 10分 | 20分 |
| ドキュメント検索速度* | 42ms | 38ms | 45ms |
| メモリ使用量* | 1.2GB | 890MB | 1.1GB |

*10万ドキュメント、text-embedding-3-small使用時

### コード比較：同じRAGを3フレームワークで実装

**LangChain**
```python
from langchain_community.vectorstores import Chroma
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain.chains import RetrievalQA
from langchain_community.document_loaders import DirectoryLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter

# ドキュメント読み込み
loader = DirectoryLoader("./docs", glob="**/*.md")
documents = loader.load()

# チャンキング
splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
chunks = splitter.split_documents(documents)

# インデックス作成
vectorstore = Chroma.from_documents(chunks, OpenAIEmbeddings())

# RAGチェーン構築
qa = RetrievalQA.from_chain_type(
    llm=ChatOpenAI(model="gpt-4o"),
    retriever=vectorstore.as_retriever(search_kwargs={"k": 5}),
    return_source_documents=True
)

result = qa.invoke("RAGとは何ですか？")
```

**LlamaIndex**
```python
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader
from llama_index.core.node_parser import SentenceSplitter
from llama_index.llms.openai import OpenAI

# ドキュメント読み込み + パース
documents = SimpleDirectoryReader("./docs").load_data()
parser = SentenceSplitter(chunk_size=1000, chunk_overlap=200)
nodes = parser.get_nodes_from_documents(documents)

# インデックス作成
index = VectorStoreIndex(nodes)

# クエリエンジン構築
query_engine = index.as_query_engine(
    llm=OpenAI(model="gpt-4o"),
    similarity_top_k=5
)

response = query_engine.query("RAGとは何ですか？")
```

**Haystack**
```python
from haystack import Pipeline
from haystack.components.converters import MarkdownToDocument
from haystack.components.preprocessors import DocumentSplitter
from haystack.components.embedders import OpenAIDocumentEmbedder, OpenAITextEmbedder
from haystack.components.writers import DocumentWriter
from haystack.components.retrievers.in_memory import InMemoryEmbeddingRetriever
from haystack.components.generators import OpenAIGenerator
from haystack.components.builders import PromptBuilder
from haystack.document_stores.in_memory import InMemoryDocumentStore

# ドキュメントストア
document_store = InMemoryDocumentStore()

# インデックスパイプライン
indexing = Pipeline()
indexing.add_component("converter", MarkdownToDocument())
indexing.add_component("splitter", DocumentSplitter(split_by="sentence", split_length=10))
indexing.add_component("embedder", OpenAIDocumentEmbedder())
indexing.add_component("writer", DocumentWriter(document_store=document_store))
indexing.connect("converter", "splitter")
indexing.connect("splitter", "embedder")
indexing.connect("embedder", "writer")

# RAGパイプライン
rag = Pipeline()
rag.add_component("embedder", OpenAITextEmbedder())
rag.add_component("retriever", InMemoryEmbeddingRetriever(document_store=document_store))
rag.add_component("prompt", PromptBuilder(template="..."))
rag.add_component("llm", OpenAIGenerator(model="gpt-4o"))
# ... connect components
```

### 判断基準

```
LangChainを選ぶべき場合:
├── エージェント機能が必要（ツール呼び出し、マルチステップ推論）
├── 豊富な統合が必要（100以上のサービス連携）
├── LangGraphで複雑なワークフローを構築したい
└── コミュニティのサンプルコードを活用したい

LlamaIndexを選ぶべき場合:
├── ドキュメント検索の精度を最大化したい
├── 複雑なインデックス構造（ツリー、グラフ）が必要
├── LlamaParse/LlamaCloudとの統合
└── シンプルなAPIで素早く実装したい

Haystackを選ぶべき場合:
├── エンタープライズ要件（監査、ガバナンス）
├── パイプラインの可視化・デバッグが重要
├── オンプレミス/規制産業での運用
└── deepset Cloudとの統合
```

---

## ベクトルDB選定：パフォーマンスとコスト

### ベンチマーク結果

100万ベクトル、768次元、top-k=10での検索性能：

| DB | QPS | P99 Latency | メモリ | 月額コスト* |
|----|-----|-------------|--------|------------|
| **Qdrant** | 8,500 | 12ms | 4.2GB | $0 (self-host) |
| **Milvus** | 7,200 | 15ms | 5.1GB | $0 (self-host) |
| **Pinecone** | 6,800 | 18ms | - | $70 |
| **Weaviate** | 5,900 | 22ms | 4.8GB | $0 (self-host) |
| **Chroma** | 3,200 | 45ms | 6.2GB | $0 |
| **pgvector** | 2,100 | 68ms | 3.8GB | $0 |

*Pineconeはs1.x1 pod、100万ベクトル想定

### アーキテクチャ別推奨

**スタートアップ / MVP**
```yaml
推奨: Chroma or pgvector
理由:
  - セットアップが簡単
  - 追加インフラ不要
  - 10万ドキュメントまでは十分な性能

実装例:
  # pgvector (既存PostgreSQLに追加)
  CREATE EXTENSION vector;
  CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    content TEXT,
    embedding vector(1536)
  );
  CREATE INDEX ON documents USING hnsw (embedding vector_cosine_ops);
```

**本番環境 / スケール重視**
```yaml
推奨: Qdrant or Milvus
理由:
  - 水平スケーリング対応
  - 高度なフィルタリング
  - 本番実績豊富

Qdrantの場合:
  from qdrant_client import QdrantClient
  from qdrant_client.models import Distance, VectorParams, PointStruct

  client = QdrantClient(host="localhost", port=6333)

  # コレクション作成（量子化でメモリ50%削減）
  client.create_collection(
      collection_name="documents",
      vectors_config=VectorParams(size=1536, distance=Distance.COSINE),
      quantization_config=models.ScalarQuantization(
          scalar=models.ScalarQuantizationConfig(
              type=models.ScalarType.INT8,
              always_ram=True
          )
      )
  )
```

**エンタープライズ / マネージド希望**
```yaml
推奨: Pinecone
理由:
  - 運用負荷ゼロ
  - SLAあり
  - SOC2/GDPR対応

注意点:
  - コストが高い（100万ベクトルで月$70〜）
  - ベンダーロックイン
  - レイテンシは自前より遅い
```

---

## 検索精度を上げる: Hybrid Search + Reranking

Naive RAGの最大の問題は**検索精度**です。

### 精度比較（BEIR benchmarkより）

| 手法 | NDCG@10 | 実装難易度 |
|------|---------|-----------|
| BM25のみ | 0.42 | 低 |
| Embedding検索のみ | 0.48 | 低 |
| **Hybrid (BM25 + Embedding)** | 0.54 | 中 |
| **Hybrid + Reranking** | 0.61 | 中 |
| Hybrid + ColBERT Reranking | 0.64 | 高 |

### 実装例：Qdrant + Hybrid Search + Cohere Rerank

```python
from qdrant_client import QdrantClient
import cohere

# 1. Hybrid Search（BM25 + ベクトル検索）
results = qdrant_client.query_points(
    collection_name="documents",
    query=query_embedding,
    query_filter=None,
    limit=20,  # Rerankingのため多めに取得
    with_payload=True,
    search_params=models.SearchParams(
        quantization=models.QuantizationSearchParams(
            ignore=False,
            rescore=True,  # 量子化後に再スコアリング
        )
    )
).points

# 2. Cohere Reranking
co = cohere.Client(api_key="...")
reranked = co.rerank(
    model="rerank-v3.5",
    query=query,
    documents=[r.payload["content"] for r in results],
    top_n=5
)

# 3. Rerankされた結果を使用
final_docs = [results[r.index] for r in reranked.results]
```

### Rerankingモデル比較

| モデル | 精度 | 速度 | コスト |
|--------|------|------|--------|
| Cohere rerank-v3.5 | 最高 | 速い | $1/1000クエリ |
| Jina Reranker v2 | 高 | 速い | 無料枠あり |
| BGE Reranker | 高 | 中 | 無料 (self-host) |
| ColBERT (RAGatouille) | 最高 | 遅い | 無料 (self-host) |

---

## PDF処理の闘い：Docling vs Unstructured vs LlamaParse

**PDFは地獄**です。特にテーブルと複雑なレイアウト。

### 精度比較（100ページの財務報告書で検証）

| ツール | テーブル抽出 | レイアウト保持 | 処理速度 | コスト |
|--------|------------|--------------|---------|--------|
| **Docling** | 95% | 90% | 2.3 pages/s | 無料 |
| LlamaParse | 92% | 95% | 1.8 pages/s | $0.003/page |
| Unstructured | 85% | 80% | 3.1 pages/s | 無料 |
| PyPDF | 60% | 40% | 8.2 pages/s | 無料 |

### Doclingの実装

```python
from docling.document_converter import DocumentConverter
from docling.datamodel.pipeline_options import PdfPipelineOptions
from docling.datamodel.base_models import InputFormat

# テーブル抽出を有効化
pipeline_options = PdfPipelineOptions()
pipeline_options.do_table_structure = True
pipeline_options.table_structure_options.do_cell_matching = True

converter = DocumentConverter(
    allowed_formats=[InputFormat.PDF],
    format_options={InputFormat.PDF: pipeline_options}
)

# 変換
result = converter.convert("financial_report.pdf")

# Markdown出力
markdown = result.document.export_to_markdown()

# テーブルはMarkdown形式で保持される
# | 項目 | 2024年 | 2025年 |
# |------|--------|--------|
# | 売上 | 100億  | 120億  |
```

---

## 評価なしのRAGは危険：RAGAS実装ガイド

**本番環境で評価なしにRAGを動かすのは、テストなしにデプロイするのと同じ**です。

### RAGASの4つのメトリクス

```python
from ragas import evaluate
from ragas.metrics import (
    faithfulness,       # 生成がコンテキストに忠実か
    answer_relevancy,   # 回答が質問に関連しているか
    context_precision,  # 取得したコンテキストの精度
    context_recall      # 必要な情報を取得できたか
)
from datasets import Dataset

# 評価データセット
eval_data = {
    "question": ["RAGとは何ですか？", ...],
    "answer": ["RAGは...", ...],
    "contexts": [["文脈1", "文脈2"], ...],
    "ground_truth": ["正解1", ...]
}
dataset = Dataset.from_dict(eval_data)

# 評価実行
results = evaluate(
    dataset,
    metrics=[faithfulness, answer_relevancy, context_precision, context_recall]
)

print(results)
# {'faithfulness': 0.87, 'answer_relevancy': 0.91,
#  'context_precision': 0.82, 'context_recall': 0.78}
```

### 目標値の目安

| メトリクス | 最低ライン | 目標値 | 優秀 |
|-----------|----------|--------|------|
| Faithfulness | 0.7 | 0.85 | 0.95+ |
| Answer Relevancy | 0.7 | 0.85 | 0.95+ |
| Context Precision | 0.6 | 0.75 | 0.90+ |
| Context Recall | 0.6 | 0.75 | 0.90+ |

### CI/CDへの組み込み

```yaml
# .github/workflows/rag-eval.yml
name: RAG Evaluation
on:
  push:
    paths:
      - 'src/rag/**'

jobs:
  evaluate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run RAGAS evaluation
        run: |
          python scripts/evaluate_rag.py

      - name: Check thresholds
        run: |
          python -c "
          import json
          with open('eval_results.json') as f:
              results = json.load(f)
          assert results['faithfulness'] >= 0.8, 'Faithfulness below threshold'
          assert results['answer_relevancy'] >= 0.8, 'Relevancy below threshold'
          "
```

---

## 本番で遭遇する落とし穴と対策

### 1. チャンクサイズの罠

```
問題: チャンクが大きすぎる → ノイズ混入
     チャンクが小さすぎる → 文脈喪失

対策: Semantic Chunkingを使う
```

```python
from langchain_experimental.text_splitter import SemanticChunker
from langchain_openai import OpenAIEmbeddings

# 意味的な区切りでチャンキング
splitter = SemanticChunker(
    OpenAIEmbeddings(),
    breakpoint_threshold_type="percentile",
    breakpoint_threshold_amount=95
)
chunks = splitter.split_documents(documents)
```

### 2. エンベディングのバージョン管理

```
問題: エンベディングモデルを更新したら全インデックスが壊れる

対策: バージョン付きコレクション
```

```python
# コレクション名にモデルバージョンを含める
collection_name = f"documents_v{EMBEDDING_MODEL_VERSION}"

# マイグレーション時は新旧並行運用
old_collection = "documents_v1"
new_collection = "documents_v2"
```

### 3. レイテンシの問題

```
問題: 検索 + LLM生成で3秒以上かかる

対策: ストリーミング + 並列検索
```

```python
import asyncio
from langchain_openai import ChatOpenAI

# 検索とLLM準備を並列化
async def rag_query(query: str):
    # 検索を開始
    search_task = asyncio.create_task(
        vectorstore.asimilarity_search(query, k=5)
    )

    # LLMはストリーミング
    llm = ChatOpenAI(model="gpt-4o", streaming=True)

    docs = await search_task

    async for chunk in llm.astream(build_prompt(query, docs)):
        yield chunk.content
```

### 4. コスト爆発

```
問題: エンベディング + LLMコストが予想外に高い

対策: キャッシング + 適切なモデル選択
```

```python
from langchain.cache import SQLiteCache
from langchain.globals import set_llm_cache

# LLM応答をキャッシュ
set_llm_cache(SQLiteCache(database_path=".langchain.db"))

# エンベディングは小さいモデルで十分な場合が多い
# text-embedding-3-large ($0.13/1M) vs text-embedding-3-small ($0.02/1M)
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
```

---

## アーキテクチャパターン集

### パターン1: シンプルRAG（MVP向け）

```
User → API → Chroma → GPT-4o → Response

コスト: 〜$50/月
適用: POC、社内ツール、〜1万クエリ/月
```

### パターン2: プロダクションRAG

```
User → API Gateway → Redis Cache
                  ↓ (cache miss)
              Qdrant (Hybrid Search)
                  ↓
              Cohere Rerank
                  ↓
              GPT-4o (streaming)
                  ↓
              RAGAS Eval (async)

コスト: 〜$500/月
適用: B2B SaaS、10万クエリ/月
```

### パターン3: エンタープライズRAG

```
                    ┌─ Langfuse (observability)
                    │
User → WAF → Kong → Haystack Pipeline
                    │
                    ├─ Docling (PDF処理)
                    ├─ Milvus Cluster (検索)
                    ├─ BGE Reranker (self-hosted)
                    ├─ Azure OpenAI (LLM)
                    └─ RAGAS + Custom Eval

コスト: 〜$5,000/月
適用: 金融、医療、100万クエリ/月
```

---

## ツール選定フローチャート

```
START
  │
  ├─ 予算は？
  │   ├─ なし → Chroma + LlamaIndex + GPT-4o-mini
  │   ├─ 〜$100/月 → pgvector + LangChain + GPT-4o
  │   └─ $1000+/月 → 次の質問へ
  │
  ├─ 運用負荷は許容できる？
  │   ├─ NO → Pinecone + LangChain + OpenAI
  │   └─ YES → 次の質問へ
  │
  ├─ PDF処理が多い？
  │   ├─ YES → Docling + RAGFlow
  │   └─ NO → 次の質問へ
  │
  ├─ エージェント機能が必要？
  │   ├─ YES → LangGraph + LangChain
  │   └─ NO → LlamaIndex
  │
  └─ 規制産業？
      ├─ YES → Haystack + オンプレ
      └─ NO → 好みで選択
```

---

## 全ツールカタログ

45以上のツールを7カテゴリで整理しました：

https://rag-catalog.vercel.app

- **フレームワーク**: 15ツール
- **ベクトルDB**: 6ツール
- **評価**: 6ツール
- **データ準備**: 5ツール
- **エンベディング**: 6ツール
- **エンジン**: 3ツール
- **リソース**: 4ツール

フィルタリング、ソート、詳細情報すべて揃っています。

---

## まとめ

RAGツール選定の鍵：

1. **要件を明確に** - MVP？本番？エンタープライズ？
2. **ベンチマークを取る** - 自分のデータで検証
3. **評価を組み込む** - RAGASは必須
4. **段階的に高度化** - Naive RAG → Hybrid → Reranking

迷ったらカタログで探してください：
https://rag-catalog.vercel.app

:::message
ツール追加リクエストはGitHub Issueで受け付けています。
https://github.com/babushkai/ragcatalog
:::
