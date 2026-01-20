---
title: "Advanced RAG完全ガイド - RAPTOR/Hybrid Search/Rerankingの実装"
emoji: "🔬"
type: "tech"
topics: ["rag", "llm", "vectordb", "検索", "ai"]
published: false
---

**「RAGを実装したけど、回答の精度がイマイチ」**

それ、**Naive RAG**の限界です。

2026年、プロダクションレベルのRAGシステムには、RAPTOR、Hybrid Search、Rerankingなどの高度な技術が必須になっています。

この記事では、RAGの精度を劇的に向上させる技術を体系的に解説します。

## RAGの進化

```
Naive RAG (2023)
├── チャンク分割 → 埋め込み → Top-K検索 → LLM生成
│   問題: 精度が低い、コンテキスト欠落

Advanced RAG (2024-2025)
├── 高度なチャンキング
├── Hybrid Search (Keyword + Semantic)
├── Reranking
│   改善: 精度向上、ただし単一ドキュメント内

Modular RAG / RAPTOR (2025-2026)
├── 階層的要約
├── 知識グラフ統合
├── 動的検索戦略
    改善: マルチホップ推論、文書間の関係理解
```

---

## チャンキング戦略

### なぜチャンキングが重要か

:::message
**チャンキングの本質:** 情報の「粒度」を決定する。粗すぎるとノイズ、細かすぎると文脈喪失。
:::

### 戦略比較

| 戦略 | 特徴 | 最適なケース |
|------|------|-------------|
| Fixed-size | シンプル、一貫性 | 均質なテキスト |
| Sentence-aware | 文境界を尊重 | 自然文書 |
| Semantic | 意味的まとまり | 複雑な文書 |
| Document-aware | 構造を維持 | 技術文書、論文 |
| Recursive | 階層的分割 | 長文書 |

### 実装

#### 1. Fixed-size Chunking

```python
def fixed_size_chunking(text, chunk_size=512, overlap=50):
    """固定サイズでチャンク分割"""
    chunks = []
    start = 0

    while start < len(text):
        end = start + chunk_size
        chunk = text[start:end]
        chunks.append(chunk)
        start = end - overlap  # オーバーラップ

    return chunks
```

**問題点:** 文の途中で切れる、意味的なまとまりが無視される

#### 2. Semantic Chunking

```python
from sentence_transformers import SentenceTransformer
import numpy as np

class SemanticChunker:
    def __init__(self, model_name="all-MiniLM-L6-v2", threshold=0.5):
        self.model = SentenceTransformer(model_name)
        self.threshold = threshold

    def chunk(self, text):
        # 1. 文に分割
        sentences = self._split_sentences(text)

        # 2. 各文を埋め込み
        embeddings = self.model.encode(sentences)

        # 3. 隣接文の類似度を計算
        similarities = []
        for i in range(len(embeddings) - 1):
            sim = self._cosine_similarity(embeddings[i], embeddings[i + 1])
            similarities.append(sim)

        # 4. 類似度が閾値を下回る箇所で分割
        chunks = []
        current_chunk = [sentences[0]]

        for i, sim in enumerate(similarities):
            if sim < self.threshold:
                # 意味的な切れ目
                chunks.append(" ".join(current_chunk))
                current_chunk = [sentences[i + 1]]
            else:
                current_chunk.append(sentences[i + 1])

        if current_chunk:
            chunks.append(" ".join(current_chunk))

        return chunks
```

#### 3. Document-aware Chunking

構造（見出し、段落、コードブロック）を尊重：

```python
class DocumentAwareChunker:
    def chunk(self, document):
        chunks = []

        # Markdownの場合
        if document.type == "markdown":
            sections = self._split_by_headers(document.content)
            for section in sections:
                # セクション内をさらに分割（必要に応じて）
                if len(section.content) > self.max_chunk_size:
                    sub_chunks = self._split_paragraphs(section.content)
                    for sub in sub_chunks:
                        chunks.append(Chunk(
                            content=sub,
                            metadata={
                                "header": section.header,
                                "level": section.level,
                            }
                        ))
                else:
                    chunks.append(Chunk(
                        content=section.content,
                        metadata={"header": section.header}
                    ))

        return chunks
```

#### 4. Contextual Chunking（Anthropic方式）

各チャンクに**文脈情報を付加**：

```python
class ContextualChunker:
    def __init__(self, llm):
        self.llm = llm

    async def chunk_with_context(self, document, chunks):
        """各チャンクに文書全体の文脈を付加"""
        contextualized = []

        for chunk in chunks:
            # LLMで文脈を生成
            context = await self.llm.generate(
                prompt=f"""
                <document>
                {document[:2000]}...  # 文書の冒頭
                </document>

                <chunk>
                {chunk.content}
                </chunk>

                このチャンクを文書全体の文脈で説明する短い文を生成してください。
                「このチャンクは...について述べています」という形式で。
                """
            )

            contextualized.append(Chunk(
                content=f"{context}\n\n{chunk.content}",
                metadata=chunk.metadata,
            ))

        return contextualized
```

