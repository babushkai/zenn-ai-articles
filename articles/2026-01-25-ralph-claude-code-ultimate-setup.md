---
title: "【保存版】RalphとClaude Codeで「完全自律開発環境」を構築する完全ガイド"
emoji: "⚡"
type: "tech"
topics: ["claudecode", "ai", "自動化", "開発環境", "エージェント"]
published: false
---

**「毎晩寝る前にRalphを起動して、朝起きたら新機能が3つ増えてる」**

これが私の日常になりました。

前回の記事でRalphを紹介しましたが、今回は**実際の環境構築から運用まで**、すべてを公開します。

:::message
この記事を読み終わる頃には、あなたも「AIに開発を丸投げできる環境」が手に入ります。
:::

## 🎯 この記事で構築する環境

```
┌─────────────────────────────────────────────────┐
│                    Ralph                         │
│  ┌─────────────────────────────────────────┐    │
│  │         Claude Code + Hooks              │    │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐ │    │
│  │  │ Pre-run │→│ Execute │→│Post-run │ │    │
│  │  │ Hooks   │  │  Task   │  │ Hooks   │ │    │
│  │  └─────────┘  └─────────┘  └─────────┘ │    │
│  └─────────────────────────────────────────┘    │
│                      ↓                           │
│  ┌─────────────────────────────────────────┐    │
│  │    Git Auto-commit + Progress Track      │    │
│  └─────────────────────────────────────────┘    │
│                      ↓                           │
│              次のタスクへループ                   │
└─────────────────────────────────────────────────┘
```

**完成形：**
- PRDを書いて寝るだけ
- Claude Codeが自動で実装
- Hooksで品質を自動担保
- 朝起きたら全タスク完了

## 📦 Step 1: 必要なツールのインストール

### Claude Codeのインストール

```bash
# npmでインストール
npm install -g @anthropic-ai/claude-code

# または直接インストール
curl -fsSL https://claude.ai/install.sh | sh

# バージョン確認
claude --version
# v2.1.0以上推奨
```

### Ralphのインストール

```bash
# リポジトリをクローン
git clone https://github.com/snarktank/ralph.git
cd ralph

# PATHに追加（~/.zshrcか~/.bashrcに追加）
export PATH="$PATH:$(pwd)"

# または専用版を使う
git clone https://github.com/frankbria/ralph-claude-code.git
cd ralph-claude-code
./install.sh
```

### 依存ツール

```bash
# macOS
brew install jq

# Ubuntu/Debian
sudo apt-get install jq
```

## 🔧 Step 2: CLAUDE.mdの最適化

プロジェクトルートに`CLAUDE.md`を作成。これがClaude Codeの「脳」になります。

```markdown
# CLAUDE.md

## プロジェクト概要
[プロジェクト名] - [1行説明]

## 技術スタック
- 言語: TypeScript 5.x
- フレームワーク: Next.js 15
- DB: PostgreSQL + Prisma
- テスト: Vitest

## アーキテクチャ
```
src/
├── app/          # Next.js App Router
├── components/   # UIコンポーネント
├── lib/          # ユーティリティ
├── services/     # ビジネスロジック
└── types/        # 型定義
```

## 開発ルール

### 必須
- 全ての関数に型を付ける
- 新機能には必ずテストを書く
- コミットメッセージはConventional Commits

### 禁止
- any型の使用
- console.logの本番コード混入
- 未使用importの放置

## コマンド
- `npm run dev` - 開発サーバー
- `npm run test` - テスト実行
- `npm run lint` - Lint実行
- `npm run typecheck` - 型チェック

## 過去の学習

### 2026-01-25
- Prismaマイグレーションは`npx prisma migrate dev`で実行
- 環境変数は`.env.local`に追加後、`.env.example`にも追記

### 2026-01-24
- Next.js 15ではServer Actionsを優先使用
- Zustandの状態はservices/以下で管理
```

:::message alert
**重要**: CLAUDE.mdは定期的に更新しましょう。Ralphが学習した内容を蓄積することで、AIの精度が上がります。
:::

## 🎛️ Step 3: Hooksの設定

Claude Codeのhooksで、品質を自動担保します。

### hooks設定ファイルの作成

```bash
# .claude/settings.jsonを作成
mkdir -p .claude
```

```json
// .claude/settings.json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "npm run lint:fix -- $CLAUDE_FILE_PATH 2>/dev/null || true"
      },
      {
        "matcher": "Write",
        "command": "echo \"[$(date)] File written: $CLAUDE_FILE_PATH\" >> .claude/activity.log"
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "command": "echo \"[$(date)] Command: $CLAUDE_TOOL_INPUT\" >> .claude/commands.log"
      }
    ],
    "Stop": [
      {
        "command": "npm run typecheck 2>&1 | tail -20"
      },
      {
        "command": "npm run test 2>&1 | tail -30"
      }
    ]
  }
}
```

### Hooksの解説

