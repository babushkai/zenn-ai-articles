---
title: "【革命】GitHubが禁断の一手！Claude・Codex・Copilotを同時に使える「Agent HQ」で開発が完全に変わる"
emoji: "🚀"
type: "tech"
topics: ["github", "claude", "openai", "copilot", "ai"]
published: true
---

**「え、ライバル同士のAIを一つの画面で使えるの？」**

2026年2月4日、GitHubが業界を震撼させる発表をしました。

**AnthropicのClaude**と**OpenAIのCodex**が、GitHubのCopilotと同じプラットフォームで使えるようになったんです。

これ、どれだけヤバいことか分かりますか？

## 結論から言うと...

GitHubの「Agent HQ」により、**Copilot Pro+またはEnterprise契約者は追加料金なしで3つのAIを使い分けられる**ようになりました。

- 同じタスクを3つのAIに投げて、ベストな回答を選べる
- プルリクエストやIssueに直接AIをアサインできる
- 全ての作業履歴がGitHubに残る

**もう「どのAIツールを契約するか」で悩む必要がなくなりました。**

## なぜこれが「禁断の一手」なのか？

普通に考えてください。

OpenAIとAnthropicは**ガチのライバル**です。Sam AltmanとDario Amodiは、かつて同じ会社にいて袂を分かった間柄。

そのライバル同士のAIを、GitHubが**同じUI上で比較させる**プラットフォームを作ったんです。

これが意味することは：

:::message
**AIエージェントは「コモディティ化」した**

もはやどのAIを使うかは重要ではなく、**どう使い分けるか**が重要な時代に突入しました。
:::

## Agent HQで何ができるのか？

### 1. エージェントの選択と実行

```
@Copilot このバグを直して
@Claude アーキテクチャをレビューして
@Codex テストを書いて
```

GitHubのコメント欄で直接メンションすることで、各AIエージェントにタスクを振れます。

### 2. 並列実行で最適解を探す

同じタスクを複数のエージェントに投げて、**アプローチの違いを比較**できます。

| エージェント | 得意なこと |
|-------------|-----------|
| Copilot | GitHub固有の文脈理解、PR作成 |
| Claude | 長いコードのレビュー、アーキテクチャ設計 |
| Codex | 高速なコード生成、テスト作成 |

### 3. フルな監査ログ

エンタープライズユーザーは、どのエージェントが何をしたか全て追跡できます。セキュリティ的にも安心。

## 実際の使い方（5ステップ）

### ステップ1: リポジトリ設定でエージェントを有効化

リポジトリのSettingsから、ClaudeとCodexをそれぞれ有効化します。

### ステップ2: Agentsタブを開く

GitHubのリポジトリページに新しく「Agents」タブが追加されています。

### ステップ3: タスクを記述

```markdown
# タスク
認証機能にRate Limitingを追加してください。
Redis を使い、IPベースで1分間に60リクエストまでに制限。
```

### ステップ4: エージェントを選択

Claude、Codex、Copilotから選択。または**複数選択**して並列実行も可能。

### ステップ5: 結果をレビュー

各エージェントは**ドラフトPR**として提出するので、人間がレビューしてからマージできます。

:::message alert
**直接コミットはしません！**
エージェントは必ずドラフトPRを作成するので、レビューなしでコードが本番に入ることはありません。
:::

## 料金体系

**追加料金は一切かかりません。**

Copilot Pro+またはCopilot Enterpriseの契約があれば、ClaudeもCodexも追加なしで使えます。

ただし、各エージェントセッションは「プレミアムリクエスト」を1回消費します。

## 開発者の反応

Anthropicのプラットフォーム責任者Katelyn Lesse氏：

> "ClaudeがコードをコミットしてPRにコメントできるようになったことで、チームはより速くイテレーションして出荷できる"

OpenAIのAlexander Embiricos氏：

> "数百万の開発者が、メインのワークスペースで直接Codexを使えるようになる"

## 今後の展開

GitHubは以下のAIプロバイダーとも協議中：

- **Google** (Gemini)
- **Cognition** (Devin)
- **xAI** (Grok)

近い将来、さらに多くのAIエージェントがAgent HQに参加する可能性があります。

## これが意味する未来

正直、この発表を見て思ったのは：

**「AIツール戦争」は終わった**

ということです。

CursorかWindsurfか、Claude CodeかCodexか...そんな議論はもはや意味がない。

これからは**全てのAIを適材適所で使い分ける時代**です。

GitHubがその「ハブ」になったことで、開発者は：

1. ツールの乗り換えコストから解放される
2. 複数AIの強みを組み合わせられる
3. ベンダーロックインのリスクが減る

## まとめ

- GitHub Agent HQでClaude・Codex・Copilotが統合
- Copilot Pro+/Enterprise契約者は追加料金なし
- 複数AIを並列実行して比較可能
- 全作業はドラフトPRとして人間がレビュー
- Google、Cognition、xAIも参加予定

**2026年は「マルチエージェント」の年です。**

一つのAIに依存するのではなく、複数のAIを使いこなすスキルが求められる時代になりました。

---

この記事が参考になったら、**いいねとストック**をお願いします！

**質問：あなたはClaude派？Codex派？それともCopilot派？** コメントで教えてください！

## 参考リンク

Pick your agent: Use Claude and Codex on Agent HQ - GitHub Blog

https://github.blog/news-insights/company-news/pick-your-agent-use-claude-and-codex-on-agent-hq/

Claude and Codex are now available in public preview on GitHub - GitHub Changelog

https://github.blog/changelog/2026-02-04-claude-and-codex-are-now-available-in-public-preview-on-github/

GitHub enables multi-agent AI coding inside repository workflows - Help Net Security

https://www.helpnetsecurity.com/2026/02/05/github-enables-coding-agents/

GitHub Adds Claude Code and OpenAI Codex to Agent HQ Platform - WinBuzzer

https://winbuzzer.com/2026/02/05/github-agent-hq-claude-codex-multi-agent-platform-xcxwbn/
