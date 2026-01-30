---
title: "マルチエージェントオーケストレーション設計パターン完全ガイド"
emoji: "🎭"
type: "tech"
topics: ["ai", "llm", "マルチエージェント", "設計パターン", "アーキテクチャ"]
published: false
---

**「シングルエージェントで限界を感じている」**

タスクが複雑になると、1つのエージェントでは対応しきれなくなります。

解決策は**マルチエージェントシステム**。しかし、適切な設計パターンを選ばないと、カオスになります。

この記事では、2026年時点で確立された**マルチエージェントオーケストレーションの設計パターン**を体系的に解説します。

## なぜマルチエージェントが必要か

### シングルエージェントの限界

```
タスク: 「新機能の設計、実装、テスト、ドキュメント作成、デプロイ」

シングルエージェント:
- コンテキストウィンドウの圧迫
- 専門性の希薄化
- エラー時の全体破綻
- 並列処理不可
```

### マルチエージェントの利点

```
同じタスクをマルチエージェントで:

┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ Architect   │    │ Developer   │    │ QA Engineer │
│ 設計専門    │───►│ 実装専門    │───►│ テスト専門   │
└─────────────┘    └─────────────┘    └─────────────┘
       │                                      │
       └──────────────┬───────────────────────┘
                      ▼
              ┌─────────────┐
              │ Tech Writer │
              │ ドキュメント │
              └─────────────┘
```

**利点:**
- 専門性の最大化
- 並列処理可能
- 障害の局所化
- スケーラビリティ

---

## 5つの基本パターン

### Pattern 1: Coordinator-Worker（コーディネーター・ワーカー）

**最もシンプルで汎用的なパターン**

```
                    ┌─────────────────┐
                    │   Coordinator    │
                    │  タスク分解・統合 │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        ▼                    ▼                    ▼
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│  Worker A   │      │  Worker B   │      │  Worker C   │
│  専門タスク  │      │  専門タスク  │      │  専門タスク  │
└─────────────┘      └─────────────┘      └─────────────┘
```

#### 実装例

```python
class Coordinator:
    def __init__(self, workers: dict[str, Worker]):
        self.workers = workers

    async def execute(self, task: Task) -> Result:
        # 1. タスクを分解
        subtasks = await self._decompose(task)

        # 2. ワーカーに割り当て
        assignments = self._assign_to_workers(subtasks)

        # 3. 並列実行
        results = await asyncio.gather(*[
            self.workers[worker_type].execute(subtask)
            for worker_type, subtask in assignments
        ])

        # 4. 結果を統合
        return await self._aggregate(results)

    async def _decompose(self, task: Task) -> list[Subtask]:
        """LLMを使ってタスクを分解"""
        response = await self.llm.generate(
            system="タスクを独立した小タスクに分解してください",
            user=task.description,
            response_format=SubtaskList,
        )
        return response.subtasks

    def _assign_to_workers(self, subtasks: list[Subtask]) -> list[tuple]:
        """サブタスクを適切なワーカーに割り当て"""
        assignments = []
        for subtask in subtasks:
            worker_type = self._select_worker(subtask)
            assignments.append((worker_type, subtask))
        return assignments
```

#### ユースケース

- コードレビュー（セキュリティ、パフォーマンス、可読性を別々に）
- ドキュメント生成（API、チュートリアル、リファレンスを別々に）
- データパイプライン（抽出、変換、ロードを別々に）

---

### Pattern 2: Hierarchical（階層型）

**大規模・複雑なタスク向け**

