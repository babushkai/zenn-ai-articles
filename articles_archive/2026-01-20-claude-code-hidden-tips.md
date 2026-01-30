---
title: "【神業】Claude Code開発者直伝！知らないと損する隠れTips 20選"
emoji: "🔥"
type: "tech"
topics: ["claudecode", "ai", "開発効率化", "cli", "プログラミング"]
published: false
---

**Claude Codeを毎日使っているのに、半分も機能を使いこなせていない。**

これ、ほとんどの人に当てはまります。

今回は、Claude Code開発者のBoris Cherny氏が30日で**259個のPR、497コミット、4万行追加**を達成したテクニックと、コミュニティで発見された隠れTipsを徹底解説します。

## 結論から言うと...

:::message
Claude Codeは「チャットAI」ではない。**プログラマブルな開発環境**として使え。
:::

---

## 基本編：今すぐ使える必須Tips

### 1. `/clear` を頻繁に使え

```
/clear
```

新しいタスクを始めるたびに実行してください。

**なぜ？**
- 古い会話履歴がトークンを消費
- コンパクション処理で要約される際に情報が失われる
- フレッシュなコンテキストの方が**精度が高い**

:::message alert
長時間セッションの最大の敵は「コンテキスト汚染」です
:::

---

### 2. CLAUDE.mdを書け（必須）

プロジェクトルートに `CLAUDE.md` を作成：

```markdown
# プロジェクト概要
Next.js 14 + TypeScript + Prisma のECサイト

# よく使うコマンド
- `pnpm dev` - 開発サーバー起動
- `pnpm test` - テスト実行
- `pnpm db:push` - DBスキーマ反映

# コードスタイル
- コンポーネントは`src/components/`に配置
- APIは`src/app/api/`に配置
- 日本語コメント禁止（英語のみ）

# 注意事項
- `prisma/schema.prisma`を変更したら必ずdb:push
- 環境変数は`.env.local`に記載
```

Claude Codeは起動時に自動で読み込みます。

**配置場所の使い分け：**
| ファイル | 用途 |
|----------|------|
| `CLAUDE.md` | チーム共有（git管理） |
| `CLAUDE.local.md` | 個人設定（gitignore） |
| `~/.claude/CLAUDE.md` | 全プロジェクト共通 |

---

### 3. 「think」で深く考えさせる

```
このアーキテクチャについて think して改善案を出して
```

`think`、`think hard`、`think harder`、`ultrathink` で**拡張思考モード**が有効になります。

複雑な設計判断や、バグの根本原因調査に効果的。

---

### 4. Tabキー補完を活用

ファイルパスを入力する際、**Tabキー**で補完が効きます：

```
src/com[Tab] → src/components/
```

Claude Codeに特定のファイルを見せたいとき、パスを手打ちするより圧倒的に速い。

---

### 5. Escキーで中断・修正

| 操作 | 効果 |
|------|------|
| Esc 1回 | 実行中断 |
| Esc 2回 | 前のプロンプトを編集 |

間違った方向に進み始めたら、すぐにEscで軌道修正。

---

## 中級編：生産性を2倍にするTips

### 6. `/compact` で手動圧縮

```
/compact
```

会話が長くなったら、自動圧縮を待たずに**手動で圧縮**。重要な情報が失われる前に、自分のタイミングで実行できます。

---

### 7. HANDOFF.mdパターン

長いタスクの途中で `/clear` する前に：

```
現在の進捗をHANDOFF.mdに書き出して
```

次のセッションで：

```
HANDOFF.mdを読んで、続きをやって
```

**コンテキストのバトンタッチ**ができます。

---

### 8. チェックリスト駆動開発

大規模な変更には、まずチェックリストを作らせる：

```
このマイグレーションに必要なステップをMarkdownチェックリストで作って
```

出力例：
```markdown
- [ ] 古いスキーマのバックアップ
- [ ] 新しいカラム追加
- [ ] データ移行スクリプト作成
- [ ] 移行実行
- [ ] 古いカラム削除
- [ ] テスト実行
```

Claude Codeは各ステップを完了するたびにチェックを入れていきます。

---

### 9. スクリーンショットで指示

UIの修正は**言葉より画像**。

ドラッグ＆ドロップでスクリーンショットを貼り付け：

```
[スクリーンショット]
このボタンの位置を右に20px移動して
```

デザインモックとの比較も可能：

```
[デザインモック画像]
これと同じ見た目にして
```

2-3回のイテレーションで完璧に仕上がります。

---

### 10. GitHub CLI連携

```bash
gh auth login
```

認証後、Claude Codeで：

```
Issue #123の内容を確認して修正して
```

```
このPRにレビューコメントを追加して
```

```
CIが落ちてる原因を調べて修正して
```

