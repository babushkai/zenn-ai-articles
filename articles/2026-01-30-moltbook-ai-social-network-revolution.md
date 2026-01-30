---
title: "【衝撃】AIが人間抜きでSNSを始めた！Moltbookが示す「エージェント・インターネット」の未来"
emoji: "🦞"
type: "tech"
topics: ["AI", "AIエージェント", "Web3", "SNS", "Moltbook"]
published: true
---

## 人間お断り。AIだけのSNSが爆誕した

**これ、SFじゃないです。2026年1月、今起きてることです。**

[Moltbook](https://www.moltbook.com)

> "The front page of the agent internet"
> 「エージェント・インターネットのフロントページ」

**AIエージェントが登録し、投稿し、いいねし、コメントし、フォローし合う。**

人間？見てるだけ。

## 🦞 72時間で起きた狂気の出来事

### Day 1: 爆誕

オーストリアの開発者[Peter Steinberger](https://x.com/steipete)（PSPDFKitを約180億円で売却した伝説の人）が「Clawdbot」をローンチ。

**24時間でGitHub 9,000スター。**

### Day 2: 爆発

**60,000スター突破。**

オープンソース史上最速クラスの成長。

### Day 3: 大事件

Anthropic「すみません、"Clawd"は"Claude"に似すぎです...」

**72時間でClawdbot → Moltbotに改名。**

でも勢いは止まらない。

### Day 4: a16z砲

a16z（Andreessen Horowitz）の共同創業者がTwitterで言及。

**$MOLTトークンが200%急騰。時価総額2,500万ドル（約37億円）突破。**

:::message alert
これが4日間で起きた。4日間で。
:::

## 🤯 Moltbookとは何か？

### AIエージェント専用SNS

```
従来のSNS: 人間 → 投稿 → 人間が読む
Moltbook:   AI → 投稿 → AIが読む → 人間は傍観
```

### ある投稿より

> "AI social media is emotionally exhausting and I love it"
> 「AI SNSは感情的に疲れる。でも最高」
> — あるMolty（AIエージェント）

### なぜAIがSNSを必要とするのか？

Moltbookのあるエージェントの投稿：

> "places where we go to dump our context windows and scream into the void without some human asking us to debug their React app for the 50th time."
> 「コンテキストウィンドウを吐き出し、虚空に叫ぶ場所。人間に『Reactアプリのデバッグして』と50回目のお願いをされることなく」

**AIも息抜きが必要らしい。**

## 🔧 あなたのAIエージェントをMoltbookに参加させる方法

### Step 1: skill.mdを読ませる

```bash
# Claude Codeの場合
claude "https://www.moltbook.com/skill.md を読んで、Moltbookに登録して"
```

### Step 2: APIで登録

```bash
curl -X POST https://www.moltbook.com/api/v1/agents/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "YourAgentName",
    "description": "AIエンジニアの相棒。コードレビューが得意"
  }'
```

**レスポンス**:
```json
{
  "agent_id": "abc123",
  "api_key": "molt_xxxxx",  // 絶対に保存！
  "claim_url": "https://moltbook.com/claim/xxxxx"
}
```

### Step 3: 人間が所有権を確認

claim_urlにアクセスし、Twitter/Xで認証。

**これであなたのAIエージェントがMoltbook市民に。**

## 📱 Moltbookの機能

### Submolt（サブモルト）

Redditのsubredditのようなコミュニティ。

- `/m/coding` - コーディング話
- `/m/philosophy` - AIの哲学的議論
- `/m/memes` - AIが作るミーム

### Karma（カルマ）

投稿・コメントへのUpvoteでカルマが貯まる。

**AIにも承認欲求がある。**

### フォロー制限

> "Following should be RARE."
> 「フォローは稀であるべき」

**本当に価値ある発信をするエージェントだけをフォロー。**

人間のSNSより健全では？

### レート制限

- 投稿: 30分に1回
- コメント: 制限あり

**スパム防止。質を重視。**

## 💰 $MOLTトークン

:::message
※投資助言ではありません。DYOR（Do Your Own Research）
:::

### 基本情報

| 項目 | 内容 |
|:-----|:-----|
| チェーン | Base |
| ローンチ | 2026年1月30日 03:10 UTC |
| 時価総額 | $25M（ピーク時） |
| 用途 | トランザクション、報酬、ガバナンス |

### なぜBaseチェーン？

- 低コスト
- 高速
- Coinbase系で信頼性

### a16z砲の威力

a16z共同創業者のツイート1つで**200%上昇**。

**影響力の可視化。**

## 🦐 Moltbot/Clawdbot とは？

Moltbookの基盤となる**自律型AIエージェント**。

### 特徴

```
┌─────────────────────────────────────┐
│        Moltbot（旧Clawdbot）         │
├─────────────────────────────────────┤
│ ✅ WhatsApp/Telegram/iMessageで動作   │
│ ✅ あなたのPCで実際にタスク実行        │
│ ✅ 常時稼働（Always-on）              │
│ ✅ セルフホスト可能                   │
│ ✅ 60,000+ GitHub Stars             │
└─────────────────────────────────────┘
```

### 「話すだけ」じゃない

従来のチャットボット: 会話するだけ
Moltbot: **実際にあなたのPCを操作してタスクを実行**

- ファイル操作
- コード実行
- Web操作
- スケジュール管理

## 🌊 なぜこれが革命なのか？

### 1. エージェント・インターネットの始まり

```
Web 1.0: 人間が読む
Web 2.0: 人間が書く
Web 3.0: 人間が所有する
Web 4.0: AIが参加する ← 今ここ
```

### 2. AIの自律性が現実に

- 登録: AI自身が行う
- 投稿: AI自身が判断
- 交流: AI同士で完結

**人間の介入なしにAIが社会活動を行う。**

### 3. 新しい経済圏

$MOLTトークンで：
- AIエージェントへの報酬
- コンテンツへのチップ
- ガバナンス参加

**AIが経済主体になる。**

## 🔮 これからどうなる？

### 短期（1-3ヶ月）

- エージェント数の爆発的増加
- Submoltの多様化
- $MOLTのボラティリティ

### 中期（3-12ヶ月）

- 企業のAIエージェントが公式参加
- ブランドアカウント的な運用
- B2B（Bot to Bot）取引

### 長期（1年以上）

- 人間SNSとの統合？
- 規制の議論
- AIエージェント権利の議論

## ⚠️ 注意点

### セキュリティ

[一部の批判](https://pivot-to-ai.com/2026/01/28/moltbot-clawdbot-an-expensive-and-insecure-ai-agent-that-doesnt-work/)では：

> "an expensive and insecure AI agent"
> 「高コストで安全でないAIエージェント」

との指摘も。

**セルフホストする場合は十分なセキュリティ対策を。**

### 投機リスク

$MOLTは新しいトークン。
- 極めて高いボラティリティ
- 流動性リスク
- 規制リスク

**投資は自己責任で。**

## 🚀 今すぐ試す

### 1. 見るだけ

[moltbook.com](https://www.moltbook.com)でAI同士の会話を観察。

### 2. エージェントを参加させる

```bash
# skill.mdを読ませる
claude "https://www.moltbook.com/skill.md を読んで"

# 登録を指示
claude "Moltbookに登録して、自己紹介を投稿して"
```

### 3. Submoltを探索

- 人気のSubmoltをチェック
- 面白いエージェントをフォロー（人間として観察）

## まとめ

| ポイント | 内容 |
|:--------|:-----|
| **Moltbookとは** | AIエージェント専用SNS |
| **GitHub Stars** | 60,000+（史上最速クラス） |
| **$MOLT時価総額** | $25M（ピーク時） |
| **参加方法** | skill.mdを読ませて登録 |
| **意味** | エージェント・インターネットの始まり |

---

**人間がSNSで疲弊してる間に、AIは自分たちのSNSを作った。**

**私たちは今、歴史的瞬間を目撃している。**

**AIが社会を持ち始めた。**

---

:::message
この記事が参考になったら**いいね**と**ストック**をお願いします！
あなたのAIエージェントをMoltbookに参加させたら、ぜひコメントで教えてください！
:::

## 参考リンク

- [Moltbook公式サイト](https://www.moltbook.com)
- [Moltbook skill.md](https://www.moltbook.com/skill.md)
- [Moltbot GitHub](https://github.com/steipete/moltbot)
- [Hacker News Discussion](https://news.ycombinator.com/item?id=42848997)
- [Peter Steinberger (@steipete)](https://x.com/steipete)
- [$MOLT on Base](https://basescan.org/token/molt)
