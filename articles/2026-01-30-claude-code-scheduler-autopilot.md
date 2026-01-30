---
title: "【神機能】Claude Codeを自動運転！Schedulerで寝てる間にコードレビューさせる方法"
emoji: "⏰"
type: "tech"
topics: ["Claude", "ClaudeCode", "自動化", "開発効率化", "AI"]
published: true
---

## あなたがコーヒーを飲んでる間に、Claudeが仕事してたら最高じゃないですか？

**結論から言うと、Claude Codeをcronのようにスケジュール実行できるようになりました。**

毎朝9時に自動コードレビュー、毎週月曜にセキュリティ監査、毎週金曜にTODOコメントの整理...

**全部、寝てる間にClaudeがやってくれます。**

## 🤖 Claude Code Schedulerとは？

[claude-code-scheduler](https://github.com/jshchnz/claude-code-scheduler)は、Claude Code公式プラグインマーケットプレイスで提供されているスケジューラープラグインです。

```
"Put Claude on autopilot."
「Claudeをオートパイロットに。」
```

OSのネイティブスケジューラー（macOS: launchd、Linux: crontab、Windows: Task Scheduler）を使って、Claude Codeタスクを自動実行します。

## 🔥 何がすごいのか？

### 1. 自然言語でスケジュール設定

cronの複雑な書式を覚える必要なし！

```
❌ 従来: 0 9 * * 1-5 /usr/local/bin/claude ...
✅ 今: "every weekday at 9am"
```

**「毎週月曜の10時」「毎日17時」と日本語感覚で指定できます。**

### 2. 完全自律モード

`--dangerously-skip-permissions`フラグで、Claudeが：
- ファイルを編集
- コマンドを実行
- **コミットまで作成**

人間の介入なしで動きます。

### 3. Git Worktree隔離

タスクは隔離されたブランチで実行され、自動でプッシュ。レビュー用のPRが勝手に作られる設計です。

## 📦 インストール方法

Claude Code v1.0.33以上が必要です。

```bash
# プラグインマーケットプレイスから追加
/plugin marketplace add jshchnz/claude-code-scheduler

# インストール
/plugin install scheduler@claude-code-scheduler
```

たった2コマンドで完了！

## 🎯 実践的な使い方5選

### 1. 毎朝の自動コードレビュー

```bash
/scheduler:schedule-add "every weekday at 9am review yesterday's commits and create a summary"
```

出社したらレビューサマリーが待ってる生活、最高では？

### 2. 週次セキュリティ監査

```bash
/scheduler:schedule-add "every Monday at 10am check for CVEs in dependencies and hardcoded secrets"
```

セキュリティチェックを忘れる心配がなくなります。

### 3. 技術的負債の追跡

```bash
/scheduler:schedule-add "every Friday at 2pm find all TODO and FIXME comments, create a report"
```

放置されたTODOを可視化して、チームに共有。

### 4. 依存パッケージの更新チェック

```bash
/scheduler:schedule-add "every Thursday at 10am list outdated npm packages and suggest updates"
```

`npm outdated`を手動で叩く時代は終わりました。

### 5. ドキュメント自動生成

```bash
/scheduler:schedule-add "every Sunday at 20pm update README based on recent code changes"
```

READMEが常に最新状態に保たれます。

## 🛠️ 主要コマンド一覧

| コマンド | 説明 |
|:--------|:-----|
| `/scheduler:schedule-add` | 新しいタスクを作成 |
| `/scheduler:schedule-list` | 全タスクを表示 |
| `/scheduler:schedule-remove` | タスクを削除 |
| `/scheduler:schedule-logs` | 実行履歴を表示 |
| `/scheduler:schedule-status` | スケジューラーの状態確認 |

## 📁 設定ファイルの場所

タスクはJSON形式で保存されます：

- **プロジェクト単位**: `.claude/schedules.json`
- **グローバル**: `~/.claude/schedules.json`

```json
{
  "tasks": [
    {
      "id": "daily-review",
      "schedule": "0 9 * * 1-5",
      "prompt": "review yesterday's commits",
      "autonomous": true,
      "worktree": true
    }
  ]
}
```

## 🔒 セキュリティ上の注意点

:::message alert
`--dangerously-skip-permissions`は強力ですが、名前の通り危険も伴います。
:::

推奨設定：
1. **Git Worktree隔離を必ず有効に** - mainブランチを直接変更させない
2. **PRレビューを挟む** - 自動コミットは必ず人間がレビュー
3. **機密情報へのアクセス制限** - `.env`などは`.claudeignore`に追加

## 🆚 他のスケジューラーとの比較

| ツール | 特徴 | おすすめ度 |
|:------|:-----|:--------:|
| **claude-code-scheduler** | 公式プラグイン、自然言語対応 | ⭐⭐⭐⭐⭐ |
| [runCLAUDErun](https://runclauderun.com/) | macOS専用ネイティブアプリ | ⭐⭐⭐⭐ |
| [claude-tasks](https://github.com/kylemclaren/claude-tasks) | TUI付き、Discord/Slack通知対応 | ⭐⭐⭐⭐ |
| GitHub Actions | CI/CD統合、Secret管理が楽 | ⭐⭐⭐ |

## 💡 応用アイデア

### CI/CDパイプラインとの連携

```yaml
# .github/workflows/claude-review.yml
name: Claude Code Review
on:
  schedule:
    - cron: '0 9 * * 1-5'
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Claude Code
        run: |
          claude --print "Review changes since last review" \
            --allowedTools "Bash,Read,Write" \
            --output review.md
```

### Slack通知との組み合わせ

```bash
/scheduler:schedule-add "every day at 17pm summarize today's work and post to slack webhook"
```

日報が自動で生成されてSlackに投稿される未来。

## 🚀 今すぐ始める3ステップ

1. **Claude Codeを最新版に更新**
   ```bash
   claude upgrade
   ```

2. **プラグインをインストール**
   ```bash
   /plugin marketplace add jshchnz/claude-code-scheduler
   /plugin install scheduler@claude-code-scheduler
   ```

3. **最初のタスクを登録**
   ```bash
   /scheduler:schedule-add "every weekday at 9am say good morning and list today's tasks"
   ```

## まとめ

- **claude-code-scheduler**でClaude Codeをスケジュール実行可能
- **自然言語**でスケジュール設定（cron不要）
- **自律モード**でファイル編集・コミットまで自動化
- **Git Worktree隔離**で安全に運用
- macOS、Linux、Windows全対応

**開発者の仕事は「Claudeに何をさせるか考えること」にシフトしています。**

あなたは寝ている間に、Claudeが働いてくれる時代です。

---

:::message
この記事が参考になったら**いいね**と**ストック**をお願いします！
「こんなタスクを自動化してみた」という方は、ぜひコメントで教えてください！
:::

## 参考リンク

- [claude-code-scheduler (GitHub)](https://github.com/jshchnz/claude-code-scheduler)
- [runCLAUDErun (macOS App)](https://runclauderun.com/)
- [claude-tasks (GitHub)](https://github.com/kylemclaren/claude-tasks)
- [Claude Code公式ドキュメント](https://docs.anthropic.com/claude/claude-code)
