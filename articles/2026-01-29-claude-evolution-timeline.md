---
title: "【完全版】Claude進化の歴史｜Claude 1からOpus 4.5まで全モデル徹底解説"
emoji: "🧠"
type: "tech"
topics: ["claude", "anthropic", "ai", "llm", "歴史"]
published: false
---

**「Claude 3.5 Opusはどこへ行った？」**

この質問、よく聞かれます。

答えは：**リリースされませんでした**。

Anthropicは3.5 Opusをスキップして、いきなりClaude 4シリーズに移行したのです。

この記事では、Claude 1から最新のOpus 4.5まで、**全モデルの進化の歴史**を完全解説します。

## 結論から言うと

:::message
Claudeは2年半で**5世代**のメジャーアップデートを経験。特に2025年5月のClaude 4リリースと、11月のOpus 4.5が転換点。現在、Opusモデルはコーディングとエンタープライズで最強クラス。
:::

## Claude年表

```
2023年3月   Claude 1 リリース
2023年7月   Claude 2 リリース
2024年3月   Claude 3ファミリー（Haiku/Sonnet/Opus）
2024年6月   Claude 3.5 Sonnet
2024年10月  Claude 3.5 Sonnet（新版）+ Haiku
2025年5月   Claude 4 Sonnet + Opus
2025年8月   Claude Opus 4.1
2025年11月  Claude Opus 4.5 ← 現在最強
```

## 各世代の詳細

### Claude 1（2023年3月）

Anthropicの最初の商用モデル。

**特徴**：
- 安全性を重視した設計
- Constitutional AI導入
- コンテキスト長：9,000トークン

**限界**：
- GPT-4に性能で劣る
- マルチモーダル非対応

### Claude 2（2023年7月）

大幅な性能向上。

**特徴**：
- コンテキスト長：**100,000トークン**（当時最長）
- コーディング能力向上
- より自然な対話

**ベンチマーク**：
- HumanEval: 71.2%
- Bar Exam: 76.5%

### Claude 3ファミリー（2024年3月）

**3つのモデル**に分化。

| モデル | 特徴 | 価格（入力/出力） |
|--------|------|------------------|
| Haiku | 速度重視 | $0.25 / $1.25 |
| Sonnet | バランス型 | $3 / $15 |
| Opus | 最高性能 | $15 / $75 |

**革新点**：
- マルチモーダル対応（画像理解）
- 200Kコンテキスト
- GPT-4を超えるベンチマーク

### Claude 3.5 Sonnet（2024年6月）

**衝撃の発表**：Sonnet価格でOpus超えの性能。

```
Claude 3 Opus: $15/$75 → 性能100
Claude 3.5 Sonnet: $3/$15 → 性能120（！）
```

**新機能**：
- Artifacts（コード実行環境）
- Projects（コンテキスト管理）

### Claude 3.5 Sonnet v2 + Haiku（2024年10月）

**Computer Use**機能が登場。

```python
# AIがPCを操作できるように
from anthropic import Anthropic

client = Anthropic()
response = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    tools=[{"type": "computer_20241022", "display_width": 1024, "display_height": 768}],
    messages=[{"role": "user", "content": "ブラウザでGoogleを開いて"}]
)
```

### Claude 3.5 Opusの謎

**予告されたが、リリースされなかった**モデル。

Anthropicの発表（2024年6月）：
> 「Claude 3.5 HaikuとClaude 3.5 Opusは年内にリリース予定」

実際：
- Claude 3.5 Haiku → **リリースされた**（2024年10月）
- Claude 3.5 Opus → **リリースされなかった**

**理由**（推測）：
1. Claude 4の開発が予想より早く進んだ
2. 3.5 Opusの性能が4シリーズと重複
3. リソースの集中投下

### Claude 4 Sonnet + Opus（2025年5月）

**メジャーバージョンアップ**。

