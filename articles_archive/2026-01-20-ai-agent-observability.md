---
title: "AIエージェントのObservability完全ガイド - 本番運用のトレーシングと評価"
emoji: "🔭"
type: "tech"
topics: ["ai", "observability", "llm", "トレーシング", "監視"]
published: false
---

**「本番で動いているエージェントが何をしているか、把握できていますか？」**

LLMエージェントの本番運用で最も見落とされがちなのが**Observability（可観測性）**です。

2026年のデータによると、エージェントを本番運用している組織の**94%**が何らかの可観測性ツールを導入し、**71.5%**が詳細なトレーシングを実装しています。

この記事では、AIエージェントのObservabilityを体系的に解説します。

## なぜエージェントにObservabilityが必要か

### 従来のソフトウェアとの違い

```
従来のソフトウェア:
Input → [決定論的処理] → Output
- 同じ入力 → 同じ出力
- デバッグ: ログとスタックトレース

AIエージェント:
Input → [LLM判断] → [ツール実行] → [LLM判断] → ... → Output
- 同じ入力 → 異なる出力（非決定論的）
- 中間の「思考」が見えない
- 失敗原因の特定が困難
```

### 可観測性がないとどうなるか

```
問題発生時:

サポート: 「エージェントが間違った回答をしました」
開発者: 「ログを見ます...」

ログ:
[INFO] Request received
[INFO] Response sent

開発者: 「何が起きたか全く分かりません」
```

---

## エージェントObservabilityの3つの柱

### 1. Tracing（トレーシング）

**エージェントの実行パスを追跡**

```
Trace: user_request_123
├── Span: parse_input (2ms)
├── Span: llm_call_1 (1,234ms)
│   ├── prompt: "ユーザーの質問を分析して..."
│   ├── completion: "この質問は製品検索に関するものです..."
│   └── tokens: {input: 150, output: 50}
├── Span: tool_call:search_products (456ms)
│   ├── args: {query: "青いTシャツ", limit: 10}
│   └── result: [{id: 1, name: "..."}, ...]
├── Span: llm_call_2 (987ms)
│   ├── prompt: "検索結果を元に回答を生成..."
│   └── completion: "以下の製品がおすすめです..."
└── Span: format_response (5ms)
```

### 2. Metrics（メトリクス）

**数値で測定可能な指標**

```python
metrics = {
    # パフォーマンス
    "latency_p50": 1.2,  # 秒
    "latency_p99": 5.8,
    "throughput": 100,    # req/min

    # コスト
    "tokens_per_request": 2500,
    "cost_per_request": 0.05,  # ドル

    # 品質
    "success_rate": 0.95,
    "hallucination_rate": 0.02,
    "user_satisfaction": 4.2,  # 5点満点

    # エラー
    "error_rate": 0.03,
    "timeout_rate": 0.01,
}
```

### 3. Evaluation（評価）

**出力の品質を定量的に評価**

```python
evaluation_dimensions = {
    "relevance": "回答は質問に対して適切か",
    "faithfulness": "回答はコンテキストに忠実か（幻覚なし）",
    "coherence": "回答は論理的で一貫しているか",
    "helpfulness": "回答はユーザーにとって有用か",
}
```

---

## トレーシングの実装

### OpenTelemetryベースの設計

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter

# セットアップ
trace.set_tracer_provider(TracerProvider())
tracer = trace.get_tracer("ai-agent")

# エクスポーター設定（Langfuse, Datadog, Grafana等に送信可能）
exporter = OTLPSpanExporter(endpoint="http://collector:4317")
trace.get_tracer_provider().add_span_processor(
    BatchSpanProcessor(exporter)
)
```

### エージェント用トレーシングデコレータ

```python
import functools
from opentelemetry import trace

tracer = trace.get_tracer("ai-agent")