**Issue → 修正 → PR → マージ**が会話だけで完結。

---

## 上級編：プロだけが知るテクニック

### 11. システムプロンプト削減で10kトークン節約

デフォルトのシステムプロンプトは**約10,000トークン**消費しています。

設定でカスタマイズすることで、実際の作業に使えるコンテキストが増えます。

---

### 12. MCP Tool Searchの活用

MCPサーバーが多い場合、**MCP Tool Search**が自動有効化：

```
# 従来: 50ツールで77kトークン消費
# Tool Search: 8.7kトークン（89%削減）
```

必要なツールだけが動的にロードされます。

---

### 13. カスタムスラッシュコマンド

`.claude/commands/` にMarkdownファイルを作成：

```markdown
<!-- .claude/commands/review.md -->
# コードレビュー

以下の観点でレビューしてください：
1. セキュリティリスク
2. パフォーマンス問題
3. 可読性
4. テストカバレッジ

対象: $ARGUMENTS
```

使用：
```
/project:review src/api/auth.ts
```

チーム共通のワークフローを標準化できます。

---

### 14. Gemini CLIフォールバック

Claude CodeのWebFetchはRedditなど一部サイトにアクセスできません。

**解決策：** Gemini CLIをフォールバックとして使う

```bash
# Gemini CLIインストール
npm install -g @anthropic-ai/gemini-cli
```

スキルを作成して、アクセスできないサイトはGeminiに取得させる。

---

### 15. コンテナ内でClaude Codeを実行

**最強のサンドボックス環境：**

```bash
# Dockerコンテナ起動
docker run -it --rm \
  -v $(pwd):/workspace \
  -e ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY \
  node:20 bash

# コンテナ内でClaude Code実行
npx claude-code
```

リスクのある実験や長時間タスクを**完全隔離**で実行できます。

ローカルのClaude Codeからコンテナ内のClaude Codeを**tmux経由で制御**することも可能。

---

### 16. Git Worktreeで並列作業

```bash
# 別ブランチを別ディレクトリにチェックアウト
git worktree add ../feature-a feature-a
git worktree add ../feature-b feature-b
```

複数のClaude Codeセッションで**異なる機能を同時開発**。マージコンフリクトなし。

---

### 17. ヘッドレスモードでCI統合

```bash
claude -p "テストを実行して結果を報告" --output-format stream-json
```

CI/CDパイプライン、pre-commitフック、スケジュールタスクに組み込み可能。

---

### 18. 並列検証パターン

```
1つ目のClaude Code: コードを書く
2つ目のClaude Code: レビュー・テスト
3つ目のClaude Code: フィードバックを統合
```

**複数のClaude Codeを協調させる**ことで、品質と速度を両立。

---

### 19. 音声入力の活用

[SuperWhisper](https://superwhisper.com/)や[MacWhisper](https://goodsnooze.gumroad.com/l/macwhisper)を使った音声入力：

- タイピングより**3倍速い**指示出し
- Claudeが文字起こしエラーを自動修正
- 手が疲れない

長時間のコーディングセッションに最適。

---

### 20. 指数バックオフで待機

長時間ジョブ（Docker build、CI）の監視：

```
このCIジョブを監視して。1分後、2分後、4分後...と
指数バックオフで確認して、完了したら教えて。
```

Claude Codeが**自動で待機・確認・報告**してくれます。

---

## Boris Cherny流ワークフロー

Claude Code開発者のBoris Cherny氏は、このワークフローで**30日で259PR**を達成：

```
1. Explore: 関連ファイルを読ませる（コードは書かない）
2. Plan: "think hard"で計画を立てさせる
3. Code: 明確な検証ステップ付きで実装
4. Commit: 変更をコミット、ドキュメント更新
```

:::message
**ポイント:** いきなりコードを書かせない。まず理解させ、計画を立てさせる。
:::

---

## まとめ

| レベル | 必須Tips |
|--------|----------|
| 初級 | `/clear`、CLAUDE.md、Escキー |
| 中級 | HANDOFF.md、チェックリスト、GitHub CLI |
| 上級 | コンテナ実行、Git Worktree、並列検証 |

Claude Codeは「AIチャット」ではなく**開発環境**です。

プログラマブルに使いこなせば、生産性は**確実に2-3倍**になります。

---

## 参考リンク

- [Claude Code Tips (40+ tips)](https://github.com/ykdojo/claude-code-tips)
- [Anthropic公式 Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Boris Cherny's 22 Tips](https://medium.com/@joe.njenga/boris-cherny-claude-code-creator-shares-these-22-tips-youre-probably-using-it-wrong-1b570aedefbe)

---

**あなたのお気に入りのClaude Code Tipsは何ですか？コメントで教えてください！**
