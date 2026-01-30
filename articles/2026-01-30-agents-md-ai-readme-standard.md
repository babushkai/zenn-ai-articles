---
title: "【知らないと損】AGENTS.mdが業界標準に！AIエージェント専用READMEの書き方完全ガイド"
emoji: "📋"
type: "tech"
topics: ["AI", "AIエージェント", "OpenAI", "GitHub", "開発効率化"]
published: true
---

## まだREADME.mdだけで満足してるの？

**結論から言うと、AIエージェント時代には「AGENTS.md」が必須です。**

- README.md → **人間用**
- AGENTS.md → **AI用**

この使い分け、できてますか？

[AGENTS.md](https://agents.md)は**6万以上のオープンソースプロジェクト**で採用され、**Linux Foundation傘下のAgentic AI Foundation**が管理する**業界標準**になっています。

OpenAIのメインリポジトリには**88個のAGENTS.md**が配置されているほど。

## 🤖 AGENTS.mdとは？

> "Think of AGENTS.md as a README for agents"
> 「AGENTS.mdはエージェント向けのREADME」

READMEは人間向けのクイックスタートや貢献ガイドライン。

AGENTS.mdは**AIコーディングエージェントが必要とする詳細なコンテキスト**を提供します：

- ビルドコマンド
- テスト手順
- コーディング規約
- セキュリティ考慮事項
- 触ってはいけないファイル

**READMEを肥大化させずに、AIに必要な情報を渡せる。**

## 🌐 対応AIツール（20以上！）

| ツール | 対応状況 |
|:------|:-------:|
| OpenAI Codex | ✅ |
| Google Jules | ✅ |
| GitHub Copilot | ✅ |
| Cursor | ✅ |
| Claude Code | ✅ |
| VS Code AI | ✅ |
| Factory | ✅ |
| Amp | ✅ |
| RooCode | ✅ |
| Gemini (Android Studio) | ✅ |

**主要AIコーディングツールが全対応。事実上の標準です。**

## 📊 GitHubの2,500リポジトリ分析から学ぶベストプラクティス

[GitHub公式ブログ](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/)が2,500以上のリポジトリを分析した結果、成功パターンが判明しました。

### 成功する6つの要素

1. **実行可能なコマンド**（フラグ付き）
2. **テスト手順**
3. **プロジェクト構造**
4. **コードスタイル例**
5. **Gitワークフロー**
6. **明確な境界線**

### 最も重要な教訓

:::message
「1つの実際のコードスニペットは、3段落の説明に勝る」
:::

抽象的な説明より、**具体的な例を見せる**のが最強。

## 🔥 神AGENTS.mdテンプレート

### 基本テンプレート（150行以内推奨）

```markdown
# AGENTS.md

## Overview
このプロジェクトは[技術スタック]を使用した[プロジェクトの目的]です。

## Tech Stack
- React 18 + TypeScript 5.3
- Vite 5.x
- Tailwind CSS 3.x
- Vitest for testing

## Commands
```bash
# 依存関係インストール
pnpm install

# 開発サーバー起動
pnpm dev

# テスト実行
pnpm test

# ビルド
pnpm build

# リント
pnpm lint
```

## Code Style
- TypeScript strict mode必須
- ESLint + Prettierの設定に従う
- Conventional Commits形式でコミット

### Good Example
```typescript
// ✅ Good: 明示的な型定義
interface UserProps {
  id: string;
  name: string;
  email: string;
}

export const UserCard: React.FC<UserProps> = ({ id, name, email }) => {
  return (
    <div className="rounded-lg p-4 shadow-md">
      <h2 className="text-lg font-bold">{name}</h2>
      <p className="text-gray-600">{email}</p>
    </div>
  );
};
```

### Bad Example
```typescript
// ❌ Bad: any型、省略されたprops
export const UserCard = (props: any) => {
  return <div>{props.name}</div>;
};
```

## Testing Guidelines
- Vitestを使用
- カバレッジ目標: 80%以上
- コミット前に必ず `pnpm test` を実行

## Project Structure
```
src/
├── components/    # UIコンポーネント
├── hooks/         # カスタムフック
├── lib/           # ユーティリティ
├── pages/         # ページコンポーネント
└── types/         # 型定義
```

## Boundaries (絶対に触るな)
- `.env*` - 環境変数ファイル
- `node_modules/` - 依存関係
- `dist/` - ビルド成果物
- `*.lock` - ロックファイル

## Git Workflow
1. `feature/` ブランチを作成
2. 変更をコミット（Conventional Commits）
3. Draft PRを作成
4. テストがパスしたらReady for Review
```

## 🎯 用途別カスタマイズ

### フロントエンド（React/Next.js）

```markdown
## Component Guidelines
- Functional componentsのみ使用
- Props interfaceは必ず定義
- Storybook storiesを追加

## State Management
- Zustandを使用（Reduxは使わない）
- サーバー状態はTanStack Query

## Styling
- Tailwind CSSのみ（CSS-in-JS禁止）
- cn()ヘルパーでクラス名を結合
```

### バックエンド（Node.js/Express）

```markdown
## API Guidelines
- RESTful設計に従う
- エンドポイントは `/api/v1/` プレフィックス
- エラーハンドリングは統一フォーマット

## Database
- Prismaを使用
- マイグレーションは `pnpm db:migrate`
- N+1クエリに注意

## Security
- 入力は必ずバリデーション
- SQLインジェクション対策済み
- 認証はJWT Bearer Token
```

### モノレポ構成

```markdown
## Monorepo Structure
各パッケージに個別のAGENTS.mdを配置。
最も近いファイルが優先されます。

packages/
├── web/AGENTS.md      # Webアプリ固有の指示
├── api/AGENTS.md      # API固有の指示
└── shared/AGENTS.md   # 共有ライブラリ固有の指示
```

## ⚡ 効果的なAGENTS.mdの書き方5箇条

### 1. コマンドを最初に書く

```markdown
## Commands
```bash
pnpm test        # テスト実行
pnpm build       # 本番ビルド
pnpm lint:fix    # リント自動修正
```
```

ツール名だけでなく**フラグとオプション**も含める。

### 2. 説明より例を見せる

```markdown
❌ 「関数は小さく保ち、単一責任にしてください」

✅
```typescript
// Good
const calculateTotal = (items: Item[]) =>
  items.reduce((sum, item) => sum + item.price, 0);

// Bad
const processOrder = (order) => {
  // 100行の処理...
}
```
```

### 3. 境界線を明確にする

```markdown
## Never Touch
- `/secrets/` - 機密情報
- `/vendor/` - サードパーティコード
- `config.production.ts` - 本番設定
```

### 4. 具体的なペルソナを与える

```markdown
❌ 「便利なコーディングアシスタント」

✅ 「Reactコンポーネントテストを書くテストエンジニア。
   以下の例に従い、ソースコードは絶対に変更しない」
```

### 5. 150行以内に収める

長すぎるファイルは：
- エージェントの処理を遅くする
- 重要な情報が埋もれる
- メンテナンスが困難

**凝縮された情報 > 網羅的な情報**

## 📁 配置場所

```
your-project/
├── AGENTS.md          # ルート（必須）
├── README.md
├── packages/
│   ├── web/
│   │   └── AGENTS.md  # サブプロジェクト用
│   └── api/
│       └── AGENTS.md  # サブプロジェクト用
```

**ディレクトリツリーで最も近いファイルが優先されます。**

## 🆚 CLAUDE.md vs AGENTS.md

| 項目 | CLAUDE.md | AGENTS.md |
|:-----|:---------|:----------|
| 対象 | Claude Code専用 | 全AIエージェント |
| 標準化 | Anthropic独自 | Linux Foundation |
| 採用数 | 不明 | 60,000+ |
| 互換性 | Claude Codeのみ | 20+ツール対応 |

**両方置くのがベストプラクティス。**

```
your-project/
├── AGENTS.md    # 全AIエージェント用
├── CLAUDE.md    # Claude Code専用の追加指示
└── README.md    # 人間用
```

## 🚀 今すぐ始める3ステップ

### Step 1: テンプレートをコピー

```bash
curl -o AGENTS.md https://raw.githubusercontent.com/agentsmd/agents.md/main/examples/basic.md
```

### Step 2: プロジェクトに合わせて編集

- Tech Stackを更新
- コマンドを追加
- 境界線を設定

### Step 3: コミット

```bash
git add AGENTS.md
git commit -m "docs: add AGENTS.md for AI coding agents"
```

## まとめ

| ポイント | 内容 |
|:--------|:-----|
| **AGENTS.mdとは** | AIエージェント専用のREADME |
| **採用数** | 60,000+プロジェクト |
| **管理団体** | Linux Foundation |
| **対応ツール** | Codex、Copilot、Cursorなど20+ |
| **推奨行数** | 150行以内 |
| **必須要素** | コマンド、例、境界線 |

**AIエージェントを使うなら、AGENTS.mdは必須インフラ。**

READMEだけの時代は終わりました。

---

:::message
この記事が参考になったら**いいね**と**ストック**をお願いします！
あなたのAGENTS.mdを見せてください！コメントで共有お待ちしてます！
:::

## 参考リンク

- [AGENTS.md 公式サイト](https://agents.md)
- [agentsmd/agents.md (GitHub)](https://github.com/agentsmd/agents.md)
- [How to write a great agents.md (GitHub Blog)](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/)
- [AGENTS.mdとは？(AI総合研究所)](https://www.ai-souken.com/article/what-is-agents-md)
- [AGENTS.md 日本語テンプレート (GitHub Gist)](https://gist.github.com/sifue/d308e5caee84fbb96fb79c6231e3636e)
