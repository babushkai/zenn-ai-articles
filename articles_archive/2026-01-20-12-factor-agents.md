---
title: "【保存版】12-Factor Agents完全解説 - 本番環境で動くAIエージェントの設計原則"
emoji: "🏭"
type: "tech"
topics: ["ai", "llm", "aiagent", "設計", "アーキテクチャ"]
published: false
---

**「AIエージェントを作ったけど、本番で使い物にならない...」**

この悩み、心当たりありませんか？

デモでは動く。チュートリアルは完走できる。でも実際のプロダクトに組み込むと80%の完成度で頭打ち。

その原因、**設計原則の欠如**かもしれません。

## 12-Factor Agents とは？

[12-Factor Agents](https://github.com/humanlayer/12-factor-agents) は、HumanLayerのDexが提唱する**本番環境で動くAIエージェントを構築するための12の設計原則**です。

Herokuが提唱した「[The Twelve-Factor App](https://12factor.net/)」をAIエージェント開発に適用したもの、と考えてください。

:::message
**核心的な主張:** 成功しているAIアプリケーションは「LLM + ツール」ではない。**ほぼ決定論的なソフトウェア**に、**戦略的にLLMを配置**したものだ。
:::

## なぜフレームワークだけでは不十分なのか

LangChainやCrewAIなどのフレームワークは便利です。しかし、本番投入している創業者の多くは**フレームワークを使っていません**。

理由は単純：

> フレームワークは速度と引き換えに柔軟性を犠牲にする。本番環境では**制御**が必要だ。

12-Factor Agentsは「フレームワークを捨てろ」とは言いません。**モジュラーな概念を既存プロダクトに組み込む**ことを推奨しています。

---

## 12の原則 完全解説

### Factor 1: 自然言語からツール呼び出しへ

```python
# ユーザーの意図を構造化されたツール呼び出しに変換
user_input = "明日の東京の天気を教えて"

# LLMが解釈して構造化
tool_call = {
    "name": "get_weather",
    "arguments": {
        "location": "Tokyo",
        "date": "2026-01-21"
    }
}
```

これがエージェントの**核心的な意思決定メカニズム**です。ユーザーの曖昧な要求を、実行可能な関数呼び出しに変換する。

---

### Factor 2: プロンプトを所有せよ

**❌ フレームワーク任せ**
```python
agent = SomeFramework.create_agent(tools=tools)
# 内部で何が起きてるか分からない
```

**✅ 自分で制御**
```python
system_prompt = """
あなたは予約管理アシスタントです。
以下のルールに従ってください：
1. 予約は営業時間内（9:00-18:00）のみ
2. 同じ時間帯に重複予約は不可
3. 不明な場合は必ず確認を取る
"""
```

フレームワークのデフォルトプロンプトに依存しない。**自分でプロンプトを書き、制御する**。

---

### Factor 3: コンテキストウィンドウを所有せよ

これが**Context Engineering**（コンテキストエンジニアリング）です。

LLMが「何を見るか」を積極的に管理する。

```python
def build_context(user_query, conversation_history, relevant_docs):
    # 関連性の高い情報だけを選択
    recent_history = conversation_history[-5:]  # 直近5件のみ
    top_docs = rank_by_relevance(relevant_docs, user_query)[:3]

    return {
        "query": user_query,
        "history": recent_history,
        "context": top_docs
    }
```

:::message alert
**やりがちなミス:** 会話履歴を全部突っ込む → トークン消費が爆発 → コスト増大 & 品質低下
:::

---

### Factor 4: ツールは構造化出力である

ツール呼び出しを「特別な機能」と考えない。**構造化された出力を生成している**と捉える。

```python
# ツール呼び出しも結局はこれ
class WeatherQuery(BaseModel):
    location: str
    date: str

# LLMの出力を構造化するだけ
response = llm.generate(
    prompt=prompt,
    response_format=WeatherQuery
)
```

この視点を持つと、実装がシンプルになります。

---

### Factor 5: 実行状態とビジネス状態を統一せよ

**❌ 二重管理**
```python
# エージェント内部の状態
agent_state = {"current_step": 3, "completed": ["A", "B"]}

# ビジネスロジックの状態（別管理）
order.status = "processing"
```

**✅ 統一管理**
```python
# 一つの真実の源
class Order:
    status: str
    agent_context: dict  # エージェントの進捗もここに
```

並列した状態管理システムを避ける。ビジネスオブジェクトとエージェント状態を同期させる。

---

### Factor 6: 起動/一時停止/再開をシンプルなAPIで

エージェントは中断・修正・再開できるべきです。

```python
class Agent:
    def pause(self) -> AgentState:
        """現在の状態をシリアライズして返す"""
        return self.serialize_state()

    def resume(self, state: AgentState):
        """保存された状態から再開"""
        self.restore_state(state)
        return self.continue_execution()
```

長時間実行されるエージェントや、人間の承認が必要なワークフローで必須です。

---

### Factor 7: 人間への連絡もツール呼び出しで

エスカレーション（人間への引き継ぎ）も、ツール呼び出しと同じ仕組みで実装する。

```python
tools = [
    get_weather,
    search_database,
    contact_human,  # 人間に連絡するのもツールの一つ
]

# LLMが判断して人間に聞く
{
    "name": "contact_human",
    "arguments": {
        "question": "この注文は特別対応が必要ですが、承認してよいですか？",
        "urgency": "high"
    }
}
```

**Human-in-the-loop**を自然に実装できます。

---

### Factor 8: 制御フローを所有せよ

**❌ LLMにループを任せる**
```python
while not done:
    action = llm.decide_next_action()
    result = execute(action)
    # LLMが終了を判断
```

**✅ 決定論的なコードで制御**
```python
def process_order(order):
    # Step 1: 在庫確認（決定論的）
    if not check_inventory(order.items):
        return notify_out_of_stock(order)

    # Step 2: 価格計算（決定論的）
    total = calculate_total(order.items)

    # Step 3: 顧客への確認（ここだけLLM）
    confirmation = llm.generate_confirmation_message(order, total)

    # Step 4: 決済処理（決定論的）
    return process_payment(order, total)
```

LLMのループに任せるのではなく、**明示的にワークフローを管理**する。

---

### Factor 9: エラーをコンテキストに圧縮せよ

失敗時、冗長なスタックトレースをLLMに渡さない。

**❌ 冗長**
```
Traceback (most recent call last):
  File "/app/main.py", line 234, in process
    result = api.call(params)
  File "/app/api.py", line 89, in call
    response = requests.post(url, json=data)
  ... (100行続く)
requests.exceptions.HTTPError: 429 Too Many Requests
```

**✅ 圧縮**
```
API呼び出し失敗: レート制限（429）。30秒後に再試行してください。
```

LLMが**次のアクションを決定できる情報**だけを渡す。

---

### Factor 10: 小さく、焦点を絞ったエージェント

**❌ なんでもできる汎用エージェント**
```python
general_agent = Agent(
    tools=[search, email, calendar, code, translate, analyze, ...]
)
```

**✅ 専門特化したエージェント群**
```python
search_agent = Agent(tools=[web_search, doc_search])
email_agent = Agent(tools=[read_email, send_email])
scheduler_agent = Agent(tools=[check_calendar, book_meeting])

# オーケストレーターが振り分け
orchestrator.route(task, agents=[search_agent, email_agent, scheduler_agent])
```

**専門家チーム > 万能選手**

---

### Factor 11: どこからでもトリガー

エージェントはチャットUIだけから起動されるわけではありません。

```python
# Webhook経由
@app.post("/webhook/stripe")
def handle_stripe_webhook(event):
    return payment_agent.process(event)

# スケジュール実行
@scheduler.task("0 9 * * *")
def daily_report():
    return report_agent.generate_daily_summary()

# API経由
@app.post("/api/analyze")
def analyze_data(request):
    return analysis_agent.run(request.data)
```

複数のエントリーポイントをサポートする設計にする。

---

### Factor 12: エージェントをステートレスReducerに

関数型プログラミングの考え方を適用します。

```python
def agent_reducer(state: AgentState, event: Event) -> AgentState:
    """
    純粋関数として設計
    入力: 現在の状態 + イベント
    出力: 新しい状態
    """
    match event.type:
        case "USER_MESSAGE":
            return handle_user_message(state, event)
        case "TOOL_RESULT":
            return handle_tool_result(state, event)
        case "ERROR":
            return handle_error(state, event)
```

**メリット:**
- テストが容易
- リプレイ可能
- 分散環境で動作

---

## まとめ：12の原則早見表

| # | 原則 | 一言で |
|---|------|--------|
| 1 | 自然言語→ツール呼び出し | 意図を構造化 |
| 2 | プロンプトを所有 | フレームワーク任せにしない |
| 3 | コンテキストを所有 | 何を見せるか制御 |
| 4 | ツール=構造化出力 | シンプルに考える |
| 5 | 状態を統一 | 二重管理を避ける |
| 6 | 起動/停止/再開 | 運用を考慮 |
| 7 | 人間連絡もツール | Human-in-the-loop |
| 8 | 制御フローを所有 | LLMループに頼らない |
| 9 | エラーを圧縮 | 次のアクションに必要な情報だけ |
| 10 | 小さく焦点を絞る | 専門家チーム |
| 11 | どこからでもトリガー | チャットだけじゃない |
| 12 | ステートレスReducer | 関数型で設計 |

---

## 最後に

:::message
**重要なマインドセット:** AIエージェントは「魔法」ではない。**エンジニアリング**である。
:::

LLMがどれだけ進化しても、これらの設計原則は変わりません。観測可能性、信頼性、保守性—これらは能力ではなく**品質**の問題だからです。

フレームワークを全否定する必要はありません。でも、**これらの原則を理解した上で**フレームワークを使うのと、何も考えずに使うのでは、結果が全く違います。

---

## 参考リンク

- [12-Factor Agents (GitHub)](https://github.com/humanlayer/12-factor-agents)
- [The Twelve-Factor App (原典)](https://12factor.net/)
- [HumanLayer](https://humanlayer.dev/)

---

**皆さんのAIエージェント開発で困っていることはありますか？コメントで教えてください！**

**この記事が参考になったら、いいねと保存をお願いします！**
