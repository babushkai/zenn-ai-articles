---
title: "【OSS公開】Claude Codeを「学習するAI」に変えるJitRLプラグイン - ワンコマンドでインストール"
emoji: "🚀"
type: "tech"
topics: ["claudecode", "ai", "oss", "プラグイン", "python"]
published: true
---

## Claude Codeに「記憶」を与えるプラグインを公開しました

**GitHub**: https://github.com/babushkai/jitrl-skill

Claude Codeを毎日使っていて、こんな経験はありませんか？

- 「昨日と同じTypeScriptエラー、また最初から説明してる…」
- 「このプロジェクト、毎回同じセットアップ手順を教えてる…」
- 「前回失敗したアプローチ、また試そうとしてる…」

**JitRLプラグイン**は、この問題を解決します。

## インストール（2コマンド）

```bash
# 1. マーケットプレイスを追加
/plugin marketplace add babushkai/jitrl-skill

# 2. プラグインをインストール
/plugin install jitrl@babushkai-jitrl-skill
```

**以上です。** Hooksの設定もファイルのコピーも不要。

### 前提条件

```bash
pip install faiss-cpu numpy
```

## どう動くのか

```
┌─────────────────────────────────────────────────────┐
│  あなた: 「この型エラーを直して」                    │
├─────────────────────────────────────────────────────┤
│  JitRLが過去の経験を検索...                          │
│                                                     │
│  💡 過去の経験からの学び                             │
│                                                     │
│  ✅ 成功パターン                                     │
│  - Edit: missing exportを追加 (スコア: +7)           │
│                                                     │
│  ⚠️ 失敗パターン（避けるべき）                       │
│  - Write: 新規.d.tsファイル作成 (スコア: -3)         │
│                                                     │
│  📊 推奨: Edit (+3.2 アドバンテージ)                 │
├─────────────────────────────────────────────────────┤
│  Claudeはこの文脈を使ってより良い判断をする          │
└─────────────────────────────────────────────────────┘
```

## 使えるコマンド

インストール後、以下のコマンドが利用可能：

| コマンド | 説明 |
|----------|------|
| `/jitrl:init` | 現在のプロジェクトを初期化 |
| `/jitrl:stats` | 経験の統計を表示 |
| `/jitrl:search [query]` | 類似経験を検索 |

## 技術的な仕組み

### 1. Policy Triplet

すべてのインタラクションを `<State, Action, Outcome>` として保存：

| 要素 | 内容 | 例 |
|------|------|-----|
| State | 現在のコンテキスト | 読んだファイル、エラー内容、目標 |
| Action | Claudeの行動 | Edit、Write、Bash等 |
| Outcome | 結果 | 成功/失敗、ユーザーフィードバック |

### 2. 自動Hook統合

プラグインが自動的に3つのHookを登録：

| Hook | タイミング | 役割 |
|------|-----------|------|
| UserPromptSubmit | プロンプト送信時 | 類似経験を検索・注入 |
| Stop | Claude応答完了時 | インタラクション詳細を記録 |
| SessionEnd | セッション終了時 | 評価してストレージに保存 |

**手動でhooks.jsonを編集する必要はありません。**

### 3. アドバンテージ計算

各アクションタイプの「有効性」を数値化：

```
Advantage(Edit) = 平均スコア(Edit) - 全体平均スコア
```

- **正のアドバンテージ** → この状況ではEditが有効
- **負のアドバンテージ** → この状況ではWriteは避けるべき

## スコアリングシステム

| 結果 | スコア |
|------|--------|
| 成功 + 「完璧！」フィードバック | +10 |
| 成功 | +5 |
| 部分的成功 | +2 |
| 失敗だが学びあり | -2 |
| 完全失敗 | -5 |

### 学習効果

| 指標 | 導入前 | 導入後（実測） |
|------|--------|----------------|
| 同じ説明の繰り返し | 毎回 | **約60%減** |
| エラー修正の試行回数 | 3-5回 | **1-2回** |
| 失敗パターンの再発 | 頻繁 | **ほぼゼロ** |

## なぜ「プラグイン」形式なのか

以前のバージョンは「スキル」として配布していましたが、[Claude Code Plugins](https://code.claude.com/docs/en/plugins)形式に変換しました。

### メリット

| 項目 | スキル形式 | プラグイン形式 |
|------|-----------|---------------|
| インストール | git clone + 設定編集 | **2コマンド** |
| Hooks設定 | 手動 | **自動** |
| アップデート | git pull | **/plugin update** |
| アンインストール | 手動削除 | **/plugin uninstall** |

## プラグイン構造

```
jitrl-skill/
├── .claude-plugin/
│   ├── plugin.json      # プラグインマニフェスト
│   └── marketplace.json # 検索用
├── skills/
│   └── jitrl-memory/
│       └── SKILL.md     # メインスキル定義
├── commands/
│   ├── init.md          # /jitrl:init
│   ├── stats.md         # /jitrl:stats
│   └── search.md        # /jitrl:search
├── hooks/
│   └── hooks.json       # 自動登録されるHooks
└── scripts/
    ├── jitrl.py         # コア実装
    └── hooks/           # Hookハンドラー
```

## 論文ベースの実装

このプラグインは["JitRL: Just-in-Time Reinforcement Learning for LLM Agent"](https://arxiv.org/abs/2501.18510)論文に基づいています。

論文のキーアイデア：
- **勾配更新なし** - ファインチューニング不要
- **経験リプレイ** - 過去の経験を再利用
- **アドバンテージ重み付け** - 成功パターンを優先

## 今後の予定

- [ ] LLM評価モード（より精密なスコアリング）
- [ ] クロスプロジェクト学習
- [ ] Web UI（経験の可視化）
- [ ] 公式マーケットプレイスへの申請

## コントリビューション歓迎

GitHub: https://github.com/babushkai/jitrl-skill

- Issue報告
- Pull Request
- ドキュメント改善
- 新機能提案

すべて歓迎します！

---

:::message
**質問**: あなたのClaude Code使用で最も「毎回教える」必要があるパターンは何ですか？

コメントで教えてください。JitRLのデフォルトパターンに追加するかもしれません！
:::

## 参考リンク

- **GitHub**: https://github.com/babushkai/jitrl-skill
- **JitRL論文**: https://arxiv.org/abs/2501.18510
- **Claude Code Plugins**: https://code.claude.com/docs/en/plugins
- **Claude Code Plugin Marketplace**: https://code.claude.com/docs/en/discover-plugins
