---
title: "【保存版】Claude Code Skillsマーケットプレイス爆誕！9万6000個のスキルを無料で使う方法"
emoji: "🚀"
type: "tech"
topics: ["ai", "claudecode", "aiagent", "skills", "productivity"]
published: false
---

**「え、まだ素のClaude Code使ってるの？」**

これ、最近の開発者コミュニティでよく聞く言葉です。

2026年1月現在、**Claude Code Skillsのマーケットプレイスが96,751個を突破**。1ヶ月前は60,000個だったのが、たった数週間で1.6倍に膨れ上がっています。

:::message alert
衝撃の事実: Anthropic、Google Labs、Vercel、Stripe、Cloudflare、Hugging Faceなど**大手テック企業が公式スキルを公開中**
:::

## 結論から言うと

- **96,751個以上**のスキルが無料で使える
- **5分でセットアップ完了** - コピペでOK
- **OpenAI Codex CLI、Cursor、GitHub Copilotとも互換**
- 「スキルを作るスキル」で**自作も超簡単**

この記事を読めば、今日から生産性が**80%向上**します（大げさじゃないです）。

---

## そもそもClaude Code Skillsって何？

### 一言で言うと「Claude専用プラグイン」

Skillsとは、Claudeに**特定のタスクを教えるための説明書**です。

```
従来のClaude Code:
「PRをレビューして」→ 一般的なレビューコメント

Skills導入後:
「PRをレビューして」→ 御社のコーディング規約に沿った
                      具体的な指摘 + 自動修正案
```

### 技術的には超シンプル

```
my-skill/
├── SKILL.md      ← Claudeへの指示書（必須）
├── scripts/      ← 実行スクリプト（任意）
└── references/   ← 参考資料（任意）
```

たったこれだけ。`SKILL.md`はMarkdownファイルなので、**プログラミング知識ゼロ**でも作れます。

---

## 今すぐ使える！3大マーケットプレイス

### 1. SkillsMP（96,751個以上）

**最大規模のコミュニティマーケットプレイス**

- 公式サイト: skillsmp.com
- GitHubから自動収集
- 最低2スター以上のリポジトリのみ
- カテゴリ・人気順でフィルタリング可能

### 2. awesome-agent-skills（VoltAgent）

**公式チームのスキルが集結**

- GitHub: github.com/VoltAgent/awesome-agent-skills
- Anthropic、Google Labs、Vercelなど大手が公開
- 200以上の厳選スキル
- Claude Code、Codex、Cursor、GitHub Copilot対応

### 3. Anthropic公式（anthropics/skills）

**最も信頼性が高い**

- GitHub: github.com/anthropics/skills
- Anthropicが直接メンテナンス
- セキュリティ監査済み

---

## 5分で始める！インストール手順

### ステップ1: マーケットプレイスを追加

```bash
# Claude Codeを開いて以下を実行
/plugin marketplace add anthropics/skills
```

### ステップ2: スキルをインストール

```bash
# 公式スキルセットを一括インストール
/plugin install example-skills@anthropic-agent-skills
```

### ステップ3: 確認

```bash
# インストール済みスキルを確認
/skills
```

これだけ！Claude Codeを再起動すれば使えます。

### 手動インストールの場合

```bash
# 個人用（全プロジェクトで使える）
~/.claude/skills/my-skill/

# プロジェクト専用（チームで共有）
.claude/skills/my-skill/
```

---

## 絶対入れるべき神スキル10選

### 公式（Anthropic）

| スキル名 | 用途 | おすすめ度 |
|---------|------|-----------|
| `skill-creator` | スキルを作るスキル | ⭐⭐⭐⭐⭐ |
| `mcp-builder` | MCPサーバー構築 | ⭐⭐⭐⭐⭐ |
| `web-artifacts-builder` | React+Tailwind生成 | ⭐⭐⭐⭐ |
| `webapp-testing` | Playwright自動テスト | ⭐⭐⭐⭐ |

### Hugging Face公式

| スキル名 | 用途 | おすすめ度 |
|---------|------|-----------|
| `hugging-face-model-trainer` | TRLでモデル訓練 | ⭐⭐⭐⭐⭐ |
| `hugging-face-datasets` | データセット管理 | ⭐⭐⭐⭐ |
| `hugging-face-evaluation` | モデル評価 | ⭐⭐⭐⭐ |

### コミュニティ人気

| スキル名 | 用途 | おすすめ度 |
|---------|------|-----------|
| `NanoBanana-PPT-Skills` | AI PPT生成 | ⭐⭐⭐⭐⭐ |
| `frontend-slides` | アニメーション付きスライド | ⭐⭐⭐⭐ |
| `rootly-incident-responder` | インシデント対応 | ⭐⭐⭐⭐ |

---

## 自分でスキルを作る方法

### 最も簡単な方法: skill-creatorを使う

```
あなた: 「PRレビューのスキルを作って。うちのコーディング規約に従って
        チェックしてほしい」

Claude: 「skill-creatorを使ってスキルを作成します...」
        → SKILL.mdが自動生成される
```

**たったこれだけ。** プログラミング不要です。

### SKILL.mdの構造

```yaml
---
name: my-pr-reviewer
description: PRをレビューし、コーディング規約違反を指摘する
---

# PRレビュースキル

## いつ使うか
- PRレビューを依頼されたとき
- コードの品質チェックを求められたとき

## 確認項目
1. 命名規則（camelCase使用）
2. 関数の行数（50行以下）
3. コメントの有無
4. テストカバレッジ

## 出力形式
- 問題点を箇条書きで列挙
- 修正案を具体的に提示
```