```
                         ┌─────────────┐
                         │   Director   │
                         │  戦略決定    │
                         └──────┬──────┘
                                │
              ┌─────────────────┼─────────────────┐
              ▼                 ▼                 ▼
       ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
       │  Manager A  │  │  Manager B  │  │  Manager C  │
       │  チーム管理  │  │  チーム管理  │  │  チーム管理  │
       └──────┬──────┘  └──────┬──────┘  └──────┬──────┘
              │                 │                 │
        ┌─────┴─────┐    ┌─────┴─────┐    ┌─────┴─────┐
        ▼           ▼    ▼           ▼    ▼           ▼
    ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐
    │Worker│ │Worker│ │Worker│ │Worker│ │Worker│ │Worker│
    └───────┘ └───────┘ └───────┘ └───────┘ └───────┘ └───────┘
```

#### 実装例

```python
class HierarchicalSwarm:
    def __init__(self):
        self.director = DirectorAgent()
        self.managers = {}
        self.workers = {}

    async def execute(self, project: Project) -> Result:
        # 1. Directorが戦略を決定
        strategy = await self.director.plan(project)

        # 2. 各Managerにドメインを割り当て
        domain_results = []
        for domain, manager in self.managers.items():
            if domain in strategy.domains:
                # Managerがサブチームを管理
                result = await manager.execute(
                    strategy.get_domain_tasks(domain),
                    workers=self.workers[domain],
                )
                domain_results.append(result)

        # 3. Directorが結果を統合・評価
        final = await self.director.integrate(domain_results)

        # 4. 必要に応じてフィードバックループ
        if not final.meets_criteria:
            return await self._refine(final)

        return final

class ManagerAgent:
    async def execute(self, tasks, workers):
        """ワーカーチームを管理"""
        # タスクを分配
        assignments = self._distribute(tasks, workers)

        # 進捗監視とリソース再配分
        while not self._all_complete(assignments):
            for worker, task in assignments:
                status = await worker.get_status()
                if status.stuck:
                    # 別のワーカーに再割り当て
                    await self._reassign(task)

        return self._collect_results(assignments)
```

#### ユースケース

- 大規模ソフトウェアプロジェクト
- 組織横断的なタスク
- 長期間にわたるプロジェクト管理

---

### Pattern 3: Pipeline（パイプライン）

**順序依存のワークフロー向け**

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Stage 1   │───►│   Stage 2   │───►│   Stage 3   │───►│   Stage 4   │
│   入力処理   │    │   分析      │    │   生成      │    │   検証      │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

#### 実装例

```python
class Pipeline:
    def __init__(self, stages: list[Agent]):
        self.stages = stages

    async def execute(self, input_data) -> Result:
        current = input_data

        for i, stage in enumerate(self.stages):
            try:
                # 各ステージの出力が次の入力
                current = await stage.process(current)

                # 中間結果をログ
                self._log_intermediate(i, current)

            except StageError as e:
                # エラー処理: リトライまたは中断
                if e.recoverable:
                    current = await self._retry(stage, current)
                else:
                    raise PipelineError(f"Stage {i} failed", stage=i)

        return current

# 使用例: コードレビューパイプライン
review_pipeline = Pipeline([
    SyntaxCheckerAgent(),      # Stage 1: 構文チェック
    SecurityScannerAgent(),    # Stage 2: セキュリティスキャン
    PerformanceAnalyzerAgent(),# Stage 3: パフォーマンス分析
    ReviewSummarizerAgent(),   # Stage 4: レビューサマリ生成
])
```

#### ユースケース

- CI/CDパイプライン
- ETL処理
- コンテンツ生成（調査→執筆→編集→公開）

---

### Pattern 4: Swarm（スウォーム）

**動的・自律的な協調向け**

```
    ┌─────────────┐
    │   Agent A   │◄─────┐
    └──────┬──────┘      │
           │             │
           ▼             │
    ┌─────────────┐      │
┌──►│   Agent B   │◄─────┼──────┐
│   └──────┬──────┘      │      │
│          │             │      │
│          ▼             │      │
│   ┌─────────────┐      │      │
│   │   Agent C   │──────┘      │
│   └──────┬──────┘             │
│          │                    │
│          ▼                    │
│   ┌─────────────┐             │
└───│   Agent D   │─────────────┘
    └─────────────┘

    ※ 中央制御なし、各エージェントが自律的に協調
```

