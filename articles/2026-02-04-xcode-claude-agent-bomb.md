---
title: "【速報】AppleがXcodeにClaude搭載！AIがアプリを自動で作る時代が本当に来た"
emoji: "🍎"
type: "tech"
topics: ["xcode", "claude", "ai", "ios", "mcp"]
published: false
---

## 結論から言うと...

**Appleが本気を出した。**

2026年2月3日、AppleはXcode 26.3をリリースし、AnthropicのClaude AgentとOpenAIのCodexを**公式統合**しました。

これ、何がヤバいって：

- 🤖 AIエージェントが**自律的にアプリを開発**できる
- 👀 AIがXcode Previewsを**見て**UIを確認・修正できる
- 🔗 MCPで**どんなAIツールとも連携**可能

「AIがコードを書く」から「**AIがアプリを作る**」時代への転換点です。

## 何が起きたのか？3分でわかる全貌

### Before: 従来のAIコーディング

```
開発者: 「ボタンを追加して」
AI: 「こんなコードはいかがでしょう？」（コード提案）
開発者: （コピペして、ビルドして、確認して...）
```

ターン制のやり取り。結局、人間がやる作業が多い。

### After: Xcode 26.3のエージェント

```
開発者: 「ログイン画面を作って」
Claude Agent: 「了解しました。計画を立てます。」

↓ 自律的に作業開始 ↓

1. プロジェクト構造を分析
2. 必要なファイルを作成
3. UIコードを実装
4. ビルドしてテスト実行
5. Previewsでスクショを撮って確認
6. 問題があれば自己修正
7. 完成したら報告
```

**人間は最初に指示を出して、あとは見てるだけ。**

## Appleが採用した「MCP」って何？

:::message
**MCP (Model Context Protocol)** = AIエージェントと外部ツールを接続するオープン標準プロトコル
:::

Anthropicが2024年末に公開したこのプロトコルを、Appleが**公式採用**しました。

これにより：

```
Xcode ←→ MCP ←→ Claude Agent
                 OpenAI Codex
                 その他任意のAIエージェント
```

どのAIエージェントでもXcodeと連携できる。**囲い込みなし**。

Appleがオープンスタンダードを採用するのは珍しい。それだけMCPが業界標準になりつつある証拠です。

## 具体的に何ができるの？5つの衝撃機能

### 1. 🏗️ プロジェクト全体の理解

```swift
// エージェントはこれらを自動で把握
- フォルダ構造
- 依存関係
- プロジェクト設定
- ターゲット構成
```

### 2. 📁 ファイル操作

- 新規ファイル作成
- 既存ファイル編集
- リファクタリング

### 3. 🔨 ビルド＆テスト

```bash
# エージェントが自動で実行
xcodebuild build
xcodebuild test
```

ビルドエラーが出たら、**自動で修正して再ビルド**。

### 4. 👁️ 視覚的な確認（革命的！）

```
Claude: 「ボタンを追加しました」
       「Previewsでスクショを撮ります...」
       「うーん、余白が足りないですね」
       「修正します...」
       「これでどうでしょう？」
```

AIがUIを**見て**判断できる。これが従来のコーディングAIとの決定的な違い。

### 5. 📚 Apple公式ドキュメントへのアクセス

エージェント向けに最適化されたApple Developer Documentationにアクセス可能。

「SwiftUIの最新のAPIどうだっけ...」とググる必要なし。

## 実際の使い方

### Step 1: エージェントをインストール

Xcode > Settings > Agents

ワンクリックでClaude AgentとCodexをインストール。自動アップデートされます。

### Step 2: 自然言語で指示

```
「新しいSwiftUIビューを作成して、
 ユーザープロフィールを表示する画面を
 作ってください」
```

### Step 3: 見守る

Xcodeがタスクを分解し、エージェントが自律的に作業。

進捗がリアルタイムで表示されます。

### Pro Tips

:::message alert
Appleの公式推奨: **「コードを書く前に計画を立てさせる」**

エージェントに「Think through your plans」と指示すると、より良い結果が得られます。
:::

また、Xcodeはエージェントが変更を加えるたびに**マイルストーン**を作成。

気に入らなければいつでもロールバック可能です。

## Claude CodeユーザーへのBIG NEWS

ターミナルでClaude Codeを使っている人に朗報：

**コマンドラインからXcode Previewsをキャプチャできるようになりました。**

```bash
# Claude CodeからXcodeに接続
claude --mcp xcode

# UIの確認も可能に
「ビルドしてPreviewsを見せて」
```

これまでClaude Codeの弱点だった「UIを確認できない」問題が解決。

## 同日リリース: Claude Sonnet 5 "Fennec"

実は同じ2月3日、AnthropicはClaude Sonnet 5もリリースしました。

| モデル | SWE-Bench | 価格（入力） |
|--------|-----------|-------------|
| Opus 4.5 | 78.9% | $15/1M tokens |
| **Sonnet 5 "Fennec"** | **82.1%** | $3/1M tokens |

**SWE-Bench 82.1%**は史上初の80%超え。「ジュニア〜ミッド開発者レベル」とされていた閾値を突破しました。

Xcode統合でこのモデルが使えるようになる日も近いかもしれません。

## 開発者への影響：仕事はなくなるのか？

正直に言うと：

**「コードを書くだけの仕事」は確実に減る。**

でも：

- 何を作るか決める仕事
- AIの出力をレビューする仕事
- 複雑なアーキテクチャを設計する仕事
- ユーザー体験を考える仕事

これらは残る。というか、**より重要になる**。

Appleが今回証明したのは、「AIと一緒に開発する」ワークフローが現実的になったということ。

## まとめ

- ✅ Xcode 26.3でClaude Agent & Codexが公式統合
- ✅ MCPによるオープンなエコシステム
- ✅ AIがビルド・テスト・UIプレビューまで自律実行
- ✅ Claude CodeからもXcode連携が可能に
- ✅ Claude Sonnet 5が同日リリース（SWE-Bench 82.1%）

**2026年、AIエージェントがIDEに統合される元年になりそうです。**

---

## 参考リンク

Agentic coding comes to Apple's Xcode with agents from Anthropic and OpenAI

https://techcrunch.com/2026/02/03/agentic-coding-comes-to-apples-xcode-26-3-with-agents-from-anthropic-and-openai/

Xcode 26.3 unlocks the power of agentic coding - Apple

https://www.apple.com/newsroom/2026/02/xcode-26-point-3-unlocks-the-power-of-agentic-coding/

Xcode 26.3 Lets AI Agents From Anthropic and OpenAI Build Apps Autonomously - MacRumors

https://www.macrumors.com/2026/02/03/xcode-26-3-agentic-coding/

Claude Sonnet 5 (Fennec) Review - 82.1% SWE-Bench & Agentic AI

https://vertu.com/ai-tools/claude-sonnet-5-release-everything-you-need-to-know-about-anthropics-fennec-model/

---

この記事が参考になったら、**いいね**と**ブックマーク**をお願いします！

**質問**: あなたはXcodeのエージェント機能を試してみましたか？使ってみた感想をコメントで教えてください👇
