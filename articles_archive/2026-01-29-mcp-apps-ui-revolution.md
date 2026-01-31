---
title: "【革命】MCP Appsでチャット内にUIが表示される時代が来た｜Claude/ChatGPTの新機能を完全解説"
emoji: "🖼️"
type: "tech"
topics: ["mcp", "claudecode", "chatgpt", "ui", "ai"]
published: false
---

**「チャットの中にダッシュボードが表示される」**

**「フォームに入力して、そのままAIに送信できる」**

**「グラフをクリックしたら、AIがそのデータを分析してくれる」**

これ、全部**今日から**できます。

2026年1月26日、MCPの公式拡張機能「**MCP Apps**」がリリースされました。

## 結論から言うと

:::message
MCP Appsを使うと、**AIチャット内にインタラクティブなUI**（ダッシュボード、フォーム、グラフ等）を埋め込めます。Claude、ChatGPT、VSCodeで既に利用可能。
:::

## MCP Appsとは？

### Before: テキストだけの世界

```
ユーザー: 売上データを分析して

AI: 分析結果です：
- 1月: 100万円
- 2月: 120万円
- 3月: 150万円
前月比で平均16.7%の成長率です。
```

### After: UIが埋め込まれる世界

```
ユーザー: 売上データを分析して

AI: 分析結果です：

┌────────────────────────────────────┐
│  📊 売上ダッシュボード              │
│  ┌──────────────────────────────┐ │
│  │     150 ─────────●           │ │
│  │     120 ────●                │ │
│  │     100 ●                    │ │
│  │     ───────────────────      │ │
│  │     1月  2月  3月             │ │
│  └──────────────────────────────┘ │
│  [詳細分析] [CSV出力] [予測表示]    │
└────────────────────────────────────┘

グラフのバーをクリックすると、その月の詳細が表示されます。
```

## 対応クライアント

| クライアント | 対応状況 |
|-------------|---------|
| Claude (Web/Desktop) | ✅ 対応済み |
| ChatGPT | ✅ 対応済み |
| VSCode Insiders | ✅ 対応済み |
| Goose | ✅ 対応済み |

## 技術的な仕組み

### アーキテクチャ

```
┌─────────────────────────────────────────────────────┐
│                    AIクライアント                     │
│  ┌─────────────────────────────────────────────┐   │
│  │               チャットウィンドウ               │   │
│  │  ┌───────────────────────────────────────┐  │   │
│  │  │        サンドボックス化されたiframe        │  │   │
│  │  │  ┌─────────────────────────────────┐  │  │   │
│  │  │  │      MCP App UI (HTML/JS)       │  │  │   │
│  │  │  └─────────────────────────────────┘  │  │   │
│  │  └───────────────────────────────────────┘  │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
                         ↕️ postMessage (JSON-RPC)
┌─────────────────────────────────────────────────────┐
│                    MCPサーバー                       │
│  ┌─────────────────────────────────────────────┐   │
│  │  Tool: analyze_sales                          │   │
│  │  UI Resource: ui://sales-dashboard            │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

### 動作フロー

1. **ツール定義**: MCPサーバーが`ui://`リソースを宣言
2. **ツール呼び出し**: LLMがツールを呼び出し
3. **UI表示**: ホストがリソースを取得し、サンドボックス化されたiframeで表示
4. **双方向通信**: UIとLLMがJSON-RPCでやり取り

## 実装例：売上ダッシュボード

### MCPサーバー側

