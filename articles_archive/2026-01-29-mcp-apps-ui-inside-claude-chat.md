---
title: "【速報】ClaudeがアプリのUIを表示できるように！MCP Apps完全解説"
emoji: "🖼️"
type: "tech"
topics: ["mcp", "claude", "ai", "claudecode", "aiagent"]
published: true
---

**「え、ChatでFigmaが動く？Slackも？Asanaも？」**

2026年1月26日、Anthropicが発表した**MCP Apps**が衝撃を与えています。

これまでAIは「テキストを返す」だけでした。
でも今、**AIの中でアプリのUIが動く**時代が始まりました。

## 何が起きたのか

[The Register](https://www.theregister.com/2026/01/26/claude_mcp_apps_arrives/)が報じた衝撃のニュース：

> **「Claude supports MCP Apps, presents UI within chat window」**
> （ClaudeがMCP Appsをサポート、チャットウィンドウ内でUIを表示）

つまり：

```
従来のAI:
User: 「プロジェクトのタイムライン作って」
AI: 「こんな感じでいかがでしょうか...」（テキストで説明）

MCP Apps:
User: 「プロジェクトのタイムライン作って」
AI: [Asanaのタイムラインエディタが表示される]
    「直接編集できますよ」
```

**AIがアプリになった。**

## MCP Appsとは何か

[Model Context Protocol Blog](http://blog.modelcontextprotocol.io/posts/2026-01-26-mcp-apps/)の公式発表：

> **「MCP Apps - Bringing UI Capabilities To MCP Clients」**
> （MCP Apps - MCPクライアントにUI機能を）

### 技術的な仕組み

```
┌─────────────────────────────────────────────────────────┐
│                    従来のMCP                            │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Claude ──[API]──► Figma                               │
│         ◄──[JSON]──                                    │
│                                                         │
│  → テキストでやり取り                                   │
│  → 結果もテキストで返ってくる                           │
│                                                         │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                    MCP Apps                             │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Claude ──[MCP Apps]──► Figma                          │
│         ◄──[UI Component]──                            │
│                                                         │
│  → Figmaのエディタが埋め込まれる                        │
│  → Claude内で直接操作できる                             │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### サポートされるUI要素

[Testing Catalog](https://www.testingcatalog.com/anthropic-integrates-interactive-mcp-apps-into-claude/)によると：

- **チャート**: データ可視化をリアルタイムで
- **フォーム**: 入力フォームを直接表示
- **ダッシュボード**: 複雑なUIも埋め込み可能
- **エディタ**: 専用エディタでの直接編集

## 実際に何ができるのか

### 1. Asanaでプロジェクト管理

```
User: 「来週のスプリントのタスクを整理して」

Claude: [Asanaのタイムラインビューが表示]
        「タスクをドラッグして並び替えられます。
         期限の変更も直接できますよ」
```

### 2. Slackでメッセージ作成

```
User: 「チームに進捗報告を送って」

Claude: [Slackメッセージエディタが表示]
        「書式を整えました。
         送信前に確認・編集できます」
```

### 3. Figmaでデザイン

```
User: 「このワイヤーフレームを修正して」

Claude: [Figmaのキャンバスが表示]
        「直接編集できます。
         変更点をハイライトしておきました」
```

### 4. Boxでファイル管理

```
User: 「プロジェクトのドキュメントを整理して」

Claude: [Boxのファイルマネージャーが表示]
        「フォルダ構造を提案しました。
         ドラッグ&ドロップで調整できます」
```

## 対応プラン

現在、以下のプランで利用可能：

| プラン | MCP Apps |
|--------|----------|
| Free | ❌ |
| Pro | ✅ |
| Max | ✅ |
| Team | ✅ |
| Enterprise | ✅ |

## 他のAIも対応開始

驚くべきことに、MCP Appsは**Claude以外のAI**でも動作します：

[MCP Blog](http://blog.modelcontextprotocol.io/posts/2026-01-26-mcp-apps/)：

> **「ChatGPT、Claude、Goose、VS Codeがこの機能をサポート」**

```
MCP Appsをサポートするクライアント:
├── Claude (Anthropic)
├── ChatGPT (OpenAI)
├── Goose (Block)
└── VS Code (Microsoft)
```

**AIの相互運用性が現実になりつつある。**

## 開発者向け：MCP Appsの作り方

MCP Appsを作成するための基本構造：

```typescript
// MCP App の基本構造
const mcpApp = {
  name: "my-dashboard",
  version: "1.0.0",

  // UI定義
  ui: {
    type: "dashboard",
    components: [
      {
        type: "chart",
        data: dynamicData,
        interactive: true
      },
      {
        type: "form",
        fields: ["name", "email", "message"],
        onSubmit: handleSubmit
      }
    ]
  },

  // MCPツールとの連携
  tools: {
    getData: async () => { /* ... */ },
    saveData: async (data) => { /* ... */ }
  }
};
```

## MCP Tool Searchとの相乗効果

[Geeky Gadgets](https://www.geeky-gadgets.com/claude-search-picked-plugin-tools/)が報じた別の神アップデート：

> **「Claude Code MCP Upgrade: Cut Tokens by 95% with Smart Loading」**

### 従来の問題

```
167ツール × 平均360トークン/ツール = 約60,000トークン
200,000トークンのコンテキストの30%が消費される
```

### MCP Tool Searchの解決策

```python
# 従来: 全ツールを事前ロード
tools = load_all_tools()  # 60,000 tokens

# MCP Tool Search: 必要な時だけロード
def get_tool(query):
    # 関連ツールのみを検索・ロード
    return search_and_load(query)  # ~3,000 tokens
```

**95%のトークン削減！**

## これが意味すること

### 1. AIがOSになる

```
従来:
  Browser → Figma
  Browser → Slack
  Browser → Asana
  （アプリを切り替える）

MCP Apps:
  Claude → Figma UI
        → Slack UI
        → Asana UI
  （AIの中で全て完結）
```

### 2. コンテキストスイッチの消滅

```
従来の作業フロー:
1. Claudeにアイデアを相談
2. Figmaを開いてデザイン
3. Claudeに戻ってフィードバック
4. Figmaに戻って修正
（4回のコンテキストスイッチ）

MCP Apps:
1. Claudeにアイデアを相談
2. Claude内でFigmaを操作
3. その場でフィードバック反映
（0回のコンテキストスイッチ）
```

### 3. AIネイティブアプリの誕生

```
2020年代前半: AIは「ツール」
2020年代後半: AIは「アシスタント」
2026年〜: AIは「ワークスペース」
```

## 実装のベストプラクティス

### DO（やるべきこと）

1. **インタラクティブなUI**を設計する
2. **リアルタイム更新**を実装する
3. **エラーハンドリング**を丁寧に
4. **アクセシビリティ**を考慮する

### DON'T（やってはいけないこと）

1. **重すぎるUI**は避ける（チャット内で動くことを忘れずに）
2. **認証フロー**をUI内に入れすぎない
3. **複雑すぎる操作**は専用アプリに任せる

## GPT-5.2との比較

同時期に発表された[GPT-5.2](https://openai.com/index/introducing-gpt-5-2/)も注目：

| 機能 | Claude + MCP Apps | GPT-5.2 |
|------|-------------------|---------|
| UI表示 | ✅ MCP Apps | ❌ |
| コード実行 | ✅ Claude Code | ✅ Code Interpreter |
| 推論能力 | ✅ Extended Thinking | ✅ 90%+ ARC-AGI |
| ツール連携 | ✅ MCP | ✅ Function Calling |

**ClaudeはUI統合、GPT-5.2は推論強化という別路線。**

## 今後の展望

### 予測1: MCP Apps Marketplace

```
2026年後半:
├── 公式MCP Apps Store
├── サードパーティアプリの爆発的増加
├── 企業向けカスタムアプリ
└── 収益化モデルの確立
```

### 予測2: AIファーストなアプリ設計

```
従来: Web/Mobile → AI対応を追加
今後: AI Native → Web/Mobileは副次的
```

### 予測3: ワークフロー革命

```
「Claudeの中で仕事が完結する」世界:
- デザイン
- コーディング
- プロジェクト管理
- コミュニケーション
- ドキュメント作成
- データ分析

→ 全てがAIの中で動く
```

## まとめ

### MCP Appsのポイント

- **UI埋め込み**: チャート、フォーム、ダッシュボードがClaude内で動作
- **アプリ操作**: Figma、Slack、Asana、Boxが直接操作可能
- **マルチAI対応**: ChatGPT、VS Codeなど他のクライアントも対応
- **95%トークン削減**: MCP Tool Searchとの相乗効果

### これからの働き方

```
Before: 複数アプリを切り替えながら作業
After:  AIの中で全て完結
```

**AIは「相談相手」から「ワークスペース」になった。**

**あなたはMCP Appsを試しましたか？どのアプリを使ってみたいですか？コメントで教えてください！**

---

## 参考文献

- [Claude supports MCP Apps (The Register)](https://www.theregister.com/2026/01/26/claude_mcp_apps_arrives/)
- [MCP Apps - Bringing UI Capabilities (MCP Blog)](http://blog.modelcontextprotocol.io/posts/2026-01-26-mcp-apps/)
- [Claude Code MCP Upgrade (Geeky Gadgets)](https://www.geeky-gadgets.com/claude-search-picked-plugin-tools/)
- [Anthropic integrates MCP apps (Testing Catalog)](https://www.testingcatalog.com/anthropic-integrates-interactive-mcp-apps-into-claude/)
- [Claude expands tool connections (Help Net Security)](https://www.helpnetsecurity.com/2026/01/27/anthropic-claude-mcp-integration/)
- [Introducing GPT-5.2 (OpenAI)](https://openai.com/index/introducing-gpt-5-2/)

---

この記事が参考になったら、**いいね**と**ストック**をお願いします！