**効果:** 検索精度が大幅に向上（特にvague queryに対して）

---

## Hybrid Search

### 問題: セマンティック検索の限界

```
クエリ: "エラーコード E1234 の解決方法"

セマンティック検索の結果:
1. "一般的なエラー解決のアプローチ" (類似度: 0.82)
2. "トラブルシューティングガイド" (類似度: 0.78)
3. "エラーハンドリングのベストプラクティス" (類似度: 0.75)

→ "E1234" という具体的なキーワードが無視されている！
```

### 解決: Keyword + Semantic

```python
class HybridSearch:
    def __init__(self, vector_store, bm25_index):
        self.vector_store = vector_store
        self.bm25 = bm25_index

    def search(self, query, k=10, alpha=0.5):
        """
        alpha: セマンティック検索の重み (0-1)
        1-alpha: キーワード検索の重み
        """
        # 1. セマンティック検索
        semantic_results = self.vector_store.search(
            query_embedding=self._embed(query),
            k=k * 2,  # 多めに取得
        )

        # 2. キーワード検索 (BM25)
        keyword_results = self.bm25.search(
            query=query,
            k=k * 2,
        )

        # 3. Reciprocal Rank Fusion (RRF) でマージ
        fused = self._rrf_fusion(
            semantic_results,
            keyword_results,
            alpha=alpha,
        )

        return fused[:k]

    def _rrf_fusion(self, results1, results2, alpha, k=60):
        """RRFでスコアを統合"""
        scores = {}

        # セマンティック結果のスコア
        for rank, doc in enumerate(results1):
            rrf_score = alpha * (1 / (k + rank + 1))
            scores[doc.id] = scores.get(doc.id, 0) + rrf_score

        # キーワード結果のスコア
        for rank, doc in enumerate(results2):
            rrf_score = (1 - alpha) * (1 / (k + rank + 1))
            scores[doc.id] = scores.get(doc.id, 0) + rrf_score

        # スコア順でソート
        sorted_docs = sorted(scores.items(), key=lambda x: x[1], reverse=True)
        return [self._get_doc(doc_id) for doc_id, _ in sorted_docs]
```

### alphaの調整指針

| クエリタイプ | alpha値 | 理由 |
|-------------|---------|------|
| 概念的な質問 | 0.7-0.8 | 意味的な類似性が重要 |
| 具体的な検索 | 0.3-0.4 | キーワードマッチが重要 |
| コード検索 | 0.2-0.3 | 関数名、変数名の一致が重要 |
| バランス型 | 0.5 | デフォルト |

---

## Reranking

### 問題: 初期検索の限界

初期検索（bi-encoder）は**高速だが粗い**：

```
検索クエリ: "Pythonでファイルを非同期で読み込む方法"

Top-10結果（検索後、LLMに渡す前）:
1. "Python asyncio入門" - 関連度: 中
2. "ファイルI/Oの基本" - 関連度: 低
3. "非同期プログラミング概論" - 関連度: 中
4. "aiofilesライブラリの使い方" - 関連度: 高 ★
5. ...

→ 4番目が最も関連性が高いのに、1番目ではない
```

### 解決: Cross-Encoder Reranking

```python
from sentence_transformers import CrossEncoder

class Reranker:
    def __init__(self, model_name="cross-encoder/ms-marco-MiniLM-L-6-v2"):
        self.model = CrossEncoder(model_name)

    def rerank(self, query, documents, top_k=5):
        """
        Cross-Encoderでリランキング
        """
        # クエリと各ドキュメントのペアを作成
        pairs = [(query, doc.content) for doc in documents]

        # スコア計算（Cross-Encoderは精度が高いが遅い）
        scores = self.model.predict(pairs)

        # スコア順でソート
        doc_scores = list(zip(documents, scores))
        doc_scores.sort(key=lambda x: x[1], reverse=True)

        return [doc for doc, _ in doc_scores[:top_k]]
```

### ColBERT: Late Interaction

Cross-Encoderは精度が高いが**遅い**。ColBERTは妥協点：

