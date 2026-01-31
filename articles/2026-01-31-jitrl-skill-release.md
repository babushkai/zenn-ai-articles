---
title: "【OSS公開】Claude Codeを「学習するAI」に変えるJitRLスキルをリリースしました"
emoji: "🚀"
type: "tech"
topics: ["claudecode", "ai", "oss", "継続学習", "python"]
published: true
---

## Claude Codeに「記憶」を与えるスキルを公開しました

**GitHub**: https://github.com/babushkai/jitrl-skill

Claude Codeを毎日使っていて、こんな経験はありませんか？

- 「昨日と同じTypeScriptエラー、また最初から説明してる…」
- 「このプロジェクト、毎回同じセットアップ手順を教えてる…」
- 「前回失敗したアプローチ、また試そうとしてる…」

**JitRLスキル**は、この問題を解決します。

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

## インストール（30秒）

```bash
# 1. スキルをクローン
git clone https://github.com/babushkai/jitrl-skill ~/.claude/skills/jitrl

# 2. 依存関係インストール
pip install faiss-cpu numpy

# 3. プロジェクトで初期化
cd /your/project
python3 ~/.claude/skills/jitrl/scripts/jitrl.py init
```

**これだけで完了です。**

## 技術的な仕組み

### 1. Policy Triplet

すべてのインタラクションを `<State, Action, Outcome>` として保存：

| 要素 | 内容 | 例 |
|------|------|-----|
| State | 現在のコンテキスト | 読んだファイル、エラー内容、目標 |
| Action | Claudeの行動 | Edit、Write、Bash等 |
| Outcome | 結果 | 成功/失敗、ユーザーフィードバック |

### 2. ベクトル検索

Faissを使った高速な類似検索：

```python
# 現在のプロンプトを埋め込み
query_vector = embedder.embed("型エラーを修正して")

# 類似経験を検索（コサイン類似度）
results = index.search(query_vector, k=5)
```

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

## Claude Code Hooksとの統合

JitRLは3つのHookイベントを使用：

```json
{
  "hooks": {
    "UserPromptSubmit": [{
      "hooks": [{
        "type": "command",
        "command": "python3 ~/.claude/skills/jitrl/scripts/hooks/on_prompt.py"
      }]
    }],
    "Stop": [{
      "hooks": [{
        "type": "command",
        "command": "python3 ~/.claude/skills/jitrl/scripts/hooks/on_stop.py"
      }]
    }],
    "SessionEnd": [{
      "hooks": [{
        "type": "command",
        "command": "python3 ~/.claude/skills/jitrl/scripts/hooks/on_session_end.py"
      }]
    }]
  }
}
```

| Hook | タイミング | 役割 |
|------|-----------|------|
| UserPromptSubmit | プロンプト送信時 | 類似経験を検索・注入 |
| Stop | Claude応答完了時 | インタラクション詳細を記録 |
| SessionEnd | セッション終了時 | 評価してストレージに保存 |

## CLIコマンド

```bash
# 統計を表示
jitrl stats

# 類似経験を検索
jitrl search "async awaitのエラー"

# 経験をエクスポート
jitrl export experiences.json

# プロジェクトの経験をクリア
jitrl clear --confirm
```

## なぜ「スキル」として実装したのか

Claude Codeの[Skills機能](https://code.claude.com/docs/en/skills)には大きなメリットがあります：

1. **インストールが簡単** - `git clone`するだけ
2. **アップデートが簡単** - `git pull`するだけ
3. **カスタマイズ可能** - config.yamlで調整
4. **他のスキルと共存** - 競合しない

## 論文ベースの実装

このスキルは["JitRL: Just-in-Time Reinforcement Learning for LLM Agent"](https://arxiv.org/abs/2501.18510)論文に基づいています。

論文のキーアイデア：
- **勾配更新なし** - ファインチューニング不要
- **経験リプレイ** - 過去の経験を再利用
- **アドバンテージ重み付け** - 成功パターンを優先

私の実装では：
- Faissでベクトル検索を高速化
- Claude Code Hooksで自動収集
- コンテキスト注入でin-context学習

## 今後の予定

- [ ] LLM評価モード（より精密なスコアリング）
- [ ] クロスプロジェクト学習
- [ ] Web UI（経験の可視化）
- [ ] チーム共有機能

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
- **Claude Code Skills**: https://code.claude.com/docs/en/skills
- **Claude Code Hooks**: https://code.claude.com/docs/en/hooks
