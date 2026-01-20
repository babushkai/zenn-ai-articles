---
title: "Context Engineering完全ガイド - プロンプトエンジニアリングの次へ"
emoji: "🧠"
type: "tech"
topics: ["ai", "llm", "contextengineering", "rag", "aiagent"]
published: false
---

**「プロンプトを改善しても、もう精度が上がらない」**

この壁にぶつかったことはありませんか？

それは「プロンプトエンジニアリング」の限界に到達したサインです。

次のステージは**Context Engineering（コンテキストエンジニアリング）**。Anthropicが提唱する、AIエージェント時代の新しいパラダイムです。

## プロンプトエンジニアリングとの違い

:::message
**Prompt Engineering:** 「何を言うか」を最適化
**Context Engineering:** 「何を見せるか」を最適化
:::

### プロンプトエンジニアリングの限界

```python
# 従来のアプローチ
prompt = """
あなたは優秀なエンジニアです。
以下のコードをレビューしてください。
バグがあれば指摘してください。
セキュリティリスクも確認してください。
パフォーマンス改善案も出してください。

{code}
"""
```

問題点：
- **静的**: 同じプロンプトを全てのケースに適用
- **限定的**: プロンプト内の指示のみに依存
- **スケールしない**: 複雑なタスクで破綻

### Context Engineeringのアプローチ

```python
def build_context(code, repo_context, user_history, project_rules):
    """
    コンテキストを動的に構築
    """
    context = {
        # 関連コードのみを選択的にロード
        "related_files": find_related_files(code),

        # プロジェクト固有のルール
        "coding_standards": project_rules.get_relevant(code),

        # 過去の類似レビュー結果
        "similar_reviews": retrieve_similar_reviews(code),

        # 現在の変更差分
        "git_diff": get_staged_changes(),
    }
    return optimize_for_token_budget(context, max_tokens=50000)
```

**コンテキストエンジニアリングの核心:**

> 常に変化する情報の宇宙から、限られたコンテキストウィンドウに**何を入れるか**を設計する技術

---

## 階層的コンテキスト構造

Anthropicが推奨する4層構造：

```
┌─────────────────────────────────────────┐
│         System Layer                     │
│   エージェントのアイデンティティと能力     │
├─────────────────────────────────────────┤
│         Task Layer                       │
│   現在のタスクに関する具体的な指示         │
├─────────────────────────────────────────┤
│         Tool Layer                       │
│   利用可能なツールの説明と使用方法         │
├─────────────────────────────────────────┤
│         Memory Layer                     │
│   関連する履歴コンテキストと学習内容       │
└─────────────────────────────────────────┘
```

### 実装例

```python
class ContextBuilder:
    def __init__(self, agent_config):
        self.agent_config = agent_config

    def build(self, task, available_tools, memory_store):
        return {
            "system": self._build_system_layer(),
            "task": self._build_task_layer(task),
            "tools": self._build_tool_layer(available_tools),
            "memory": self._build_memory_layer(task, memory_store),
        }

    def _build_system_layer(self):
        """
        エージェントの基本的なアイデンティティ
        - 変更頻度: 低（セッション単位）
        - トークン予算: 500-1000
        """
        return {
            "role": self.agent_config.role,
            "capabilities": self.agent_config.capabilities,
            "constraints": self.agent_config.constraints,
        }

    def _build_task_layer(self, task):
        """
        現在のタスクに関する指示
        - 変更頻度: 高（タスク単位）
        - トークン予算: 1000-3000
        """
        return {
            "objective": task.objective,
            "success_criteria": task.success_criteria,
            "constraints": task.constraints,
        }

    def _build_tool_layer(self, available_tools):
        """
        利用可能なツール
        - MCP Tool Searchで動的に最適化可能
        - トークン予算: 2000-10000（ツール数による）
        """
        # 関連性の高いツールのみを選択
        relevant_tools = self._filter_relevant_tools(available_tools)
        return [tool.to_schema() for tool in relevant_tools]

    def _build_memory_layer(self, task, memory_store):
        """
        関連する過去のコンテキスト
        - トークン予算: 残り全て
        """
        return {
            "episodic": memory_store.retrieve_episodic(task, k=5),
            "semantic": memory_store.retrieve_semantic(task, k=10),
            "procedural": memory_store.retrieve_procedural(task),
        }
```

