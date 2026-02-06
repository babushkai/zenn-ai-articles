---
title: "【速報】WordPress.comがClaude公式連携！AIにサイト分析を丸投げする時代が来た"
emoji: "🚀"
type: "tech"
topics: ["wordpress", "claude", "mcp", "ai", "ブログ"]
published: true
---

**あなたのWordPressサイト、最後にちゃんと分析したのはいつですか？**

「Google Analyticsは見てるけど、結局何をすればいいかわからない」
「古い記事が放置されてるけど、どれを更新すべきかわからない」
「コメント管理が面倒すぎて見て見ぬふりをしている」

こんな悩み、ありませんか？

**2026年2月5日、WordPress.comが歴史的な発表をしました。**

Claude（Anthropic）との公式コネクタをリリース。これはWordPressホスティング業界初の公式AI連携です。

つまり、**AIにサイトデータを読ませて、具体的なアドバイスをもらえる時代が来た**ということです。

## 結論から言うと...

:::message
WordPress.com + Claude連携でできること：
1. **過去30日間のトラフィック分析**をAIが自動で読み解く
2. **1年以上更新されていない古い記事**を自動検出
3. **エンゲージメントが低下しているページ**を発見
4. **サイト全体のコメント傾向**を分析
5. すべて「あなたのサイトの実データ」に基づいた具体的アドバイス
:::

一般的なAIの「SEOを頑張りましょう」みたいな抽象的アドバイスではなく、**「この記事のPVが先月比50%減少しています。タイトルを変更することを検討してください」**のような具体的指摘が可能になりました。

## なぜこれが革命的なのか？

### Before: 従来のサイト分析

```
1. Google Analyticsにログイン
2. データをエクスポート
3. スプレッドシートで分析
4. 何をすべきか自分で考える
5. 結局よくわからないまま放置
```

### After: Claude連携後

```
あなた: 「私のサイトで改善すべき点を教えて」

Claude: 「分析しました。以下の3つの記事が要注意です：
1. /blog/old-article-2024/ - 1年以上更新なし、月間PV 500
2. /blog/tutorial-post/ - 直帰率85%、コンテンツ追加推奨
3. /blog/news-item/ - コメント0件、CTAの見直しを」
```

**所要時間：約30秒。**

## セットアップ方法【5分で完了】

### ステップ1: WordPress.comでMCPを有効化

まず、WordPress.comの管理画面でMCP（Model Context Protocol）を有効にします。

:::message alert
**重要**: MCP機能は有料プラン限定です。無料プランでは利用できません。
:::

### ステップ2: 公開するツールを選択

Claudeにアクセスを許可するツール（機能）を選びます。現時点では**読み取り専用**なので、AIがあなたのコンテンツを勝手に変更・削除する心配はありません。

### ステップ3: Claude側で接続

**Claude Desktop / Webの場合：**

1. Settings を開く
2. Connectors をクリック
3. "Browse connectors" ボタンを押す
4. "WordPress.com" を検索
5. "+" ボタンで接続
6. OAuth 2.1認証を完了

これだけです。

### ステップ4: 使い始める

接続後、Claudeに以下のような質問ができます：

- 「私のサイトの過去30日間のトラフィックを分析して」
- 「更新が必要な古い記事をリストアップして」
- 「最もエンゲージメントが高い記事の共通点は？」
- 「コメントの傾向から読者が求めているものは？」

## 技術的な仕組み：MCP Adapterとは

裏側では**WordPress MCP Adapter**というプラグインが動いています。

```
WordPress Abilities API
        ↓
    MCP Adapter（変換）
        ↓
  Model Context Protocol
        ↓
   Claude / 他のAI
```

WordPress側で登録された「Abilities（機能）」をMCPフォーマットに変換し、AIエージェントが発見・実行できるようにしています。

### デフォルトで提供される3つのツール

| ツール名 | 機能 |
|---------|------|
| `mcp-adapter-discover-abilities` | 利用可能な機能の一覧表示 |
| `mcp-adapter-get-ability-info` | 機能の詳細情報を取得 |
| `mcp-adapter-execute-ability` | 機能を実行 |

開発者は独自のAbilitiesを追加することで、Claudeにさらに多くの操作を許可できます。

## セキュリティは大丈夫？

心配な方も多いと思いますが、以下の点が確保されています：