def trace_llm_call(func):
    """LLM呼び出しをトレース"""
    @functools.wraps(func)
    async def wrapper(*args, **kwargs):
        with tracer.start_as_current_span("llm_call") as span:
            # 入力を記録
            span.set_attribute("llm.model", kwargs.get("model", "unknown"))
            span.set_attribute("llm.prompt", str(kwargs.get("prompt", ""))[:1000])

            try:
                result = await func(*args, **kwargs)

                # 出力を記録
                span.set_attribute("llm.completion", str(result)[:1000])
                span.set_attribute("llm.tokens.input", result.usage.input_tokens)
                span.set_attribute("llm.tokens.output", result.usage.output_tokens)
                span.set_status(trace.Status(trace.StatusCode.OK))

                return result

            except Exception as e:
                span.set_status(trace.Status(trace.StatusCode.ERROR, str(e)))
                span.record_exception(e)
                raise

    return wrapper

def trace_tool_call(func):
    """ツール呼び出しをトレース"""
    @functools.wraps(func)
    async def wrapper(*args, **kwargs):
        with tracer.start_as_current_span(f"tool:{func.__name__}") as span:
            span.set_attribute("tool.name", func.__name__)
            span.set_attribute("tool.args", str(kwargs)[:500])

            try:
                result = await func(*args, **kwargs)
                span.set_attribute("tool.result", str(result)[:500])
                span.set_status(trace.Status(trace.StatusCode.OK))
                return result

            except Exception as e:
                span.set_status(trace.Status(trace.StatusCode.ERROR, str(e)))
                raise

    return wrapper
```

### 使用例

```python
class Agent:
    @trace_llm_call
    async def think(self, prompt):
        return await self.llm.generate(prompt=prompt)

    @trace_tool_call
    async def search_database(self, query):
        return await self.db.search(query)

    async def run(self, user_input):
        with tracer.start_as_current_span("agent_run") as span:
            span.set_attribute("user.input", user_input)

            # 思考
            thought = await self.think(f"ユーザー入力: {user_input}")

            # ツール実行
            results = await self.search_database(thought.query)

            # 回答生成
            response = await self.think(f"結果: {results}")

            span.set_attribute("agent.response", str(response)[:500])
            return response
```

---

## 評価システムの実装

### LLM-as-Judge

人間のスケールが難しい場合、LLMで評価：

```python
class LLMJudge:
    def __init__(self, evaluator_llm):
        self.llm = evaluator_llm

    async def evaluate_relevance(self, question, answer, context=None):
        """回答の関連性を評価 (1-5)"""
        prompt = f"""
        以下の質問と回答のペアを評価してください。

        質問: {question}
        回答: {answer}
        {"コンテキスト: " + context if context else ""}

        回答は質問に対して適切ですか？
        1-5のスコアで評価し、理由も述べてください。

        出力形式:
        スコア: [1-5]
        理由: [説明]
        """

        result = await self.llm.generate(prompt)
        return self._parse_score(result)

    async def evaluate_faithfulness(self, answer, context):
        """幻覚がないか評価"""
        prompt = f"""
        以下の回答が、与えられたコンテキストの情報のみに基づいているか評価してください。

        コンテキスト: {context}
        回答: {answer}

        回答はコンテキストの情報のみに基づいていますか？
        コンテキストにない情報が含まれていれば「幻覚あり」と判定してください。

        出力形式:
        判定: [幻覚なし / 幻覚あり]
        該当箇所: [幻覚がある場合、その部分を引用]
        """

        result = await self.llm.generate(prompt)
        return self._parse_faithfulness(result)

    async def evaluate_all(self, question, answer, context=None):
        """複数の観点で一括評価"""
        results = await asyncio.gather(
            self.evaluate_relevance(question, answer, context),
            self.evaluate_faithfulness(answer, context) if context else None,
            self.evaluate_coherence(answer),
            self.evaluate_helpfulness(question, answer),
        )

        return {
            "relevance": results[0],
            "faithfulness": results[1],
            "coherence": results[2],
            "helpfulness": results[3],
        }
```

### 人間による評価との組み合わせ

```python
class HybridEvaluator:
    def __init__(self, llm_judge, human_review_queue):
        self.llm = llm_judge
        self.human_queue = human_review_queue

    async def evaluate(self, interaction):
        # 1. まずLLMで自動評価
        llm_scores = await self.llm.evaluate_all(
            interaction.question,
            interaction.answer,
            interaction.context,
        )

        # 2. 低スコアまたは重要なケースは人間レビューへ
        needs_human_review = (
            llm_scores["relevance"] < 3 or
            llm_scores["faithfulness"] == "幻覚あり" or
            interaction.is_high_stakes
        )

        if needs_human_review:
            await self.human_queue.add(interaction, llm_scores)

        return {
            "llm_scores": llm_scores,
            "needs_human_review": needs_human_review,
        }