---

## 高度なテクニック

### 1. マスキングとフィルタリング

全てのコンテキストを常に見せる必要はありません。

```python
class ContextMasker:
    def __init__(self):
        self.masks = {
            "planning": ["implementation_details", "test_results"],
            "implementation": ["design_discussions"],
            "review": ["draft_versions"],
        }

    def apply_mask(self, context, current_phase):
        """
        フェーズに応じて不要な情報をマスク
        """
        masked = copy.deepcopy(context)
        for key in self.masks.get(current_phase, []):
            if key in masked:
                masked[key] = "[MASKED]"
        return masked
```

**なぜマスキングが重要か？**
- ノイズの削減 → 精度向上
- トークン節約 → コスト削減
- 焦点の維持 → タスク完遂率向上

### 2. 階層化メモリ（Layered Memory）

人間の記憶システムを模倣：

```python
class LayeredMemory:
    def __init__(self):
        # 作業記憶（現在のコンテキストウィンドウ）
        self.working_memory = []

        # 短期記憶（セッション内、圧縮済み）
        self.short_term = []

        # 長期記憶（永続化、ベクトルDB）
        self.long_term = VectorStore()

    def remember(self, information, importance_score):
        self.working_memory.append(information)

        if len(self.working_memory) > self.working_limit:
            # 重要度に基づいて圧縮・移動
            to_compress = self._select_for_compression()
            compressed = self._compress(to_compress)
            self.short_term.append(compressed)

        if importance_score > 0.8:
            # 重要な情報は長期記憶へ
            self.long_term.store(information)

    def recall(self, query, context_budget):
        """
        クエリに関連する記憶を予算内で取得
        """
        results = []

        # まず作業記憶から
        results.extend(self._recall_from_working(query))

        # 予算が残っていれば短期記憶から
        if self._has_budget(results, context_budget):
            results.extend(self._recall_from_short_term(query))

        # さらに残っていれば長期記憶から
        if self._has_budget(results, context_budget):
            results.extend(self.long_term.search(query))

        return self._fit_to_budget(results, context_budget)
```

### 3. Cognitive Workspace パラダイム

従来のRAGを超えた、認知的なアプローチ：

```python
class CognitiveWorkspace:
    """
    受動的な検索ではなく、能動的にコンテキストをキュレーション
    """

    def __init__(self):
        self.workspace = {}
        self.attention_weights = {}

    def curate(self, task, available_information):
        """
        何を保持し、何を圧縮し、何を破棄するかを決定
        """
        for info in available_information:
            relevance = self._calculate_relevance(info, task)
            recency = self._calculate_recency(info)
            importance = self._calculate_importance(info)

            score = self._weighted_score(relevance, recency, importance)

            if score > 0.7:
                # 高スコア: そのまま保持
                self.workspace[info.id] = info
            elif score > 0.4:
                # 中スコア: 圧縮して保持
                self.workspace[info.id] = self._compress(info)
            # 低スコア: 破棄

        return self.workspace
```

**研究結果:** Cognitive Workspaceパラダイムは、単純なRAGと比較して**58.6%のメモリ再利用率**を達成（RAGは0%）。

---

## MCP（Model Context Protocol）との関係

Context EngineeringとMCPは補完関係にあります：

```
Context Engineering
├── 何をコンテキストに入れるかの設計思想
│
MCP (Model Context Protocol)
├── 外部ツール・データソースとの接続規格
│
実装
├── Context Engineeringの原則に基づき
└── MCPでツール・データを接続
```