| Hook | タイミング | 用途 |
|------|-----------|------|
| `PreToolUse` | ツール実行前 | コマンドログ、バックアップ |
| `PostToolUse` | ツール実行後 | 自動フォーマット、Lint |
| `Stop` | 応答完了時 | テスト実行、型チェック |
| `SessionEnd` | セッション終了 | ハンドオフ生成 |

### 高度なHooks例

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "command": "prettier --write $CLAUDE_FILE_PATH 2>/dev/null || true",
        "description": "Auto-format on file write"
      },
      {
        "matcher": "Edit|Write",
        "command": "git diff --stat $CLAUDE_FILE_PATH | head -5",
        "description": "Show changes"
      }
    ],
    "Stop": [
      {
        "command": "npm run test -- --reporter=dot --silent 2>&1 | tail -10",
        "timeout": 120000,
        "description": "Run tests after each response"
      }
    ],
    "SessionEnd": [
      {
        "command": "echo \"## Session $(date)\" >> HANDOFF.md && git diff --stat >> HANDOFF.md",
        "description": "Generate handoff on session end"
      }
    ]
  }
}
```

## 📋 Step 4: PRDの作成

### prd.jsonテンプレート

```json
{
  "project": "my-awesome-app",
  "version": "1.0.0",
  "stories": [
    {
      "id": "auth-001",
      "title": "ユーザー登録APIの実装",
      "priority": 1,
      "status": "incomplete",
      "acceptance_criteria": [
        "POST /api/auth/register でユーザー登録できる",
        "emailとpasswordを受け取る",
        "既存emailは409エラーを返す",
        "パスワードはbcryptでハッシュ化",
        "テストカバレッジ80%以上"
      ],
      "technical_notes": "Prismaのuserモデルを使用"
    },
    {
      "id": "auth-002",
      "title": "ログインAPIの実装",
      "priority": 2,
      "status": "incomplete",
      "depends_on": ["auth-001"],
      "acceptance_criteria": [
        "POST /api/auth/login でログインできる",
        "JWTトークンを発行する",
        "無効な認証情報は401エラー",
        "テストカバレッジ80%以上"
      ]
    },
    {
      "id": "auth-003",
      "title": "ログアウトAPIの実装",
      "priority": 3,
      "status": "incomplete",
      "depends_on": ["auth-002"],
      "acceptance_criteria": [
        "POST /api/auth/logout でログアウトできる",
        "トークンを無効化する"
      ]
    }
  ],
  "constraints": {
    "no_breaking_changes": true,
    "maintain_backwards_compatibility": true
  }
}
```

### PRD作成のコツ

```
✅ 良いタスク（原子的）
- "usersテーブルにemail_verified列を追加"
- "User.findByEmailメソッドを実装"
- "ログインAPIのレートリミットを追加"

❌ 悪いタスク（大きすぎる）
- "認証システムを実装"
- "ユーザー管理機能を作る"
- "セキュリティを強化"
```

## 🚀 Step 5: Ralphの起動

### 基本起動

```bash
# プロジェクトディレクトリで実行
cd my-project
ralph
```

### オプション付き起動

```bash
# 最大イテレーション数を指定（コスト制御）
ralph --max-iterations 20

# 特定のPRDファイルを指定
ralph --prd features/auth.json

# デバッグモード
ralph --verbose

# ドライラン（実行せずに確認）
ralph --dry-run
```

### 監視モード

```bash
# 別ターミナルで進捗監視
ralph-monitor

# または手動で
watch -n 5 'cat progress.txt | tail -20'
```

## 📊 Step 6: 進捗管理

### progress.txtの確認

```bash
cat progress.txt
```

```markdown
# Progress Log

## Iteration 1 - 2026-01-25 01:30:00
- Task: auth-001 (ユーザー登録APIの実装)
- Status: COMPLETED
- Files changed: 4
- Tests: 12 passed

## Iteration 2 - 2026-01-25 02:15:00
- Task: auth-002 (ログインAPIの実装)
- Status: COMPLETED
- Files changed: 3
- Tests: 18 passed

## Iteration 3 - 2026-01-25 03:00:00
- Task: auth-003 (ログアウトAPIの実装)
- Status: IN_PROGRESS
```

### Git履歴の確認

```bash
git log --oneline -10

# 出力例:
# a1b2c3d feat(auth): implement logout API
# d4e5f6g feat(auth): implement login API with JWT
# g7h8i9j feat(auth): implement user registration
# ...
```

## ⚠️ Step 7: セーフティ設定

### コスト制御

```bash
# .ralph.configに設定
cat > .ralph.config << 'EOF'
MAX_ITERATIONS=50
RATE_LIMIT_PER_HOUR=100
MAX_COST_USD=100
CIRCUIT_BREAKER_THRESHOLD=5
EOF
```

### 自動停止条件

```json
// ralph-config.json
{
  "exit_conditions": {
    "all_tasks_complete": true,
    "max_iterations": 50,
    "max_errors_consecutive": 3,
    "max_cost_usd": 100
  },
  "safety": {
    "require_tests_pass": true,
    "require_typecheck_pass": true,
    "no_force_push": true,
    "protected_files": [
      ".env",
      ".env.local",
      "package-lock.json"
    ]
  }
}
```

## 🔄 Step 8: 自動化（CI/CD統合）

### GitHub Actionsで毎晩実行

```yaml
# .github/workflows/ralph-nightly.yml
name: Ralph Nightly Development

