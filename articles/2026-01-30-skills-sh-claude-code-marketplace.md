---
title: "【保存版】skills.shが神すぎる！Claude Codeを10倍強化する最強スキル20選"
emoji: "🧠"
type: "tech"
topics: ["Claude", "ClaudeCode", "AI", "Vercel", "開発効率化"]
published: true
---

## まだ素のClaude Code使ってるの？それ、損してますよ

**結論から言うと、skills.shを使えばClaude Codeが別次元の存在になります。**

2026年1月、Vercelが[skills.sh](https://skills.sh)をローンチ。

**6時間で2万インストール突破。**

Guillermo Rauch（Vercel CEO）曰く：

> "A package manager for AI coding assistants. One command, and your AI agent knows 10+ years of React and Next.js optimization patterns."
> 「AIコーディングアシスタント用のパッケージマネージャー。1コマンドで、AIエージェントが10年分以上のReact/Next.js最適化パターンを習得する」

**これ、npmのAIエージェント版です。**

## 🚀 skills.shとは？

[skills.sh](https://skills.sh)は**AIエージェント用スキルのオープンエコシステム**です。

```bash
npx skills add <owner/repo>
```

たった1コマンドで、Claude Code、Cursor、GitHub Copilot、Geminiなど**15以上のAIコーディングツール**にスキルをインストールできます。

### 対応AIエージェント

| エージェント | 対応状況 |
|:----------:|:-------:|
| Claude Code | ✅ |
| GitHub Copilot | ✅ |
| Cursor | ✅ |
| Cline | ✅ |
| Gemini | ✅ |
| Windsurf | ✅ |
| OpenAI Codex | ✅ |

**業界標準になりつつあります。**

## 📊 今バズってるスキルTOP10

2026年1月30日時点のランキングです：

| 順位 | スキル名 | インストール数 | 用途 |
|:---:|:--------|:-------------:|:-----|
| 1 | vercel-react-best-practices | 72,500+ | React/Next.js最適化 |
| 2 | find-skills | 55,900+ | スキル検索 |
| 3 | web-design-guidelines | 55,000+ | UI/UXレビュー |
| 4 | remotion-best-practices | 52,200+ | 動画生成 |
| 5 | frontend-design | 26,900+ | フロントエンド設計 |
| 6 | playwright-automation | 18,000+ | E2Eテスト自動化 |
| 7 | prompt-engineering | 15,000+ | プロンプト最適化 |
| 8 | stripe-integration | 12,000+ | 決済実装 |
| 9 | react-native-guidelines | 10,000+ | モバイル開発 |
| 10 | vercel-deploy-claimable | 8,000+ | ワンクリックデプロイ |

**合計35,000以上のスキルが公開されています。**

## 🔥 神スキル詳細解説

### 1. vercel-react-best-practices（72.5Kインストール）

**Vercel公式が作った、10年分のReact最適化ノウハウ。**

```bash
npx skills add vercel-labs/agent-skills
```

40以上のルールが8カテゴリに分類：

| カテゴリ | 例 |
|:--------|:---|
| インポート最適化 | バレルファイル回避で200-800ms短縮 |
| useEffect | カスケード呼び出し検出 |
| データフェッチ | Server Components活用 |
| バンドルサイズ | 動的インポート推奨 |

各ルールに**CRITICAL〜LOW**の影響度ラベル付き。

**インストール後の変化：**

```
Before: 「このコンポーネント作って」→ 普通のReactコード
After: 「このコンポーネント作って」→ Vercelエンジニア品質のコード
```

### 2. web-design-guidelines（55Kインストール）

**100以上のUI/UXルールでコードレビュー。**

チェック項目：
- ✅ アクセシビリティ（フォーカス状態、ARIAラベル）
- ✅ パフォーマンス（画像最適化、レイジーロード）
- ✅ UX（フォームバリデーション、エラーハンドリング）
- ✅ 国際化（RTL対応、翻訳準備）

```bash
# UIレビューを依頼するだけで自動適用
「このフォームのUIをレビューして」
```

### 3. remotion-best-practices（52.2Kインストール）

**Reactで動画を作るRemotionの公式スキル。**

Jonny Burger（Remotion作者）が発表したデモ動画は**14.7万回再生**。

```bash
「30秒のプロモーション動画を作って」
```

Claude Codeが自動でRemotionコードを生成し、動画を出力！

### 4. playwright-automation

**ブラウザ自動化の専門家に変身。**

```bash
「このサイトのE2Eテストを書いて」
```

- ページオブジェクトパターン自動適用
- 待機戦略の最適化
- スクリーンショット・動画キャプチャ

### 5. prompt-engineering

**Anthropic公式のプロンプトエンジニアリングベストプラクティス。**

```bash
「このプロンプトを最適化して」
```

- 構造化出力の誘導
- Few-shot例の配置
- システムプロンプトの設計

## 📦 インストール方法

### 方法1: npxコマンド（推奨）

```bash
# Vercel公式スキルセット
npx skills add vercel-labs/agent-skills

# 個別スキル
npx skills add owner/skill-name
```

### 方法2: Claude Codeプラグイン経由

```bash
# マーケットプレイスを追加
/plugin marketplace add anthropics/skills

# スキルをインストール
/plugin install example-skills@anthropic-agent-skills
```

### 方法3: 手動インストール

`.claude/skills/`フォルダに`SKILL.md`を配置：

```markdown
---
name: my-custom-skill
description: カスタムスキルの説明
triggers:
  - keyword: "レビュー"
  - keyword: "最適化"
---

# スキルの指示内容
このスキルが呼び出されたら、以下のルールに従ってください...
```

## 🎯 用途別おすすめスキル

### フロントエンド開発者向け

```bash
npx skills add vercel-labs/agent-skills  # React最適化
npx skills add design/web-design-guidelines  # UIレビュー
npx skills add testing/playwright-automation  # E2Eテスト
```

### バックエンド開発者向け

```bash
npx skills add api/openapi-generator  # API設計
npx skills add db/prisma-best-practices  # DB最適化
npx skills add security/owasp-guidelines  # セキュリティ
```

### モバイル開発者向け

```bash
npx skills add vercel-labs/react-native-guidelines  # RN最適化
npx skills add mobile/expo-best-practices  # Expo設定
```

### DevOps向け

```bash
npx skills add ci/github-actions-patterns  # CI/CD
npx skills add deploy/vercel-deploy-claimable  # デプロイ
npx skills add infra/terraform-modules  # IaC
```

## 💡 自作スキルの作り方

### 1. SKILL.mdを作成

```markdown
---
name: my-review-skill
description: コードレビューの専門家
triggers:
  - keyword: "レビュー"
  - keyword: "PR"
---

# コードレビュースキル

## レビュー観点
1. **可読性**: 変数名、関数名は意図が明確か
2. **保守性**: 将来の変更に耐えられるか
3. **パフォーマンス**: N+1クエリ、不要な再レンダリング
4. **セキュリティ**: インジェクション、XSS

## 出力フォーマット
- 🔴 CRITICAL: 必ず修正
- 🟡 WARNING: 推奨修正
- 🟢 SUGGESTION: 検討
```

### 2. 配置

```
~/.claude/skills/my-review-skill/SKILL.md  # グローバル
.claude/skills/my-review-skill/SKILL.md    # プロジェクト単位
```

### 3. 自動で有効化

Claudeはタスクに基づいて**関連するスキルを自動で呼び出し**ます。手動選択不要！

## ⚠️ 注意点とベストプラクティス

:::message alert
スキルはClaudeにコード実行権限を与えます。信頼できるソースのみ使用してください！
:::

### セキュリティ

1. **公式・有名リポジトリ優先** - vercel-labs、anthropicsなど
2. **スクリプト内容を確認** - 実行前に中身をチェック
3. **APIキーは環境変数で** - ハードコード厳禁

### パフォーマンス

1. **スキルは必要最小限** - 入れすぎるとコンテキスト圧迫
2. **長いスキルは分割** - SKILL.mdはメニューとして機能させる
3. **トリガーを適切に設定** - 不要な呼び出しを防ぐ

## 🚀 まとめ

| ポイント | 内容 |
|:--------|:-----|
| **skills.shとは** | AIエージェント用スキルのnpm |
| **対応ツール** | Claude Code、Cursor、Copilotなど15+ |
| **スキル数** | 35,000以上が公開中 |
| **インストール** | `npx skills add <owner/repo>` |
| **神スキル** | vercel-react-best-practices（72.5K） |

**素のClaude Codeを使い続けるのは、素のnpmを使い続けるようなもの。**

スキルを入れて、Claude Codeを**10年分の専門知識を持ったシニアエンジニア**に変えましょう。

---

:::message
この記事が参考になったら**いいね**と**ストック**をお願いします！
「このスキルがおすすめ！」という情報があれば、ぜひコメントで教えてください！
:::

## 参考リンク

- [skills.sh公式サイト](https://skills.sh)
- [vercel-labs/agent-skills (GitHub)](https://github.com/vercel-labs/agent-skills)
- [Vercel公式ブログ - Skills発表](https://vercel.com/changelog/introducing-skills-the-open-agent-skills-ecosystem)
- [Claude Code Skills ドキュメント](https://code.claude.com/docs/ja/skills)
- [Claude Code Skillsの使い方 (Zenn)](https://zenn.dev/kg_filled/articles/50f762610d48c7)