1. **読み取り専用**: コンテンツの作成・削除・更新は**不可**
2. **OAuth 2.1認証**: 最新のセキュリティプロトコルを採用
3. **いつでも取り消し可能**: 接続はユーザーがコントロール
4. **権限チェック**: `permission_callback`による細かい権限制御
5. **公式サポート**: Anthropic・WordPress.com両社が公式にサポート

:::message
**野良プラグインではなく公式連携**というのが重要なポイントです。セキュリティリスクを最小限に抑えつつ、AI活用のメリットを享受できます。
:::

## 実際にどう使う？5つのユースケース

### 1. コンテンツ監査の自動化

**プロンプト例：**
```
私のサイトで以下の条件に当てはまる記事をリストアップして：
- 1年以上更新なし
- 月間PVが100以下
- それでも検索流入がある
リライト優先度をつけて提示して。
```

### 2. 読者インサイトの発見

**プロンプト例：**
```
過去30日間のコメントを分析して、
読者が最も興味を持っているトピックを3つ教えて。
次に書くべき記事のアイデアも提案して。
```

### 3. トラフィック異常の検知

**プロンプト例：**
```
先月と比べてPVが大きく変動した記事を教えて。
増加した記事と減少した記事、それぞれの理由を推測して。
```

### 4. 内部リンク最適化

**プロンプト例：**
```
私のサイトで最もPVの高い記事トップ10と、
そこからリンクされていない関連記事を見つけて。
内部リンクの追加提案をして。
```

### 5. 競合分析のベース作成

**プロンプト例：**
```
私のサイトの強みと弱みを、
トラフィックデータとコンテンツ構成から分析して。
どのカテゴリを強化すべきか提案して。
```

## 他のAIエージェントとも連携可能

WordPress MCP Adapterは**Claude専用ではありません**。MCPに対応している他のAIツールとも連携できます：

- **Cursor**（AIコードエディタ）
- **その他のMCP対応AIアシスタント**

つまり、今後登場するAIエージェントとも自動的に互換性を持つ可能性があります。

## 注意点と制限

### 現時点での制限

1. **有料プラン限定**: WordPress.comの無料プランでは利用不可
2. **WordPress.com限定**: 自己ホストのWordPress.orgでは別途プラグインが必要
3. **読み取り専用**: 現時点では編集機能はなし（今後拡張の可能性）

### セルフホスト（WordPress.org）の場合

WordPress.orgを使っている場合は、GitHubからMCP Adapterプラグインをダウンロードして手動でインストールする必要があります。

設定ファイル例（Claude Desktop）：
```json
{
  "mcpServers": {
    "wordpress": {
      "command": "node",
      "args": ["/path/to/wp-mcp-server/index.js"],
      "env": {
        "WP_URL": "https://your-site.com",
        "WP_AUTH_TOKEN": "your-application-password"
      }
    }
  }
}
```

## まとめ：AIがサイト運営のパートナーになる時代

WordPress.comとClaudeの公式連携は、**単なる便利機能の追加ではありません**。

これは「AIがあなたの実データを理解して、具体的なアドバイスをくれる」という新しいパラダイムの始まりです。

:::message
**今日からできること：**
1. WordPress.comの有料プランユーザーは今すぐ設定可能
2. 5分のセットアップでAIサイト分析が使い放題
3. 古い記事の棚卸し、トラフィック分析、コメント分析を自動化
:::

「ブログ運営って面倒」と思っていた人こそ、この連携を試してみてください。AIに面倒な分析を丸投げして、**あなたは書くことに集中する**。それが2026年のブログ運営スタイルです。

---

**この記事が役に立ったら、いいね・保存をお願いします！**

質問やフィードバックがあれば、コメントで教えてください。WordPress × AIの活用法、もっと深掘りした記事も書いていきます。

**あなたはもうWordPress + Claude連携を試しましたか？感想を教えてください！**

## 参考リンク

WordPress.com has a Claude Connector

https://wordpress.com/blog/2026/02/05/claude-connector/

From Abilities to AI Agents: Introducing the WordPress MCP Adapter

https://developer.wordpress.org/news/2026/02/from-abilities-to-ai-agents-introducing-the-wordpress-mcp-adapter/

Connect AI Agents to WordPress.com with OAuth 2.1 and MCP

https://wordpress.com/blog/2026/01/22/connect-ai-agents-to-wordpress-oauth-2-1/