| モデル | GPQA | HumanEval | 価格（入力/出力） |
|--------|------|-----------|------------------|
| Claude 4 Sonnet | 82.1% | 88.5% | $3 / $15 |
| Claude 4 Opus | 86.3% | 91.2% | $15 / $75 |

**新機能**：
- 改良されたエージェント能力
- より長い出力
- マルチターン推論の向上

### Claude Opus 4.1（2025年8月）

**マイナーアップデート**。

- コーディング精度向上
- 指示追従性改善
- レイテンシ削減

### Claude Opus 4.5（2025年11月）

**現在の最強モデル**。

**ベンチマーク**：

| ベンチマーク | Opus 4.5 | GPT-4o | Gemini 3 Flash |
|-------------|----------|--------|----------------|
| GPQA Diamond | **89.1%** | 88.2% | 90.4% |
| HumanEval | **93.5%** | 90.2% | 92.1% |
| SWE-bench | **72.3%** | 65.1% | 68.7% |

**特徴**：
- コーディングタスクで最高性能
- エンタープライズワークフロー特化
- スプレッドシート等のビジネスタスク

## 現在利用可能なモデル（2026年1月）

### 推奨モデル

| モデル | ID | 用途 |
|--------|-----|------|
| Opus 4.5 | claude-opus-4-5-20251101 | 最高性能が必要な場面 |
| Sonnet 4 | claude-sonnet-4-20250514 | 日常的な開発 |
| Haiku 3.5 | claude-3-5-haiku-20241022 | 高速処理 |

### 廃止されたモデル

:::message alert
以下のモデルは**廃止済み**です。APIリクエストはエラーを返します。
:::

- claude-3-opus-20240229
- claude-3-sonnet-20240229
- claude-3-5-sonnet-20240620
- claude-3-5-sonnet-20241022

## Claude Codeとの関係

**Claude Code**はCLIツールであり、モデルではありません。

```
Claude Code（ツール）
    ↓ 使用
Claude Opus 4.5 / Sonnet 4（モデル）
```

Claude Codeの設定：
```bash
# デフォルト：Sonnet 4
claude "Hello"

# Opus 4.5を指定
claude --model opus-4.5 "Complex task"

# Haikuを指定（高速）
claude --model haiku "Quick question"
```

## 今後の予測

### Claude 5？

公式発表はありませんが、業界の予測：

```
2026年Q2-Q3: Claude 5 Sonnet?
2026年Q4: Claude 5 Opus?
```

### 予想される進化

1. **100万トークンコンテキスト**（Geminiに追いつく）
2. **ネイティブツール統合**（Google Workspaceのような）
3. **マルチモーダル強化**（動画理解）
4. **リアルタイム対話**（低レイテンシ音声）

## まとめ

| 世代 | 年月 | 最大の変化 |
|------|------|-----------|
| Claude 1 | 2023/03 | Constitutional AI |
| Claude 2 | 2023/07 | 100Kコンテキスト |
| Claude 3 | 2024/03 | 3モデル分化、マルチモーダル |
| Claude 3.5 | 2024/06 | Sonnet＞Opus、Computer Use |
| Claude 4 | 2025/05 | エージェント能力 |
| Claude 4.5 | 2025/11 | コーディング最強 |

:::message
Claudeの進化は「安全性」と「実用性」の両立を追求してきた歴史。Opus 4.5は現時点でコーディングタスクにおいて最高峰のモデルです。
:::

---

この記事が役に立ったら、**いいね**と**保存**をお願いします！

**質問**: あなたが最も印象的だったClaudeのアップデートは何ですか？コメントで教えてください！

## 参考リンク

- [Anthropic Claude Timeline - ScriptByAI](https://www.scriptbyai.com/anthropic-claude-timeline/)
- [Claude Opus 4.5 - Anthropic](https://www.anthropic.com/claude/opus)
- [Claude Release Notes - Anthropic Docs](https://platform.claude.com/docs/en/release-notes/overview)
- [Model Deprecations - Claude API Docs](https://platform.claude.com/docs/en/about-claude/model-deprecations)
