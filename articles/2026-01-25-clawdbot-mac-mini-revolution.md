---
title: "【話題沸騰】Mac Mini + Clawdbotで「自分専用のJarvis」を構築する完全ガイド｜180M tokens消費した結果"
emoji: "🦞"
type: "tech"
topics: ["ai", "macos", "claudeai", "自動化", "パーソナルai"]
published: false
---

**「1週間で1億8000万トークン消費した。でも後悔はしていない。」**

MacStoriesのFederico Viticciがそう語った**Clawdbot**が、今AIコミュニティで大炎上しています。

> "Clawdbotは、2026年におけるインテリジェントでパーソナルなAIアシスタントの意味を根本的に変えた"
> — [MacStories](https://www.macstories.net/stories/clawdbot-showed-me-what-the-future-of-personal-ai-assistants-looks-like/)

結論から言うと、**これは「自分だけのJarvis」を現実にする最も簡単な方法**です。

## 🦞 Clawdbotとは何か？

https://github.com/clawdbot/clawdbot

**GitHub Stars: 8,000+** ⭐

Clawdbotは、**ローカルで動作するオープンソースのパーソナルAIアシスタント**です。

特徴:
- 🏠 **ローカルファースト**: あなたのMac Mini/PC上で動作
- 💬 **マルチチャネル**: WhatsApp, Telegram, Slack, Discord, iMessage...
- 🎤 **音声対応**: 常時リスニング、Voice Wake
- 🔧 **ツール実行**: ファイル操作、ブラウザ制御、システムコマンド
- 🔒 **プライバシー**: データは全てローカル

```
┌─────────────────────────────────────────────────────┐
│                   Your Mac Mini                      │
│  ┌───────────────────────────────────────────────┐  │
│  │              Clawdbot Gateway                  │  │
│  │         ws://127.0.0.1:18789                  │  │
│  └───────────────────────────────────────────────┘  │
│         │           │           │           │       │
│         ▼           ▼           ▼           ▼       │
│    ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐    │
│    │WhatsApp│ │Telegram│ │ Slack  │ │iMessage│    │
│    └────────┘ └────────┘ └────────┘ └────────┘    │
└─────────────────────────────────────────────────────┘
                         │
                         ▼
              ┌─────────────────────┐
              │   Claude / GPT API   │
              └─────────────────────┘
```

## 🔥 なぜ今バズっているのか？

### MacStoriesの衝撃レポート

Federico Viticciは、M4 Mac Miniに**Clawdbot**をインストールし、「Navi」と名付けたAIアシスタントを1週間使い倒しました。

**結果:**
- 💰 1億8000万トークン消費
- 🤯 「2026年のパーソナルAIの未来を見た」
- 📱 WhatsApp経由で24時間アクセス可能

> "Clawdbotは、カスタマイズ可能で適応的な新世代の『柔軟なソフトウェア』の究極の表現だ"
> — Federico Viticci

### Hacker Newsでの議論

> "AIはあなたの仕事を奪わない。$599のMac MiniとClaudeを持った人があなたの仕事を奪う"
> — [Hacker News](https://news.ycombinator.com/item?id=46750927)

**なぜMac Miniなのか？**
- 💵 $599〜という低価格
- ⚡ M4チップの高性能
- 🔌 24時間稼働に最適（低消費電力）
- 🏠 自宅サーバーとして理想的

## 📦 セットアップ完全ガイド

### 必要なもの

| 項目 | 推奨 | 最低 |
|------|------|------|
| Mac | M4 Mac Mini | M1以降 |
| Node.js | v22+ | v22+ |
| API | Claude Pro/Max | Claude Pro |
| ストレージ | 256GB+ | 128GB |

### Step 1: インストール

```bash
# Clawdbotをグローバルインストール
npm install -g clawdbot@latest

# オンボーディング（デーモンも自動設定）
clawdbot onboard --install-daemon
```

:::message
**ポイント**: `--install-daemon`でmacOSのlaunchdに自動登録。Mac再起動後も自動起動します。
:::

### Step 2: 初期設定

オンボーディングウィザードが起動:

1. **Gateway設定** - ローカルWebSocket設定
2. **Workspace作成** - AIの作業空間
3. **チャネル接続** - WhatsApp, Telegram等
4. **Skills設定** - 追加機能

### Step 3: チャネル接続（WhatsApp例）

```bash
# WhatsAppを追加
clawdbot channel add whatsapp

# QRコードをスキャン
# → スマホのWhatsAppでスキャン
```

これで、**WhatsAppからあなた専用のAIにメッセージを送れるようになります。**

### Step 4: 動作確認

WhatsAppで自分に送信:
```
こんにちは、今日の予定を教えて
```

**Clawdbotが応答！** 🎉

## ⚙️ 設定ファイル詳解

```json
// ~/.clawdbot/clawdbot.json
{
  "agent": {
    "model": "anthropic/claude-opus-4-5",
    "personality": "あなたは私のパーソナルアシスタント「ナビ」です。丁寧だが親しみやすく、技術的な質問には詳細に答えます。"
  },
  "channels": {
    "whatsapp": {
      "enabled": true,
      "dmPairing": true
    },
    "telegram": {
      "enabled": true,
      "botToken": "YOUR_BOT_TOKEN"
    }
  },
  "tools": {
    "browser": { "enabled": true },
    "filesystem": { "enabled": true },
    "system": { "enabled": true }
  },
  "voice": {
    "enabled": true,
    "provider": "elevenlabs",
    "wakeWord": "ヘイナビ"
  }
}
```

## 🛠️ できること一覧

### 1. マルチチャネル対応

| チャネル | 対応 | 備考 |
|---------|------|------|
| WhatsApp | ✅ | Baileys経由 |
| Telegram | ✅ | Bot API |
| Slack | ✅ | Bolt SDK |
| Discord | ✅ | discord.js |
| iMessage | ✅ | macOSのみ |
| Signal | ✅ | signal-cli |
| Teams | ✅ | Microsoft Graph |
| Matrix | ✅ | オープンソース好きに |

### 2. ローカルツール実行

```typescript
// ブラウザ制御
「Googleで"Mac Mini M4 レビュー"を検索して、上位3つの記事をまとめて」

// ファイル操作
「デスクトップのスクリーンショットを整理して、日付別フォルダに分けて」

// システムコマンド
「今動いてるDockerコンテナを教えて」

// 通知
「15分後にミーティングをリマインドして」
```

### 3. 音声インターフェース

```
🎤 "ヘイナビ、今日の天気は？"
🔊 "東京は晴れ、最高気温12度です"
```

**Voice Wake対応**: 常時リスニングでハンズフリー

### 4. マルチエージェント

```json
// 異なるチャネルに異なるペルソナ
{
  "workspaces": {
    "personal": {
      "channels": ["whatsapp", "imessage"],
      "personality": "親しみやすいアシスタント"
    },
    "work": {
      "channels": ["slack", "teams"],
      "personality": "プロフェッショナルなビジネスアシスタント"
    }
  }
}
```

## 🔒 セキュリティ設計

### デフォルトで安全

```bash
# セキュリティ診断
clawdbot doctor

# 出力例:
# ✅ DM pairing mode: enabled
# ✅ Sandbox mode: docker
# ✅ Tool permissions: restricted
# ⚠️ Elevated bash: disabled (推奨)
```

### DMペアリング

知らない人からのDMはブロック。ペアリングコードで認証:

```
[Unknown sender]
> こんにちは

[Clawdbot]
> ペアリングコードを入力してください: ****
```

### Dockerサンドボックス

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "docker"  // 非mainセッションはDocker内で実行
      }
    }
  }
}
```

## 📊 コスト試算

### Federico Viticciの実績

| 期間 | トークン消費 | 推定コスト |
|------|-------------|-----------|
| 1週間 | 180M tokens | ~$270 (Claude Pro) |

### 一般的な使用パターン

| 使い方 | 月間トークン | 月額コスト |
|-------|------------|-----------|
| ライト（1日10メッセージ） | ~5M | ~$7.50 |
| ミディアム（1日50メッセージ） | ~25M | ~$37.50 |
| ヘビー（常時使用） | ~100M+ | ~$150+ |

:::message alert
**注意**: Claude Proの月額$20には使用制限があります。ヘビーユーザーはClaude Max（$100/月）またはAPI直接利用を検討。
:::

## 🚀 発展的な使い方

### Tailscale統合（外出先からアクセス）

```bash
# Tailscaleをインストール
brew install tailscale

