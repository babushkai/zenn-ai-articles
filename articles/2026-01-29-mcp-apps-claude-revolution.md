---
title: "【速報】ClaudeがUIを表示できるようになった！MCP Appsが革命的すぎる件"
emoji: "🚀"
type: "tech"
topics: ["claude", "mcp", "ai", "anthropic", "aiagent"]
published: true
---

**「え、チャット画面の中でアプリが動くの？」**

はい、動きます。しかも3日前に発表されたばかりの激アツ機能です。

2026年1月26日、Anthropicが発表した**MCP Apps**により、Claudeのチャットウィンドウ内でサードパーティアプリのUIが直接表示・操作できるようになりました。

これ、地味に見えて**AIの歴史が変わるレベル**の発表です。

## 結論から言うと

:::message
MCP Appsにより、AIは「テキストを返すだけの存在」から「UIを持つアプリケーションプラットフォーム」へ進化しました。
:::

今までのAI：「この表を作りました」→ テキストで返答
これからのAI：「この表を作りました」→ **インタラクティブなダッシュボードが表示される**

## MCP Appsとは何か？

### Before: 従来のMCP

Model Context Protocol (MCP) は、AIが外部ツールやAPIと連携するための「USB-C的な」標準規格として2024年末にAnthropicが発表しました。

```
ユーザー → Claude → MCP Server → 外部API
                  ← テキスト結果 ←
```

データは取れる。でも返ってくるのはテキストだけ。

### After: MCP Apps

```
ユーザー → Claude → MCP Server → 外部API
                  ← インタラクティブUI ←
```

**チャート、フォーム、ダッシュボード、ゲームまで**がチャット内で動く。

## 何がすごいのか？5つのポイント

### 1. もう別タブを開かなくていい

Slackの通知確認→Slackアプリへ
Figmaのデザイン確認→Figmaへ
スプレッドシートの編集→Google Sheetsへ

この「コンテキストスイッチ地獄」から解放されます。

**全部Claudeの画面内で完結**。

### 2. ローンチパートナーがガチ

初日から使えるアプリ：
- **Figma** - デザインレビューがチャット内で
- **Slack** - メッセージ確認・返信がその場で
- **Asana** - タスク管理をAIと一緒に
- **Canva** - 画像編集までチャット内で
- **monday.com** - プロジェクト管理
- **Amplitude** - 分析ダッシュボード
- **Box** - ファイル管理
- **Hex** - データ分析

### 3. OpenAIも追従を発表

MCP Appsは、実は**OpenAIのApps SDKとの共同開発**。ChatGPTも今週中にサポート予定。

これ、競合同士が手を組んだってことです。それくらい**業界標準になる**と確信されている。

### 4. VS Codeでも使える

GitHub Copilotを使っている人朗報。VS Code内でもMCP Apps対応が発表されています。

コーディング中に「このAPIのドキュメント見せて」→ **その場でインタラクティブなドキュメントが表示**

### 5. セキュリティもちゃんと考えられている

「外部コードがチャット内で動くって危なくない？」

Anthropicの回答：
- サンドボックス化されたiframeで実行
- 事前宣言されたテンプレートのみ許可
- すべての通信はJSON-RPCでログ取得可能
- ツール呼び出しには明示的なユーザー同意が必要

## 実際に何ができるのか？

### ユースケース1: データ分析

**Before:**
```
ユーザー: 先月の売上データを分析して
Claude: 以下が分析結果です...（長いテキスト）
```

**After:**
```
ユーザー: 先月の売上データを分析して
Claude: [インタラクティブなダッシュボードが表示]
        ↑ フィルター変更、ドリルダウン、エクスポートがその場で可能
```

### ユースケース2: デザインレビュー

**Before:**
```
ユーザー: このFigmaファイルレビューして
Claude: リンクを開いて確認しました。問題点は...
```

**After:**
```
ユーザー: このFigmaファイルレビューして
Claude: [Figmaのデザインがチャット内に表示]
        ここの余白が狭すぎますね（画面上で直接ハイライト）
```

### ユースケース3: タスク管理

**Before:**
```
ユーザー: 今日のタスクは？
Claude: 以下の5件です: 1. xxx 2. xxx...
```

**After:**
```
ユーザー: 今日のタスクは？
Claude: [Asanaのタスクボードが表示]
        ↑ ドラッグ&ドロップで優先度変更、その場で完了マーク
```

## 開発者視点：MCP Appsの作り方

MCP Appsを作るには、既存のMCPサーバーに**UIリソース**を追加するだけ。

```typescript
// 従来のMCPツール
server.tool("analyze_data", async (params) => {
  const result = await analyzeData(params);
  return { type: "text", text: JSON.stringify(result) };
});

// MCP Apps対応
server.tool("analyze_data", async (params) => {
  const result = await analyzeData(params);
  return {
    type: "ui",
    resource: {
      uri: "app://dashboard",
      html: generateDashboardHTML(result)
    }
  };
});
```

:::message alert
**注意**: HTML内で実行されるJavaScriptには制限があります。外部リソースの読み込み、Cookie/LocalStorageへのアクセスなどは制限されています。
:::

## 2026年、AIエージェントはどうなる？

MCP Appsの発表は、より大きなトレンドの一部です。

### Gartnerの予測
> 2026年末までに、エンタープライズアプリの40%がAIエージェントを使用（2025年は5%未満）

### Databricksの調査
> マルチエージェントワークフローの利用が4ヶ月で327%増加

### 日本市場の動向
- ソフトバンク：配送効率40%向上を達成
- 製造業：在庫管理・予知保全でPoCが活発化
- 2026年は「AIで稼ぐ企業」と「AIがコストになる企業」の**二極化元年**

## まとめ：今すぐやるべきこと

1. **MCPに慣れておく**
   - [MCP公式ドキュメント](https://modelcontextprotocol.io/)をチェック
   - Claude Codeを使っているなら既にMCP対応済み

2. **ローンチパートナーのアプリを試す**
   - Claude Pro/Team/Enterpriseで利用可能
   - Figma, Slack, Asanaあたりから始めるのがおすすめ

3. **自社ツールのMCP対応を検討**
   - 社内ツールをMCP Apps化すれば、AIアシスタントと完全統合

## 最後に

MCP Appsは「AIがUIを持つ」という、当たり前のようで革命的な進化です。

2025年は「AIを試す年」でした。
2026年は「AIで成果を出す年」です。

**この波に乗り遅れないでください。**

---

この記事が参考になったら**いいね**と**ストック**をお願いします！

質問があればコメントで聞いてください。特にMCP Appsの具体的な実装について知りたいことがあれば、続編記事で解説します。

**あなたはMCP Appsで何を作りたいですか？** コメントで教えてください！