#### 実装例

```python
class SwarmAgent:
    def __init__(self, agent_id, capabilities, swarm_network):
        self.id = agent_id
        self.capabilities = capabilities
        self.network = swarm_network

    async def run(self):
        """自律的に動作"""
        while True:
            # 1. タスクキューを確認
            task = await self.network.get_available_task(self.capabilities)

            if task:
                # 2. 自分で処理できるか判断
                if self._can_handle(task):
                    result = await self._execute(task)
                    await self.network.report_result(task.id, result)
                else:
                    # 3. 他のエージェントに委譲
                    await self._delegate(task)

            # 4. 他のエージェントからの依頼を確認
            requests = await self.network.get_requests(self.id)
            for req in requests:
                await self._handle_request(req)

    async def _delegate(self, task):
        """タスクを適切なエージェントに委譲"""
        # 能力マッチングで最適なエージェントを探す
        candidates = await self.network.find_agents(
            required_capabilities=task.required_capabilities,
            exclude=[self.id],
        )

        if candidates:
            best = self._select_best(candidates, task)
            await self.network.request_help(best, task)
```

#### ユースケース

- 探索タスク（Webクローリング、情報収集）
- 負荷分散が必要な処理
- 耐障害性が重要なシステム

---

### Pattern 5: Debate/Adversarial（討論型）

**品質向上・検証向け**

```
                    ┌─────────────────┐
                    │      Judge      │
                    │   最終判定      │
                    └────────┬────────┘
                             │
                    ┌────────┴────────┐
                    │                 │
                    ▼                 ▼
             ┌─────────────┐   ┌─────────────┐
             │  Proposer   │◄─►│  Critic     │
             │  提案者      │   │  批評者      │
             └─────────────┘   └─────────────┘
```

#### 実装例

```python
class DebateOrchestrator:
    def __init__(self, proposer, critic, judge, max_rounds=3):
        self.proposer = proposer
        self.critic = critic
        self.judge = judge
        self.max_rounds = max_rounds

    async def execute(self, task) -> Result:
        # 1. 初期提案
        proposal = await self.proposer.generate(task)

        for round in range(self.max_rounds):
            # 2. 批評
            critique = await self.critic.evaluate(proposal)

            # 3. 判定: 十分な品質か？
            judgment = await self.judge.assess(proposal, critique)

            if judgment.approved:
                return Result(
                    content=proposal,
                    rounds=round + 1,
                    quality_score=judgment.score,
                )

            # 4. 批評を元に改善
            proposal = await self.proposer.refine(proposal, critique)

        # 最大ラウンド到達
        return Result(
            content=proposal,
            rounds=self.max_rounds,
            quality_score=judgment.score,
            warning="Max rounds reached without full approval",
        )

# 使用例: コード品質向上
code_debate = DebateOrchestrator(
    proposer=CodeWriterAgent(),
    critic=CodeReviewerAgent(),
    judge=QualityAssessorAgent(),
)
```

#### ユースケース

- コード品質の向上
- 法的文書のレビュー
- 研究論文の検証
- 意思決定の妥当性確認

---

## パターン選択ガイド

| パターン | 最適な状況 | 避けるべき状況 |
|----------|-----------|---------------|
| **Coordinator-Worker** | 並列処理可能なタスク | 順序依存が強いタスク |
| **Hierarchical** | 大規模・長期プロジェクト | 単純なタスク |
| **Pipeline** | 明確なステージがあるワークフロー | 並列処理が必要な場合 |
| **Swarm** | 動的・探索的なタスク | 厳密な順序が必要な場合 |
| **Debate** | 品質が最重要な出力 | 速度が最重要な場合 |

---

## 高度なトピック

### ハイブリッドパターン

