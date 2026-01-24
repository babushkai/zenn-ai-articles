---
title: "【衝撃】Claude Codeの生みの親が明かした「週100PR」を実現する神ワークフロー13選"
emoji: "🔥"
type: "tech"
topics: ["claudecode", "ai", "生成ai", "開発効率化", "プログラミング"]
published: false
---

**「1ヶ月で259PR、497コミット」**

この数字を見て、あなたはどう思いましたか？

「嘘でしょ？」「どうせAIが勝手にやってるだけでしょ」

いいえ、これは**Claude Codeの生みの親であるBoris Cherny氏**が実際に叩き出した数字です。しかも彼はこのワークフローをX（旧Twitter）で公開し、世界中の開発者を震撼させました。

:::message
**結論から言うと：** Boris氏は「5つのClaudeを並列実行」「Plan Modeで事前設計」「共有CLAUDE.md」という3つの柱で、常人の10倍以上の生産性を実現しています。
:::

## なぜ今、この記事を読むべきなのか

2026年1月、AIコーディングツールの競争は激化しています。Cursor、GitHub Copilot、各種AIエディタが乱立する中、**Claude Codeは「エンジニアが本当に使うツール」として急速にシェアを伸ばしています。**

しかし、ほとんどの開発者は**Claude Codeの本当のポテンシャルを引き出せていません。**

Boris氏曰く：
> 「多くの人がClaude Codeを単なるチャットボットとして使っている。それは、フェラーリでコンビニに行くようなものだ」

この記事では、Boris氏が明かした**13のパワーテクニック**を完全解説します。

---

## 🚀 テクニック1：5つのClaudeを並列実行せよ

**Before：** 1つのターミナルでClaude Codeを実行、待ち時間が発生
**After：** 5つの並列セッションで、待ち時間ゼロ

```bash
# ターミナルを5つ開いて番号付け
# Tab 1: フロントエンド開発
# Tab 2: バックエンドAPI
# Tab 3: テスト作成
# Tab 4: ドキュメント
# Tab 5: コードレビュー対応
```

Boris氏は**ローカルで5つ、claude.aiで5〜10個**のセッションを同時に実行しています。

:::message alert
**重要ポイント：** 各セッションは**別々のgit checkout**を使用すること！ブランチやworktreeではなく、完全に独立したディレクトリで作業することで、コンフリクトを完全に回避しています。
:::

---

## 🎯 テクニック2：Plan Mode（計画モード）を必ず使え

これが**最も重要なテクニック**かもしれません。

```
Shift + Tab を2回押す → Plan Modeに入る
```

Boris氏のワークフロー：

1. **目標がPRの場合**、まずPlan Modeで設計を練る
2. Claudeと**何度も往復**して、計画を磨き上げる
3. 計画に満足したら、**auto-accept editsモード**に切り替え
4. Claudeが**一発で実装完了**

> 「Plan Modeなしでコードを書かせるのは、設計図なしで家を建てるようなものだ」
> — Boris Cherny

---

## ⚡ テクニック3：Opus 4.5を使え、Sonnetではなく

多くの開発者が「Sonnetの方が速いから」とSonnetを選んでいます。

**これは間違いです。**

Boris氏の主張：
- Opusは確かに**レスポンスが遅い**
- しかし、**人間のガイダンスが少なくて済む**
- 結果として、**トータルの完了時間は短くなる**

```
Sonnet: 応答速度 ⚡⚡⚡ → 手直し多い → 合計時間 ⏱️⏱️⏱️⏱️
Opus:  応答速度 ⚡ → 一発でOK → 合計時間 ⏱️⏱️
```

:::message
**2026年1月現在：** `claude-opus-4-5-20251101` がClaude Code標準で利用可能です。特に複雑なコーディングタスクでは、Opus 4.5 with thinkingを使用することを強く推奨します。
:::

---

## 📚 テクニック4：共有CLAUDE.mdでチーム全体を強化

Boris氏のチームでは、**CLAUDE.mdファイルをgitにコミット**しています。

```markdown
# CLAUDE.md（チーム共有版）

## プロジェクトの概要
このプロジェクトは...

## コーディング規約
- 関数は50行以内
- TypeScriptを使用
- テストは必須

## よくある間違い
- ❌ any型の使用
- ❌ console.logの放置
- ✅ 適切なエラーハンドリング
```

**さらに重要なのは：** PRのコードレビュー時に `@claude` でタグ付けして、ガイドラインを**継続的に更新**していること。

---

## 🔧 テクニック5：カスタマイズはしない

これは意外かもしれません。

**Boris氏はClaude Codeをほぼカスタマイズしていません。**

> 「Claude Codeは箱から出してすぐに最高のパフォーマンスを発揮する。下手にカスタマイズすると、むしろ性能が落ちることがある」

もちろん、CLAUDE.mdでプロジェクト固有の指示は与えますが、ツール自体の設定はデフォルトのままです。

---

## 🔔 テクニック6：システム通知を活用せよ