```python
class ColBERTReranker:
    """
    Late Interaction: トークンレベルの類似度計算
    Cross-Encoderより高速、Bi-Encoderより高精度
    """

    def __init__(self, model):
        self.model = model

    def rerank(self, query, documents):
        # 1. クエリをトークン単位で埋め込み
        query_embeddings = self.model.encode_query(query)  # shape: (q_len, dim)

        scores = []
        for doc in documents:
            # 2. ドキュメントをトークン単位で埋め込み
            doc_embeddings = self.model.encode_doc(doc.content)  # shape: (d_len, dim)

            # 3. MaxSim: 各クエリトークンに最も近いドキュメントトークンを見つける
            # score = sum of max similarities for each query token
            similarity_matrix = query_embeddings @ doc_embeddings.T
            max_sims = similarity_matrix.max(dim=1).values
            score = max_sims.sum()

            scores.append((doc, score))

        # ソート
        scores.sort(key=lambda x: x[1], reverse=True)
        return [doc for doc, _ in scores]
```

### Rerankingパイプライン

```python
class RAGPipeline:
    def __init__(self, retriever, reranker, llm):
        self.retriever = retriever
        self.reranker = reranker
        self.llm = llm

    async def query(self, question):
        # 1. 初期検索（広く、多めに）
        initial_results = await self.retriever.search(
            question,
            k=50,  # 多めに取得
        )

        # 2. リランキング（精度向上）
        reranked = self.reranker.rerank(
            question,
            initial_results,
            top_k=5,  # 絞り込み
        )

        # 3. LLM生成
        context = self._format_context(reranked)
        response = await self.llm.generate(
            system="以下のコンテキストに基づいて回答してください。",
            user=f"コンテキスト:\n{context}\n\n質問: {question}",
        )

        return response
```

---

## RAPTOR: 階層的検索

### 問題: フラットなチャンクの限界

```
質問: "このプロジェクト全体のアーキテクチャを説明して"

従来のRAG:
- 個別のコードファイルのチャンクを取得
- 全体像が見えない
- 関連するチャンクが散在
```

### RAPTOR (Recursive Abstractive Processing for Tree-Organized Retrieval)

**アイデア:** チャンクを階層的にクラスタリング・要約し、抽象度の異なるレベルで検索

```
                    [プロジェクト全体の要約]
                           Level 3
                    /                    \
        [フロントエンドの要約]        [バックエンドの要約]
              Level 2                    Level 2
           /        \                  /        \
    [React関連]  [状態管理]      [API設計]   [DB設計]
      Level 1     Level 1        Level 1    Level 1
       /   \         |             |          |
    [チャンク群]  [チャンク群]  [チャンク群] [チャンク群]
      Level 0     Level 0      Level 0    Level 0
```

### 実装

```python
class RAPTOR:
    def __init__(self, embedder, llm, vector_store):
        self.embedder = embedder
        self.llm = llm
        self.store = vector_store

    def build_tree(self, documents, max_levels=3):
        """RAPTORツリーを構築"""
        # Level 0: 元のチャンク
        current_level = self._chunk_documents(documents)
        all_nodes = list(current_level)

        for level in range(1, max_levels + 1):
            if len(current_level) <= 1:
                break

            # 1. 埋め込みを計算
            embeddings = self.embedder.encode([n.content for n in current_level])

            # 2. クラスタリング
            clusters = self._cluster(embeddings, min_cluster_size=3)

            # 3. 各クラスタを要約
            next_level = []
            for cluster_indices in clusters:
                cluster_nodes = [current_level[i] for i in cluster_indices]

                # LLMで要約を生成
                summary = self._summarize(cluster_nodes)

                summary_node = Node(
                    content=summary,
                    level=level,
                    children=[n.id for n in cluster_nodes],
                )
                next_level.append(summary_node)
                all_nodes.append(summary_node)

            current_level = next_level

        # 全ノードをベクトルストアに保存
        self._store_all(all_nodes)

        return all_nodes

    def search(self, query, k=5):
        """
        全レベルから検索
        - 具体的な質問 → 下位レベル（詳細なチャンク）
        - 抽象的な質問 → 上位レベル（要約）
        """
        results = self.store.search(
            query_embedding=self.embedder.encode(query),
            k=k,
            # 全レベルから検索
        )

        return results

    def _summarize(self, nodes):
        """クラスタ内のノードを要約"""
        combined = "\n\n".join([n.content for n in nodes])

        prompt = f"""
        以下の関連するテキストを、重要な情報を保持しながら要約してください。

        テキスト:
        {combined}

        要約:
        """
        return self.llm.generate(prompt)
```

### RAPTORの効果

| クエリタイプ | 従来RAG | RAPTOR |
|-------------|---------|--------|
| 詳細な質問 | ○ | ○ |
| 全体像の質問 | △ | ◎ |
| マルチホップ | △ | ○ |

---

## 高度なパターン

### Query Transformation

検索前にクエリを変換：