### MCP Tool Searchの活用

多数のMCPサーバーがある場合、全てのツール定義をコンテキストに入れると膨大なトークンを消費：

```
従来: 50ツール × 1500トークン = 75,000トークン（ツール定義だけで）
```

MCP Tool Searchは、タスクに関連するツールのみを動的にロード：

```
Tool Search: 必要な5ツールのみ = 7,500トークン（90%削減）
```

---

## 実践的なコンテキスト最適化

### トークン予算の配分指針

```
総予算: 100,000トークン の場合

┌─────────────────────────────────────────┐
│ System Layer      │  1,000 (1%)        │
│ Task Layer        │  3,000 (3%)        │
│ Tool Layer        │ 10,000 (10%)       │
│ Memory Layer      │ 36,000 (36%)       │
│ 現在のタスク入力   │ 30,000 (30%)       │
│ 出力用バッファ    │ 20,000 (20%)       │
└─────────────────────────────────────────┘
```

### コンテキスト品質チェックリスト

```markdown
□ 関連性: 現在のタスクに直接関係する情報のみか？
□ 新鮮さ: 古い情報が残っていないか？
□ 重複排除: 同じ情報が複数回含まれていないか？
□ 階層化: 重要度に応じた構造になっているか？
□ 予算遵守: トークン予算内に収まっているか？
```

---

## ACE: Agentic Context Engineering

最新の研究動向として、**エージェント自身がコンテキストを最適化**する方向性：

```python
class AgenticContextEngineering:
    """
    エージェントが自律的にコンテキストを進化させる
    """

    def __init__(self, agent, feedback_system):
        self.agent = agent
        self.feedback = feedback_system
        self.context_playbook = {}

    def evolve_context(self, task_result):
        """
        タスク結果に基づいてコンテキスト戦略を更新
        """
        if task_result.success:
            # 成功したコンテキスト構成を記録
            self.context_playbook[task_result.task_type] = {
                "context_config": task_result.context_used,
                "success_rate": self._update_success_rate(task_result),
            }
        else:
            # 失敗から学習
            self._analyze_failure(task_result)
            self._adjust_context_strategy(task_result)

    def get_optimal_context(self, new_task):
        """
        過去の学習に基づいて最適なコンテキストを構築
        """
        similar_tasks = self._find_similar_tasks(new_task)
        best_config = self._select_best_config(similar_tasks)
        return self._adapt_config(best_config, new_task)
```

---

## まとめ

:::message
**Context Engineeringの本質:**
LLMに「何を見せるか」を戦略的に設計し、限られたコンテキストウィンドウの価値を最大化する技術
:::

### 重要ポイント

| 概念 | 説明 |
|------|------|
| 階層的構造 | System → Task → Tool → Memory の4層 |
| マスキング | フェーズに応じて不要情報を隠す |
| 階層化メモリ | 作業・短期・長期の3層記憶 |
| Cognitive Workspace | 能動的なコンテキストキュレーション |
| ACE | エージェント自身によるコンテキスト最適化 |

### プロンプトエンジニアリングとの共存

> 「プロンプトは意図を設定し、コンテキストは状況認識を提供する」

両方が必要です。しかし、エンタープライズ環境で精度・メモリ・ガバナンスが求められる場合、**コンテキストエンジニアリングなしでは戦えません**。

---

## 参考リンク

- [Anthropic - Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Context Engineering Guide | Prompt Engineering Guide](https://www.promptingguide.ai/guides/context-engineering-guide)
- [Hugging Face - Context Engineering: The Evolution](https://huggingface.co/blog/Svngoku/context-engineering-the-evolution-beyond-prompt-en)
- [CIO - Context Engineering: Improving AI by Moving Beyond the Prompt](https://www.cio.com/article/4080592/context-engineering-improving-ai-by-moving-beyond-the-prompt.html)

---

**コンテキストエンジニアリングを実践している方、どんな工夫をしていますか？コメントで共有してください！**