実際のシステムでは、複数のパターンを組み合わせます：

```python
class HybridOrchestration:
    """
    Hierarchical + Pipeline + Debate の組み合わせ
    """
    async def execute(self, project):
        # 1. Hierarchical: 全体管理
        director = DirectorAgent()
        strategy = await director.plan(project)

        # 2. Pipeline: 各フェーズを順次実行
        for phase in strategy.phases:
            # 3. Coordinator-Worker: フェーズ内は並列
            results = await self._parallel_execute(phase.tasks)

            # 4. Debate: 各フェーズの成果物を検証
            validated = await self._validate_with_debate(results)

            if not validated.approved:
                # リトライ
                results = await self._refine_and_retry(phase, validated.feedback)

        return await director.finalize(results)
```

### 通信プロトコル

エージェント間通信の設計も重要：

```python
class AgentMessage:
    """A2Aプロトコルに準拠したメッセージ"""
    sender: str
    receiver: str
    message_type: Literal["request", "response", "notification"]
    payload: dict
    correlation_id: str  # リクエスト-レスポンスの対応付け
    timestamp: datetime

class AgentNetwork:
    async def send(self, message: AgentMessage):
        """メッセージ送信（非同期）"""
        await self.message_queue.publish(
            topic=message.receiver,
            message=message.to_json(),
        )

    async def receive(self, agent_id: str) -> AgentMessage:
        """メッセージ受信"""
        raw = await self.message_queue.subscribe(topic=agent_id)
        return AgentMessage.from_json(raw)
```

### 障害処理

```python
class FaultTolerantOrchestrator:
    async def execute_with_fallback(self, task, primary_worker, fallback_workers):
        try:
            return await asyncio.wait_for(
                primary_worker.execute(task),
                timeout=30.0,
            )
        except (asyncio.TimeoutError, WorkerError) as e:
            # フォールバック
            for fallback in fallback_workers:
                try:
                    return await fallback.execute(task)
                except WorkerError:
                    continue

            raise AllWorkersFailedError(task)
```

---

## 2026年のフレームワーク比較

| フレームワーク | 強み | 弱み |
|---------------|------|------|
| **LangGraph** | グラフベース、柔軟 | 学習曲線 |
| **OpenAI Agents SDK** | シンプル、本番品質 | カスタマイズ限定 |
| **Microsoft Agent Framework** | エンタープライズ統合 | 重い |
| **Swarms** | 大規模スウォーム | 複雑 |

:::message
**2026年の現実:** フレームワークの性能差は、主に**ツール実行とコンテキスト合成**で生じる。エージェント間のハンドオフ自体は大差ない。
:::

---

## まとめ

### 5つの基本パターン

1. **Coordinator-Worker**: シンプルで汎用的
2. **Hierarchical**: 大規模プロジェクト向け
3. **Pipeline**: 順序依存ワークフロー
4. **Swarm**: 自律・動的協調
5. **Debate**: 品質重視の検証

### 設計原則

- **単一責任**: 各エージェントは1つの専門領域
- **疎結合**: エージェント間の依存を最小化
- **障害隔離**: 1つの失敗が全体に波及しない
- **観測可能性**: 全てのやり取りをログ

---

## 参考リンク

- [Choosing the Right Orchestration Pattern (Kore.ai)](https://www.kore.ai/blog/choosing-the-right-orchestration-pattern-for-multi-agent-systems)
- [Design Patterns for Multi-Agent Orchestration](https://www.wethinkapp.ai/blog/design-patterns-for-multi-agent-orchestration)
- [Swarms Documentation](https://docs.swarms.world/en/latest/swarms/concept/swarm_architectures/)
- [Multi-Agent Collaboration via Evolving Orchestration (arXiv)](https://arxiv.org/html/2505.19591v1)

---

**あなたのプロジェクトではどのパターンを使っていますか？実装で困ったことがあればコメントで共有してください！**