5つのClaudeを並列で動かすと、**どのタブがあなたの入力を待っているか**分からなくなります。

Boris氏の解決策：**システム通知**

Claude Codeは入力が必要になると通知を送ります。これにより：
- 他の作業をしながら待機できる
- 必要なときだけ該当タブに戻れる
- 時間のロスがゼロ

---

## 🌐 テクニック7：「テレポート」コマンドでWeb⇔ローカルを行き来

Boris氏は、**claude.aiとローカルのClaude Codeを連携**させています。

```bash
# Webで作成した内容をローカルに「テレポート」
claude --teleport <session-id>
```

これにより：
- 複雑な調査や設計はWebのUIで
- 実装はローカルのターミナルで
- **シームレスな作業の引き継ぎ**

---

## 📊 テクニック8：1ヶ月で259PR達成の秘訣

Boris氏の30日間の実績を分解すると：

| 指標 | 数値 |
|------|------|
| PR数 | 259 |
| コミット数 | 497 |
| 追加行数 | 40,000+ |
| 削除行数 | 38,000+ |

**ポイント：** 追加と削除がほぼ同数 = **リファクタリングも含めた質の高い開発**

---

## 🤖 テクニック9：Claude Code 2.1.0の新機能を使いこなせ

2026年1月にリリースされたClaude Code 2.1.0には、生産性を爆上げする機能が追加されています：

### MCP list_changed通知
MCPサーバーが動的にツールを更新可能に。再接続不要でワークフローが途切れない。

### 権限拒否後もエージェント継続
サブエージェントが権限を拒否されても、**代替アプローチを試行**。完全停止しない。

### スキルのホットリロード
新しいスキルが**再起動なしで即座に利用可能**。

```bash
# Claude Code 2.1.0へのアップデート
npm update -g @anthropic-ai/claude-code
```

---

## 💡 テクニック10：MCPツール検索で精度74%→88%へ

Anthropicの最新発表によると、**Tool Search機能**によりMCP評価の精度が劇的に向上：

- Opus 4: 49% → **74%**
- Opus 4.5: 79.5% → **88.1%**

これは何を意味するか？

**Claudeが適切なツールを見つける能力が格段に上がった**ということです。

---

## 🔄 テクニック11：auto-accept editsモードの正しい使い方

```
Plan Mode → 計画確定 → auto-accept edits ON → 一発実装
```

このフローが重要な理由：

1. 計画なしでauto-acceptすると、**暴走のリスク**
2. 計画ありでauto-acceptすれば、**一発で完璧な実装**
3. 人間の確認は**計画段階で集中的に**行う

---

## 📱 テクニック12：MacBook + ターミナル5分割

Boris氏の作業環境は意外とシンプル：

- MacBook（スペック詳細は非公開）
- ターミナル5分割
- claude.aiを別ウィンドウで
- **シンプルな環境で最大の成果**

> 「複雑なツールチェーンより、シンプルなワークフローを極めろ」

---

## 🎉 テクニック13：Claude Cowork（非エンジニア向け）の活用

2026年1月12日、Anthropicは**Cowork**をリリースしました。

これはClaude Codeの非技術者版で：
- ファイル操作、データ整理などを自動化
- Claude Maxサブスクライバー限定
- macOSデスクトップアプリで利用可能

エンジニアとしても、**非技術的なタスク**（ドキュメント整理、データ変換など）をCoworkに任せることで、さらなる効率化が可能です。

---

## まとめ：今すぐ実践できる3つのアクション

1. **Plan Mode（Shift+Tab×2）を今日から使う**
2. **チームでCLAUDE.mdを共有する**
3. **Opus 4.5 with thinkingに切り替える**

この3つだけで、あなたの開発効率は**確実に2倍以上**になります。

---

## 📚 参考リンク

- [Boris Cherny's Workflow - VentureBeat](https://venturebeat.com/technology/the-creator-of-claude-code-just-revealed-his-workflow-and-developers-are)
- [22 Tips from Boris Cherny - Medium](https://medium.com/@joe.njenga/boris-cherny-claude-code-creator-shares-these-22-tips-youre-probably-using-it-wrong-1b570aedefbe)
- [Claude Code's Creator 100 PRs a Week - Medium](https://medium.com/vibe-coding/claude-codes-creator-100-prs-a-week-his-setup-will-surprise-you-7d6939c99f2b)
- [Claude Code 2.1.0 Update - VentureBeat](https://venturebeat.com/orchestration/claude-code-2-1-0-arrives-with-smoother-workflows-and-smarter-agents)
- [13 Power Techniques from Boris Cherny - Medium](https://jinlow.medium.com/13-claude-code-power-techniques-insights-from-its-creator-ef4dddd3a9a1)

---

:::message
**この記事が役に立ったら、いいね👍と保存📌をお願いします！**

皆さんはClaude Codeをどのように活用していますか？ぜひコメントで教えてください！

次回は「MCPサーバーの構築方法」について完全解説予定です。お楽しみに！
:::