# Clawdbot用にServeを設定
tailscale serve --bg 18789
```

→ 外出先からも`https://your-tailnet.ts.net`経由でアクセス可能

### Skills（プラグイン）追加

```bash
# ClawdHubからSkillsをインストール
clawdbot skill install home-automation
clawdbot skill install calendar-sync
clawdbot skill install notion-integration
```

### iOS/Androidノード

Clawdbotの**コンパニオンアプリ**をインストールすると:

- 📸 カメラへのアクセス
- 🖼️ スクリーン録画
- 🗣️ Voice Wake
- 📍 位置情報

**スマホを「センサー」としてMac Mini上のAIに接続。**

## 🆚 他のソリューションとの比較

| 機能 | Clawdbot | Claude Desktop | ChatGPT App |
|------|----------|----------------|-------------|
| ローカル実行 | ✅ | ❌ | ❌ |
| マルチチャネル | ✅ (10+) | ❌ | ❌ |
| 音声Wake | ✅ | ❌ | ✅ |
| ツール実行 | ✅ (全権限) | 🔶 (制限) | 🔶 (制限) |
| オープンソース | ✅ | ❌ | ❌ |
| カスタマイズ | ∞ | 🔶 | 🔶 |
| プライバシー | ✅ (ローカル) | ❌ | ❌ |