### ポイント: descriptionが命

```yaml
# ❌ 悪い例
description: PRレビューする

# ✅ 良い例
description: PRレビューを実行し、コーディング規約違反・セキュリティ
             脆弱性・パフォーマンス問題を検出して修正案を提示する
```

Claudeは`description`を見て**100以上のスキルから最適なものを選択**します。具体的に書くほど、正しく呼び出されます。

---

## 知っておくべき仕組み

### 遅延読み込み（超重要）

```
起動時:
  └─ 全スキルの name と description だけ読み込み（軽い）

ユーザーリクエスト受信:
  └─ description を見てマッチするスキルを判断

スキル選択時:
  └─ SKILL.md の本文を読み込み

必要に応じて:
  └─ scripts/ や references/ を読み込み
```

だから**100個以上のスキル**を入れても、パフォーマンスに影響しません。

### 複数スキルの同時使用

```
あなた: 「このPRをレビューして、問題があれば修正してコミットして」

Claude:
  1. pr-reviewer スキルでレビュー
  2. code-fixer スキルで修正
  3. git-automation スキルでコミット

→ 3つのスキルが連携して動作
```

---

## セキュリティ上の注意

:::message warn
⚠️ コミュニティスキルはセキュリティ監査されていません
:::

### 安全に使うためのチェックリスト

1. **インストール前にコードを読む**
2. **スター数が低いリポジトリは避ける**（最低10以上推奨）
3. **scriptsフォルダの中身を確認**
4. **本番環境では公式スキルのみ使用**

```bash
# インストール前に中身を確認
cat ~/.claude/skills/suspicious-skill/SKILL.md
cat ~/.claude/skills/suspicious-skill/scripts/*.sh
```

---

## 互換性: 他ツールでも使える！

### 対応ツール一覧

| ツール | 対応状況 |
|--------|---------|
| Claude Code | ✅ フル対応 |
| OpenAI Codex CLI | ✅ フル対応 |
| ChatGPT | ✅ フル対応 |
| Cursor | ✅ フル対応 |
| GitHub Copilot | ✅ フル対応 |
| Gemini CLI | ✅ フル対応 |
| Windsurf | ✅ フル対応 |

2024年12月にAnthropicがAgent Skills仕様を**オープンスタンダード化**。OpenAIもすぐに採用し、業界標準になりました。

---

## 実践例: 私の開発ワークフロー

```
朝のルーティン:
1. /daily-standup → 昨日のコミットから日報を自動生成
2. /pr-review → 溜まっているPRを一括レビュー
3. /issue-triage → 新規Issueを優先度分類

コーディング中:
4. /test-gen → 書いたコードのテストを自動生成
5. /doc-gen → 関数のドキュメントを自動生成
6. /security-check → セキュリティ脆弱性をスキャン

終業時:
7. /commit-push-pr → コミット、プッシュ、PR作成を一括実行
```

**これ全部、スキルで自動化しています。**

---

## よくある質問

### Q: 有料ですか？

**A: すべて無料です。** SkillsMPもawesome-agent-skillsもオープンソース。

### Q: Windows/Mac/Linux対応していますか？

**A: すべて対応しています。** Claude Codeが動く環境なら使えます。

### Q: スキルが多すぎて選べません

**A: まずは以下の3つから始めてください:**
1. `skill-creator` - スキルを作るスキル
2. `mcp-builder` - MCPサーバー構築
3. お好みの1つ（PRレビュー、テスト生成など）

### Q: 社内の機密コードに使っても大丈夫？

**A: スキル自体はローカルで動作します。** ただし、Claude APIを通じてコードが送信されることに変わりはありません。機密性の高いプロジェクトでは、Anthropicの利用規約とセキュリティポリシーを確認してください。

---

## まとめ

:::message
**今日のアクション:**
1. `/plugin marketplace add anthropics/skills` を実行
2. `/plugin install example-skills@anthropic-agent-skills` でインストール
3. `/skills` で確認
4. `skill-creator` で自分専用スキルを作る
:::

### 覚えておくべき数字

| 項目 | 数値 |
|------|------|
| SkillsMP総スキル数 | 96,751個以上 |
| 1ヶ月前からの増加 | 1.6倍 |
| セットアップ時間 | 5分 |
| 対応ツール | 7種類以上 |

### 次のステップ

1. **今日**: 公式スキルをインストール
2. **今週中**: 自分の業務に合わせたスキルを1つ作成
3. **今月中**: チームで共有してワークフローを統一

---

## 参考リンク

- [Claude Code Skills公式ドキュメント](https://code.claude.com/docs/ja/skills)
- [SkillsMP - Agent Skills Marketplace](https://skillsmp.com/)
- [VoltAgent/awesome-agent-skills](https://github.com/VoltAgent/awesome-agent-skills)
- [Anthropic公式Skills](https://github.com/anthropics/skills)
- [Agent Skillsを一番かんたんに作る方法](https://zenn.dev/aun_phonogram/articles/475f3cca8f40a3)

---

**96,751個のスキル、使わないのはもったいなくないですか？**

**この記事が参考になったら、いいねと保存をお願いします！** 🙏

**あなたのお気に入りのスキルは何ですか？コメントで教えてください！**