```python
class QueryTransformer:
    def __init__(self, llm):
        self.llm = llm

    async def expand_query(self, query):
        """クエリ拡張: 複数の言い換えを生成"""
        prompt = f"""
        以下の検索クエリを、同じ意図を持つ3つの異なる言い方に変換してください。

        元のクエリ: {query}

        変換後:
        1.
        2.
        3.
        """
        expanded = await self.llm.generate(prompt)
        return self._parse(expanded)

    async def decompose_query(self, query):
        """複雑なクエリを分解"""
        prompt = f"""
        以下の複雑な質問を、検索しやすい単純な質問に分解してください。

        質問: {query}

        分解:
        """
        decomposed = await self.llm.generate(prompt)
        return self._parse(decomposed)

    async def hypothetical_document(self, query):
        """HyDE: 仮想的な回答を生成して検索"""
        prompt = f"""
        以下の質問に対する理想的な回答を生成してください。
        （実際の情報がなくても、あるべき回答を想像して）

        質問: {query}

        理想的な回答:
        """
        hypothetical = await self.llm.generate(prompt)
        return hypothetical  # これを埋め込みとして検索に使用
```

### Self-RAG

LLMが自分で検索の必要性を判断：

```python
class SelfRAG:
    def __init__(self, llm, retriever):
        self.llm = llm
        self.retriever = retriever

    async def generate(self, query):
        # 1. 検索が必要か判断
        needs_retrieval = await self._judge_retrieval_need(query)

        if needs_retrieval:
            # 2. 検索実行
            docs = await self.retriever.search(query)

            # 3. 各ドキュメントの関連性を評価
            relevant_docs = []
            for doc in docs:
                is_relevant = await self._judge_relevance(query, doc)
                if is_relevant:
                    relevant_docs.append(doc)

            # 4. 回答生成
            response = await self._generate_with_context(query, relevant_docs)

            # 5. 回答の品質を自己評価
            quality = await self._judge_quality(query, response, relevant_docs)

            if quality.needs_more_info:
                # 追加検索
                more_docs = await self._search_for_gaps(query, response, quality.gaps)
                response = await self._refine_response(response, more_docs)

            return response
        else:
            # 検索なしで直接回答
            return await self.llm.generate(query)
```

---

## パフォーマンス最適化

### インデックス戦略

```python
# HNSW (Hierarchical Navigable Small World) パラメータ
index_config = {
    "M": 16,           # 各ノードの接続数（大きいほど精度↑、メモリ↑）
    "ef_construction": 200,  # 構築時の探索幅（大きいほど精度↑、構築時間↑）
    "ef_search": 100,  # 検索時の探索幅（大きいほど精度↑、レイテンシ↑）
}
```

### バッチ処理

```python
class BatchedRetriever:
    def __init__(self, retriever, batch_size=32):
        self.retriever = retriever
        self.batch_size = batch_size

    async def search_batch(self, queries):
        """複数クエリをバッチ処理"""
        results = []

        for i in range(0, len(queries), self.batch_size):
            batch = queries[i:i + self.batch_size]
            # 並列実行
            batch_results = await asyncio.gather(*[
                self.retriever.search(q) for q in batch
            ])
            results.extend(batch_results)

        return results
```

---

## まとめ

### RAG最適化チェックリスト

```markdown
□ チャンキング戦略は適切か？
  - 文書タイプに合った方法を選択
  - Contextual chunkingを検討

□ Hybrid Searchを実装しているか？
  - Keyword + Semantic
  - alphaを適切に調整

□ Rerankingを導入しているか？
  - Cross-Encoder or ColBERT
  - 初期検索は広く、リランクで絞る

□ 階層的検索を検討したか？
  - RAPTORで抽象度の異なるレベル
  - 全体像を問う質問に対応

□ クエリ変換を使っているか？
  - クエリ拡張
  - HyDE
```

### 精度向上の優先順位

1. **チャンキング最適化** - 基盤
2. **Hybrid Search** - 即効性あり
3. **Reranking** - 精度の決め手
4. **RAPTOR** - 複雑なクエリ対応
5. **Query Transformation** - さらなる改善

---

## 参考リンク

- [RAG Techniques (GitHub)](https://github.com/NirDiamant/RAG_Techniques)
- [Optimizing RAG with Hybrid Search & Reranking](https://superlinked.com/vectorhub/articles/optimizing-rag-with-hybrid-search-reranking)
- [Improving RAG with RAPTOR](https://superlinked.com/vectorhub/articles/improve-rag-with-raptor)
- [Advanced RAG Techniques (Neo4j)](https://neo4j.com/blog/genai/advanced-rag-techniques/)

---

**あなたのRAGシステムではどんな工夫をしていますか？コメントで教えてください！**