## 💡 私の使い方

### 朝のルーティン

```
🗣️ "ヘイナビ、おはよう"

🤖 "おはようございます。今日の予定です:
    - 10:00 チームミーティング
    - 14:00 クライアントコール
    - 天気: 晴れ、12°C
    - 未読メール: 3件（重要1件）"
```

### 仕事中

```
[Slack → Clawdbot]
「このエラーログ解析して」
[ログファイルを添付]

[Clawdbot]
「分析結果:
1. PostgreSQLの接続タイムアウト
2. 原因: コネクションプール枯渇
3. 推奨: max_connections を 100 → 200 に変更
コマンド: ALTER SYSTEM SET max_connections = 200;」
```

### 帰宅後

```
[WhatsApp → Clawdbot]
「今日のニュースまとめて」

[Clawdbot]
「📰 今日のAIニュース:
1. Anthropic Cowork正式リリース
2. OpenAI GPT-5.2発表
3. Clawdbot、MacStoriesで特集される（あなたも使ってますね！）」
```

## 🔮 これは始まりに過ぎない

2026年、**「AIアシスタント時代」の到来**が現実になりました。

Daniel Kokotajlo（AI Futures Project）が2021年に予測した通り:

> "AIアシスタントの時代は2026年に到来する"

Clawdbotは、その予言を現実にしている最前線のプロジェクトです。

**$599のMac Mini + Clawdbot = あなた専用のJarvis**

試さない理由がありますか？

---

## 🔗 リンク

- **GitHub**: https://github.com/clawdbot/clawdbot
- **公式サイト**: https://clawd.bot/
- **MacStories記事**: https://www.macstories.net/stories/clawdbot-showed-me-what-the-future-of-personal-ai-assistants-looks-like/
- **Techmeme**: https://www.techmeme.com/260124/p17

---

## 🙏 最後に

この記事が参考になったら、**いいねと保存**をお願いします！

**質問**: Clawdbot、試してみましたか？どんな使い方をしていますか？自分専用AIに何と名前をつけましたか？コメントで教えてください！

次回は「**ClawdbotにMCPサーバーを接続して最強のツール環境を構築する**」を書く予定です。お楽しみに！