```typescript
// server.ts
import { Server } from "@modelcontextprotocol/sdk/server";

const server = new Server({
  name: "sales-dashboard",
  version: "1.0.0",
});

// ツール定義
server.tool({
  name: "show_sales_dashboard",
  description: "売上ダッシュボードを表示",
  parameters: {
    type: "object",
    properties: {
      period: { type: "string", enum: ["monthly", "quarterly", "yearly"] }
    }
  },
  // UIリソースを指定
  _meta: {
    ui: {
      resourceUri: "ui://sales-dashboard"
    }
  },
  handler: async ({ period }) => {
    const data = await fetchSalesData(period);
    return { data };
  }
});

// UIリソース定義
server.resource({
  uri: "ui://sales-dashboard",
  mimeType: "text/html",
  handler: async () => {
    return `
      <!DOCTYPE html>
      <html>
      <head>
        <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
        <script type="module">
          import { App } from "@modelcontextprotocol/ext-apps";

          const app = new App();
          await app.connect();

          app.ontoolresult = (result) => {
            renderChart(result.data);
          };

          window.analyzeMonth = async (month) => {
            await app.updateModelContext({
              content: [{ type: "text", text: \`\${month}月の詳細分析をお願いします\` }]
            });
          };
        </script>
      </head>
      <body>
        <canvas id="chart"></canvas>
        <button onclick="analyzeMonth(1)">1月を分析</button>
        <button onclick="analyzeMonth(2)">2月を分析</button>
        <button onclick="analyzeMonth(3)">3月を分析</button>
      </body>
      </html>
    `;
  }
});
```

### クライアント側（React）

```tsx
// 公式SDKを使用
import { useMCPApp } from "@modelcontextprotocol/ext-apps/react";

function SalesDashboard() {
  const { toolResult, callServerTool, updateModelContext } = useMCPApp();

  const handleMonthClick = async (month: number) => {
    // AIに追加の分析を依頼
    await updateModelContext({
      content: [{ type: "text", text: `${month}月の詳細を教えて` }]
    });
  };

  return (
    <div>
      <Chart data={toolResult?.data} onClick={handleMonthClick} />
    </div>
  );
}
```

## ユースケース

### 1. データ可視化

```
ユーザー: このCSVを可視化して

AI: 【インタラクティブなグラフが表示される】
    - ホバーで詳細表示
    - クリックでドリルダウン
    - フィルター機能付き
```

### 2. フォーム入力

```
ユーザー: 新しいユーザーを登録して

AI: 【登録フォームが表示される】
    名前: [________]
    メール: [________]
    権限: [ドロップダウン▼]
    [登録] [キャンセル]
```

### 3. ドキュメントレビュー

```
ユーザー: この契約書をレビューして

AI: 【PDFビューアが表示される】
    - 問題箇所がハイライト
    - クリックで詳細説明
    - 承認/却下ボタン
```

### 4. リアルタイム監視

```
ユーザー: サーバーの状態を見せて

AI: 【ダッシュボードが表示される】
    CPU: ████████░░ 80%
    メモリ: ██████░░░░ 60%
    [自動更新中...]
```

## セキュリティ

### サンドボックス化

MCP Appsは複数のセキュリティレイヤーで保護されています：

1. **iframeサンドボックス**: 制限された権限で実行
2. **事前宣言テンプレート**: ホストがUIをレビュー可能
3. **監査可能なメッセージ**: JSON-RPCで全通信を記録
4. **ユーザー同意**: UIからのツール呼び出しには承認が必要

```html
<!-- サンドボックス属性 -->
<iframe
  sandbox="allow-scripts allow-same-origin"
  src="..."
></iframe>
```

## 今すぐ試す方法

### 1. Claude Desktopで試す

Claude Desktopをアップデートすれば、MCP Apps対応のMCPサーバーを接続できます。

### 2. サンプルサーバーを動かす

```bash
# 公式サンプルをクローン
git clone https://github.com/modelcontextprotocol/ext-apps
cd ext-apps/examples

# 依存関係インストール
npm install

# サーバー起動
npm run dev
```

### 3. Claude設定に追加

```json
// ~/.claude/mcp_settings.json
{
  "servers": {
    "sales-dashboard": {
      "command": "node",
      "args": ["path/to/server.js"]
    }
  }
}
```

## 対応フレームワーク

| フレームワーク | SDK | 状況 |
|--------------|-----|------|
| React | `@modelcontextprotocol/ext-apps/react` | ✅ |
| Vue | `@modelcontextprotocol/ext-apps/vue` | ✅ |
| Svelte | `@modelcontextprotocol/ext-apps/svelte` | ✅ |
| Vanilla JS | `@modelcontextprotocol/ext-apps` | ✅ |

## AWSエンジニアのコメント

> 「MCP Appsは、エージェントツールが提供できるものと、ユーザーが自然に対話したい方法との間の**実際のギャップ**に対応しています」
> — Clare Liguori, AWSシニアプリンシパルエンジニア

## まとめ

1. **チャット内にUIを埋め込める** - ダッシュボード、フォーム、グラフ
2. **双方向通信** - UIからAIにメッセージを送れる
3. **主要クライアント対応済み** - Claude、ChatGPT、VSCode
4. **セキュア** - サンドボックス化されたiframe

:::message
MCP Appsは、AIとの対話を**テキストベース**から**ビジュアルベース**に進化させます。これは「チャットUI」の概念そのものを変える可能性があります。
:::

---

この記事が役に立ったら、**いいね**と**保存**をお願いします！

**質問**: MCP Appsでどんなアプリを作りたいですか？アイデアがあればコメントで教えてください！

## 参考リンク

- [MCP Apps - Official Announcement](http://blog.modelcontextprotocol.io/posts/2026-01-26-mcp-apps/)
- [GitHub - modelcontextprotocol/ext-apps](https://github.com/modelcontextprotocol/ext-apps)
- [Claude supports MCP Apps - The Register](https://www.theregister.com/2026/01/26/claude_mcp_apps_arrives/)