```

---

## 異常検出とアラート

### リアルタイム監視

```python
class AgentMonitor:
    def __init__(self, alert_thresholds):
        self.thresholds = alert_thresholds
        self.metrics = MetricsCollector()

    async def check_health(self):
        """定期的にヘルスチェック"""
        current_metrics = await self.metrics.get_current()

        alerts = []

        # レイテンシ異常
        if current_metrics["latency_p99"] > self.thresholds["latency_p99"]:
            alerts.append(Alert(
                severity="warning",
                message=f"P99 latency {current_metrics['latency_p99']}s exceeds threshold",
            ))

        # エラー率異常
        if current_metrics["error_rate"] > self.thresholds["error_rate"]:
            alerts.append(Alert(
                severity="critical",
                message=f"Error rate {current_metrics['error_rate']:.2%} exceeds threshold",
            ))

        # 幻覚率異常
        if current_metrics["hallucination_rate"] > self.thresholds["hallucination_rate"]:
            alerts.append(Alert(
                severity="critical",
                message=f"Hallucination rate {current_metrics['hallucination_rate']:.2%} is high",
            ))

        # コスト異常
        if current_metrics["cost_per_hour"] > self.thresholds["cost_per_hour"]:
            alerts.append(Alert(
                severity="warning",
                message=f"Hourly cost ${current_metrics['cost_per_hour']:.2f} exceeds budget",
            ))

        return alerts
```

### ドリフト検出

```python
class DriftDetector:
    """出力品質のドリフトを検出"""

    def __init__(self, baseline_metrics, window_size=1000):
        self.baseline = baseline_metrics
        self.window = deque(maxlen=window_size)

    def record(self, evaluation_result):
        self.window.append(evaluation_result)

    def detect_drift(self):
        if len(self.window) < 100:
            return None  # データ不足

        current_metrics = self._calculate_current_metrics()

        drifts = []
        for metric_name, baseline_value in self.baseline.items():
            current_value = current_metrics.get(metric_name)

            if current_value is None:
                continue

            # 統計的有意差を検定
            drift_score = self._calculate_drift_score(
                baseline_value,
                current_value,
            )

            if drift_score > 0.05:  # 5%以上の変化
                drifts.append({
                    "metric": metric_name,
                    "baseline": baseline_value,
                    "current": current_value,
                    "drift_score": drift_score,
                })

        return drifts if drifts else None
```

---

## 2026年のObservabilityプラットフォーム

### 主要プラットフォーム比較

| プラットフォーム | 特徴 | 価格帯 |
|----------------|------|--------|
| **Langfuse** | オープンソース、セルフホスト可能 | 無料〜 |
| **LangSmith** | LangChain統合、使いやすい | $39/月〜 |
| **Braintrust** | 自動評価、コスト分析 | Enterprise |
| **Arize Phoenix** | オープンソース、ドリフト検出 | 無料〜 |
| **Datadog LLM** | 既存のDatadog統合 | 従量課金 |

### Langfuseの実装例

```python
from langfuse import Langfuse
from langfuse.decorators import observe

# 初期化
langfuse = Langfuse()

class ObservedAgent:
    @observe(name="agent_run")
    async def run(self, user_input):
        # トレースは自動で記録される

        thought = await self.think(user_input)
        results = await self.search(thought.query)
        response = await self.respond(results)

        # スコアを記録
        langfuse.score(
            trace_id=langfuse.get_current_trace_id(),
            name="user_feedback",
            value=await self.get_user_feedback(),
        )

        return response

    @observe(name="llm_call", capture_input=True, capture_output=True)
    async def think(self, prompt):
        return await self.llm.generate(prompt)

    @observe(name="tool_call:search")
    async def search(self, query):
        return await self.db.search(query)
```

---

## CI/CDへの統合

### 評価テストの自動化

```python
# tests/test_agent_quality.py
import pytest
from agent import Agent
from evaluator import LLMJudge

@pytest.fixture
def agent():
    return Agent()

@pytest.fixture
def judge():
    return LLMJudge()