on:
  schedule:
    - cron: '0 0 * * *'  # 毎日UTC 0:00（JST 9:00）
  workflow_dispatch:      # 手動実行も可能

jobs:
  ralph:
    runs-on: ubuntu-latest
    timeout-minutes: 360  # 6時間上限

    steps:
      - uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Install Ralph
        run: |
          git clone https://github.com/snarktank/ralph.git /tmp/ralph
          export PATH="$PATH:/tmp/ralph"

      - name: Run Ralph
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          ralph --max-iterations 30

      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v5
        with:
          title: "[Ralph] Automated Development - $(date +%Y-%m-%d)"
          body: |
            ## 🤖 Ralph Automated Development

            This PR was automatically generated by Ralph.

            ### Changes
            $(cat progress.txt | tail -50)
          branch: ralph/auto-dev-${{ github.run_number }}
          commit-message: "feat: automated development by Ralph"
```

### ローカルでのスケジュール実行（macOS）

```bash
# launchdで毎晩1時に実行
cat > ~/Library/LaunchAgents/com.ralph.nightly.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.ralph.nightly</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>-c</string>
        <string>cd /path/to/your/project && ralph --max-iterations 30</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>1</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>
    <key>StandardOutPath</key>
    <string>/tmp/ralph-nightly.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/ralph-nightly.err</string>
</dict>
</plist>
EOF

launchctl load ~/Library/LaunchAgents/com.ralph.nightly.plist
```

## 💰 コスト試算

### 実際の運用コスト例

| プロジェクト規模 | イテレーション数 | 概算コスト/日 |
|-----------------|-----------------|--------------|
| 小規模（5タスク） | 10-20 | $5-15 |
| 中規模（15タスク） | 30-50 | $25-50 |
| 大規模（30タスク+） | 50-100 | $75-150 |

### コスト最適化Tips

1. **タスクを小さく分割** → 少ないトークンで完了
2. **CLAUDE.mdを充実** → 無駄な試行錯誤が減る
3. **テストを先に書く** → 方向性の迷いが減る
4. **max-iterationsを設定** → 暴走防止

## 🎓 実践例：認証システムを一晩で構築

### PRD

```json
{
  "stories": [
    {"id": "1", "title": "Prismaスキーマにuserモデル追加", "priority": 1},
    {"id": "2", "title": "ユーザー登録API実装", "priority": 2},
    {"id": "3", "title": "ログインAPI実装", "priority": 3},
    {"id": "4", "title": "JWTミドルウェア実装", "priority": 4},
    {"id": "5", "title": "保護されたAPIルート作成", "priority": 5},
    {"id": "6", "title": "ログアウトAPI実装", "priority": 6},
    {"id": "7", "title": "パスワードリセット機能", "priority": 7},
    {"id": "8", "title": "メール認証機能", "priority": 8}
  ]
}
```

### 結果

```
🕐 開始: 23:00
🕗 終了: 06:30 (7.5時間)

📊 統計:
- イテレーション: 24回
- コミット: 24個
- 新規ファイル: 18
- テスト: 47個（全パス）
- コスト: 約$35

✅ 完了タスク: 8/8 (100%)
```

**寝てる間に認証システム完成。**

## 🔮 まとめ：これからの開発フロー

```
従来: 要件定義 → 設計 → 実装 → テスト → デプロイ
              ↑        ↑      ↑
            人間     人間    人間

これから: 要件定義 → 設計 → 実装 → テスト → デプロイ
              ↑        ↑      ↑
            人間    Ralph   Ralph
```

**人間の仕事は「何を作るか決める」こと。**
**作るのはAIに任せる時代。**

---

## 🔗 リンク

- **Ralph**: https://github.com/snarktank/ralph
- **Ralph for Claude Code**: https://github.com/frankbria/ralph-claude-code
- **Claude Code Hooks Guide**: https://code.claude.com/docs/en/hooks-guide
- **Claude Code Best Practices**: https://www.anthropic.com/engineering/claude-code-best-practices

---

## 🙏 最後に

この記事が参考になったら、**いいねと保存**をお願いします！

**質問**: この環境、実際に構築してみた人いますか？どんなプロジェクトでRalphを活用していますか？コメントで教えてください！

**次回予告**: 「RalphをDockerで完全隔離して安全に運用する方法」を書く予定です。お楽しみに！
