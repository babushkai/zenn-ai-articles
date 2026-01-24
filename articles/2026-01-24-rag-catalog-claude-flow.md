---
title: "Claude Flowでマルチエージェント開発 - RAGカタログを30分で構築した方法"
emoji: "🏗️"
type: "tech"
topics: ["claudecode", "rag", "nextjs", "ai", "マルチエージェント"]
published: false
---

## はじめに

「RAGツールが多すぎて、どれを使えばいいかわからない」

LangChain、LlamaIndex、Haystack、Dify... RAGエコシステムは急速に拡大し、45以上のツールが存在します。

そこで、**RAGカタログ**を作りました。

https://rag-catalog.vercel.app

この記事では、**Claude Flow**を使ってマルチエージェント開発でこのサイトを30分で構築した方法を解説します。

## Claude Flowとは

Claude Flowは、Claude Code上で動作するマルチエージェントオーケストレーションフレームワークです。

```bash
# Swarmの初期化
npx @claude-flow/cli@latest swarm init --topology hierarchical-mesh --max-agents 8
```

主な特徴：

| 機能 | 説明 |
|------|------|
| **マルチエージェント** | 複数のエージェントが並列で作業 |
| **階層メッシュ** | 階層的 + メッシュ型のハイブリッド構成 |
| **メモリ管理** | HNSWベクトルDBで高速パターン検索 |
| **自己最適化** | 過去の成功パターンから学習 |

## 開発プロセス

### Step 1: プロジェクト作成

```bash
npx create-next-app@latest ragcatalog --typescript --tailwind --app
```

### Step 2: Claude Flowエージェントを並列起動

Claude Flowの真価は**並列エージェント実行**にあります。

```
# 2つのエージェントを同時に起動
Agent 1: レイアウト + ホームページ + ヘッダー/フッター
Agent 2: カテゴリページ + アイテム詳細ページ + カードコンポーネント
```

これにより、通常の半分の時間で開発が完了しました。

### Step 3: カタログデータの構造化

```typescript
// src/data/catalog.ts
export interface CatalogItem {
  id: string;
  name: string;
  description: string;
  category: Category;
  url: string;
  github?: string;
  stars?: number;
  tags: string[];
  features?: string[];
  useCases?: string[];
}

export type Category =
  | 'frameworks'      // RAGフレームワーク
  | 'evaluation'      // 評価・最適化
  | 'engines'         // RAGエンジン
  | 'data-preparation' // データ準備
  | 'vector-databases' // ベクトルDB
  | 'embedding-models' // エンベディング
  | 'resources';      // リソース
```

## カタログに収録したツール

### RAGフレームワーク（15ツール）

| ツール | GitHub Stars | 特徴 |
|--------|-------------|------|
| LangChain | 105k | 最も人気。プロトタイピングに最適 |
| Dify | 55k | ノーコードでRAGアプリ構築 |
| LlamaIndex | 38k | インデックス構造が強力 |
| RAGFlow | 35k | PDF/テーブル解析に強い |
| Mem0 | 25k | 永続メモリレイヤー |
| DSPy | 20k | プロンプト自動最適化 |

### ベクトルデータベース（6ツール）

| ツール | Stars | 特徴 |
|--------|-------|------|
| Milvus | 32k | 数十億スケール対応 |
| Qdrant | 22k | 高性能フィルタリング |
| Chroma | 16.5k | シンプル、LLMネイティブ |
| pgvector | 13k | PostgreSQL拡張 |
| Weaviate | 12.5k | GraphQL API |

### 評価ツール（6ツール）

- **RAGAS**: LLMベースの自動評価
- **Langfuse**: オープンソースLLMオブザーバビリティ
- **Phoenix**: Arizeによるトレーシング
- **DeepEval**: 14以上のメトリクス + CI/CD統合

## デザインの変遷

### Before: 過剰装飾

最初のデザインは「チーズィー」でした：

- グラデーションブロブ背景
- アニメーションバッジ
- グロー効果
- 紫のグラデーションテキスト

### After: ミニマルデザイン

修正後：

```css
/* 削除したもの */
.gradient-text { ... }      /* グラデーションテキスト */
.glow { ... }               /* グロー効果 */
.card-hover { ... }         /* 派手なホバー */

/* 残したもの */
body {
  background: #09090b;      /* シンプルな黒 */
  color: #fafafa;           /* シンプルな白 */
}
```

**デザイン原則：**

1. ボーダーベースのカード（影なし）
2. リスト形式で一覧性を重視
3. 一貫したzincカラーパレット
4. 余白を多めに

## CI/CD: セマンティックバージョニング

### Conventional Commits

```bash
feat: 新機能追加 → マイナーバージョン (1.x.0)
fix: バグ修正 → パッチバージョン (1.0.x)
feat!: 破壊的変更 → メジャーバージョン (x.0.0)
```

### GitHub Actions設定

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches: [main]

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - run: npm ci
      - run: npx semantic-release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 自動リリース

pushするだけで：

1. **semantic-release**がコミットを分析
2. バージョンを自動決定（v1.0.0 → v1.1.0）
3. CHANGELOGを生成
4. GitHubリリースを作成
5. Vercelが自動デプロイ

## まとめ

### Claude Flowの利点

| 従来の開発 | Claude Flow |
|-----------|-------------|
| 1つずつ実装 | 並列エージェント |
| 手動でファイル作成 | 自動生成 |
| 試行錯誤 | パターン学習で最適化 |

### 成果物

- **サイト**: https://rag-catalog.vercel.app
- **GitHub**: https://github.com/babushkai/ragcatalog
- **バージョン**: v1.0.0（セマンティックバージョニング）
- **ツール数**: 45+

### 次のステップ

- [ ] ツールの比較機能
- [ ] ユーザー投稿
- [ ] スター数の自動更新
- [ ] 英語版

---

**Claude Flowを使えば、複雑なWebアプリケーションも並列開発で高速に構築できます。**

ぜひ試してみてください。

```bash
npx @claude-flow/cli@latest swarm init --topology hierarchical-mesh --max-agents 8
```