@pytest.mark.asyncio
async def test_relevance_threshold(agent, judge):
    """回答の関連性が閾値を超えることを確認"""
    test_cases = [
        {"question": "Pythonでリストを逆順にする方法は？", "expected_topic": "Python"},
        {"question": "機械学習のオーバーフィッティングとは？", "expected_topic": "ML"},
    ]

    for case in test_cases:
        response = await agent.run(case["question"])
        score = await judge.evaluate_relevance(case["question"], response)

        assert score >= 4, f"Relevance score {score} is below threshold for: {case['question']}"

@pytest.mark.asyncio
async def test_no_hallucination(agent, judge):
    """幻覚がないことを確認"""
    # 既知のコンテキストで質問
    context = "Pythonは1991年にGuido van Rossumによって作られました。"
    question = "Pythonはいつ作られましたか？"

    response = await agent.run(question, context=context)
    faithfulness = await judge.evaluate_faithfulness(response, context)

    assert faithfulness["judgment"] == "幻覚なし", f"Hallucination detected: {faithfulness}"
```

### GitHub Actions統合

```yaml
# .github/workflows/agent-quality.yml
name: Agent Quality Check

on:
  pull_request:
    branches: [main]

jobs:
  evaluate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run evaluation tests
        run: |
          pytest tests/test_agent_quality.py -v

      - name: Compare with baseline
        run: |
          python scripts/compare_baseline.py \
            --current-results results.json \
            --baseline baseline.json \
            --threshold 0.05

      - name: Upload results to Langfuse
        run: |
          python scripts/upload_evaluation.py \
            --results results.json \
            --commit ${{ github.sha }}
```

---

## ベストプラクティス

### 1. 全てのLLM呼び出しをトレース

```python
# ✓ 良い例
@observe()
async def llm_call(prompt):
    return await llm.generate(prompt)

# ✗ 悪い例
async def llm_call(prompt):
    return await llm.generate(prompt)  # トレースなし
```

### 2. 構造化ログを使用

```python
# ✓ 良い例
logger.info("tool_called", extra={
    "tool_name": "search",
    "args": {"query": query},
    "duration_ms": 150,
    "result_count": len(results),
})

# ✗ 悪い例
logger.info(f"Called search with {query}, got {len(results)} results in 150ms")
```

### 3. コストを常に追跡

```python
class CostTracker:
    PRICING = {
        "gpt-4o": {"input": 0.005, "output": 0.015},  # per 1K tokens
        "claude-3-opus": {"input": 0.015, "output": 0.075},
    }

    def calculate_cost(self, model, input_tokens, output_tokens):
        prices = self.PRICING.get(model, {"input": 0, "output": 0})
        return (
            (input_tokens / 1000) * prices["input"] +
            (output_tokens / 1000) * prices["output"]
        )
```

### 4. サンプリングを活用

```python
# 全リクエストを評価するのはコスト高
class SampledEvaluator:
    def __init__(self, sample_rate=0.1):
        self.sample_rate = sample_rate

    async def maybe_evaluate(self, interaction):
        if random.random() < self.sample_rate:
            return await self.evaluate(interaction)
        return None
```

---

## まとめ

### Observabilityチェックリスト

```markdown
□ トレーシング
  - 全LLM呼び出しをトレース
  - 全ツール呼び出しをトレース
  - 入出力を記録（プライバシー考慮）

□ メトリクス
  - レイテンシ（P50, P90, P99）
  - スループット
  - エラー率
  - コスト

□ 評価
  - LLM-as-Judge設定
  - 人間レビューフロー
  - CI/CD統合

□ アラート
  - 異常検出ルール
  - ドリフト監視
  - コストアラート

□ インフラ
  - OpenTelemetry準拠
  - プラットフォーム選定
  - データ保持ポリシー
```

---

## 参考リンク

- [Langfuse Documentation](https://langfuse.com/docs)
- [OpenTelemetry for AI](https://opentelemetry.io/)
- [State of Agent Engineering (LangChain)](https://www.langchain.com/state-of-agent-engineering)
- [AI Observability Tools Guide (Braintrust)](https://www.braintrust.dev/articles/best-ai-observability-tools-2026)

---

**あなたのエージェントObservability環境はどうなっていますか？工夫していることがあればコメントで教えてください！**
