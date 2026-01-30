---
title: "【革命】AIがUIを表示する時代が来た！MCP Appsで開発が根本から変わる"
emoji: "🚀"
type: "tech"
topics: ["AI", "MCP", "Claude", "ChatGPT", "LLM"]
published: true
---

## まだテキストだけでAIと会話してるの？

**結論から言うと、AIとの対話が「テキストのやり取り」から「インタラクティブなUI操作」に進化しました。**

2026年1月26日、MCPの公式ブログで衝撃の発表がありました。

> "Tools can now return interactive UI components that render directly in the conversation."
> 「ツールがインタラクティブなUIコンポーネントを会話内で直接レンダリングできるようになった」

つまり、**AIがダッシュボード、フォーム、グラフ、リアルタイムモニタリング画面をチャット内に表示できる**ようになったんです。

これ、ヤバくないですか？

## 🤯 MCP Appsとは何か？

MCP Apps は**MCPの初めての公式拡張機能**です。

これまでのAIとの対話：
```
あなた: 売上データを分析して
AI: はい、分析結果です。1月は前月比15%増で...（テキストが延々と続く）
```

MCP Apps導入後：
```
あなた: 売上データを分析して
AI: [インタラクティブなダッシュボードがチャット内に表示される]
    [グラフをクリックするとドリルダウンできる]
    [フィルタを変更すると即座に更新される]
```

**文字で読む時代は終わりました。**

## 🏗️ 技術的な仕組み

MCP Appsは2つの重要なプリミティブで構成されています：

### 1. UIメタデータ付きツール

```typescript
// ツールがUIリソースへの参照を返す
{
  _meta: {
    ui: {
      resourceUri: "ui://my-server/dashboard"
    }
  }
}
```

### 2. UIリソース

`ui://` スキームで提供されるサーバーサイドリソース。HTML/JavaScriptがバンドルされています。

```typescript
// サーバー側のUI定義
const dashboardResource = {
  uri: "ui://analytics/dashboard",
  content: bundledHtmlAndJs
};
```

ホスト（Claude、ChatGPTなど）がリソースを取得し、**サンドボックス化されたiframe**内でレンダリング。JSON-RPCとpostMessageで双方向通信を実現しています。

## 🎯 具体的に何ができるのか？

### 1. データ探索ダッシュボード

```typescript
// フィルタリングとドリルダウン機能付きダッシュボード
app.onToolResult((result) => {
  renderChart(result.data);
  enableDrillDown();
});
```

- クリックで詳細表示
- 日付範囲の変更
- カテゴリフィルタ

### 2. 設定ウィザード

```typescript
// 依存フィールドを持つ設定フォーム
app.renderForm({
  fields: [
    { name: "環境", type: "select", options: ["開発", "本番"] },
    { name: "リージョン", type: "select", dependsOn: "環境" }
  ]
});
```

環境を「本番」に変えると、リージョンの選択肢が自動で変わる！

### 3. ドキュメントレビュー

- PDFをインライン表示
- ハイライト機能
- コメント追加

### 4. リアルタイムモニタリング

```typescript
// ライブメトリクス更新
app.subscribe("metrics", (data) => {
  updateDashboard(data);
});
```

サーバーの状態、APIレスポンス時間、エラー率をリアルタイムで監視！

## 🛠️ 開発者SDK

`@modelcontextprotocol/ext-apps`パッケージが提供されています：

```typescript
import { App } from '@modelcontextprotocol/ext-apps';

const app = new App();

// ツール結果を受け取る
app.onToolResult((result) => {
  console.log('受信:', result);
});

// UIからサーバーツールを呼び出す
await app.callTool('analyzeData', { query: 'sales' });

// モデルコンテキストを更新
app.updateContext({ userSelection: selectedItem });
```

**フレームワークに依存しない**標準的なpostMessage通信なので、React、Vue、Svelteなど好きなものが使えます。

## ✅ 対応クライアント

すでに以下のクライアントがサポートしています：

| クライアント | 状態 |
|:----------:|:----:|
| Claude (Web/Desktop) | ✅ 対応済み |
| Visual Studio Code Insiders | ✅ 対応済み |
| Goose | ✅ 対応済み |
| ChatGPT | 🚀 今週ロールアウト中 |

**主要なAIクライアントが一斉に対応**したのは、MCPがAI業界の標準になりつつある証拠です。

## 🔒 セキュリティ

「AIがUIを表示するなんて危険では？」と思った方、安心してください。

- **iframeサンドボックス**: 制限された権限で実行
- **事前宣言テンプレート**: ホストがUIを事前にレビュー可能
- **監査可能なJSON-RPCメッセージ**: すべての通信をログ記録
- **ユーザー同意**: ツール実行前に必ず確認

セキュリティファーストで設計されています。

## 🌊 なぜこれが革命なのか？

### Before: テキストの海で溺れる

```
AI: 分析結果です。
- 1月: 売上1,234,567円（前月比+15%）
- 2月: 売上1,345,678円（前月比+9%）
- 3月: 売上1,456,789円（前月比+8%）
... (50行続く)
```

読むの疲れますよね？

### After: 直感的なインタラクション

チャット内にグラフが表示され、クリックするだけで詳細が見れる。フィルタを変えれば即座に更新。

**情報の消費速度が10倍以上になります。**

## 💡 活用アイデア

1. **営業ダッシュボード**: 「今月の売上見せて」→ インタラクティブなグラフが表示
2. **コードレビュー**: 「このPRレビューして」→ diffビューアがインライン表示
3. **データベースGUI**: 「ユーザーテーブル見せて」→ フィルタ可能なテーブルUI
4. **設定管理**: 「環境変数を設定」→ フォームUIで安全に入力
5. **監視ダッシュボード**: 「サーバー状態は？」→ リアルタイムメトリクス表示

## 🚀 今すぐ試す方法

### 1. Claude Desktopで試す

最新版のClaude Desktopをインストールすれば、MCP Apps対応のMCPサーバーと連携できます。

### 2. VS Code Insidersで試す

VS Code InsidersでMCP拡張機能をインストールし、対応サーバーを設定。

### 3. 自分でMCPサーバーを作る

```bash
npm install @modelcontextprotocol/ext-apps
```

## まとめ

- **MCP Apps**はMCP初の公式拡張機能
- AIがチャット内で**インタラクティブなUI**を表示可能に
- **ダッシュボード、フォーム、リアルタイムモニタリング**など多彩な用途
- Claude、ChatGPT、VS Codeなど**主要クライアントが対応**
- セキュリティも**サンドボックス+ユーザー同意**で安心

**テキストだけのAI対話は過去のものになります。**

---

:::message
この記事が参考になったら**いいね**と**ストック**をお願いします！
「MCP Appsでこんな使い方してみた」という方は、ぜひコメントで教えてください！
:::

## 参考リンク

- [MCP Apps公式ブログ](http://blog.modelcontextprotocol.io/posts/2026-01-26-mcp-apps/)
- [Model Context Protocol公式サイト](https://modelcontextprotocol.io/)
- [@modelcontextprotocol/ext-apps npm](https://www.npmjs.com/package/@modelcontextprotocol/ext-apps)
