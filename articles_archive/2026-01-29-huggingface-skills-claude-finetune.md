---
title: "【革命】ClaudeがオープンソースLLMをファインチューニングできるようになった｜Hugging Face Skills完全ガイド"
emoji: "🤗"
type: "tech"
topics: ["ai", "huggingface", "claudecode", "ファインチューニング", "llm"]
published: true
---

**「Claude、このデータでLlama 3.2をファインチューニングして」**

これが、2026年1月の今、**実際に動くコマンド**になりました。

Hugging Faceがリリースした**「Skills」**により、Claude Code（および他のコーディングエージェント）が**LLMのファインチューニングを完全に自動化**できるようになったのです。

:::message
スクリプトを書くだけじゃない。**GPUジョブの投入、進捗監視、モデルのHub公開**まで全自動。
:::

## 🔥 Hugging Face Skillsとは

### 従来のファインチューニングワークフロー

```
┌─────────────────────────────────────────────────────────┐
│            従来のファインチューニング作業                  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. トレーニングスクリプト作成（数時間）                   │
│  2. 環境構築・依存関係解決（数時間）                       │
│  3. GPUインスタンス起動・設定（1時間）                     │
│  4. ジョブ投入・エラー対応（数時間〜数日）                  │
│  5. メトリクス監視・ハイパラ調整（数日）                   │
│  6. モデル変換・デプロイ（数時間）                         │
│                                                         │
│  合計: 1週間〜数週間                                      │
└─────────────────────────────────────────────────────────┘
```

### Hugging Face Skills導入後

```
┌─────────────────────────────────────────────────────────┐
│            Skills導入後のワークフロー                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  あなた: 「このデータでQwen 2.5をSFTして、                │
│          DPOで整列させて、GGUFに変換して」                │
│                                                         │
│  Claude: 「了解。ジョブを投入します...                    │
│          [進捗: ████████░░ 80%]                         │
│          完了しました。Hubにpushしました:                  │
│          huggingface.co/your-org/your-model」            │
│                                                         │
│  合計: 数時間（あなたの作業時間: 1分）                     │
└─────────────────────────────────────────────────────────┘
```

## 🛠️ セットアップ方法

### Claude Codeへの導入

```bash
# 1. Hugging Face トークンを環境変数に設定
export HF_TOKEN="hf_xxxxxxxxxxxxxxxxxxxxx"

# 2. Skills MCPを追加
claude mcp add --transport http hf-skills \
  https://huggingface.co/mcp?bouquet=skills \
  --header "Authorization: Bearer $HF_TOKEN"
```

これだけで、Claudeがファインチューニング能力を獲得します。

### 対応エージェント

| エージェント | 対応状況 |
|------------|---------|
| **Claude Code** | ✅ 対応済み |
| **OpenAI Codex** | ✅ 対応済み |
| **Gemini CLI** | ✅ 対応済み |
| **Cursor** | 🔄 対応予定 |
| **Windsurf** | 🔄 対応予定 |
| **Continue** | 🔄 対応予定 |

## 📊 Skillsでできること

### 1. 教師ありファインチューニング（SFT）

```
あなた: 「customer_support.jsonlでLlama 3.2 3Bを
        カスタマーサポート向けにファインチューニングして」

Claude: ジョブを設定しています...
        - ベースモデル: meta-llama/Llama-3.2-3B-Instruct
        - データセット: customer_support.jsonl (15,000サンプル)
        - 手法: LoRA (rank=16, alpha=32)
        - GPU: A100 40GB x 1

        ジョブを投入しました。ID: job-abc123
        推定完了時間: 2時間15分

        [進捗監視中...]
```

### 2. DPO（Direct Preference Optimization）

```
あなた: 「SFT後のモデルにDPOを適用して、
        より丁寧な回答を生成するようにして」

Claude: DPOジョブを設定しています...
        - 入力モデル: your-org/llama-sft-customer
        - 選好データ: preferences.jsonl
        - β値: 0.1

        ジョブ投入完了。
```

### 3. RLVR（Reinforcement Learning with Verifiable Rewards）

```
あなた: 「数学問題の正答率を上げるためにRLVRを適用」

Claude: RLVR設定中...
        - 検証関数: 数学的正確性チェッカー
        - 報酬設計: 正答 +1.0, 部分正解 +0.3, 誤答 -0.5

        ジョブ投入完了。
```

### 4. GGUF変換とローカルデプロイ

```
あなた: 「完成したモデルをGGUF Q4_K_Mに変換して」

Claude: 変換中...
        - 元サイズ: 6.4GB
        - 変換後: 1.8GB (72%削減)
        - 量子化: Q4_K_M

        Hubにアップロード完了:
        huggingface.co/your-org/model-Q4_K_M-GGUF

        ローカル実行コマンド:
        ollama run hf.co/your-org/model-Q4_K_M-GGUF
```

## 🎯 実践例：カスタムコーディングアシスタントの作成

### Step 1: データ準備

```json
// training_data.jsonl
{"messages": [
  {"role": "user", "content": "Pythonでクイックソートを実装して"},
  {"role": "assistant", "content": "```python\ndef quicksort(arr):\n    if len(arr) <= 1:\n        return arr\n    pivot = arr[len(arr) // 2]\n    left = [x for x in arr if x < pivot]\n    middle = [x for x in arr if x == pivot]\n    right = [x for x in arr if x > pivot]\n    return quicksort(left) + middle + quicksort(right)\n```"}
]}
```

### Step 2: Claudeに依頼

```
あなた: training_data.jsonlを使って、
       Qwen 2.5 7B をコーディングアシスタント用に
       ファインチューニングして。
       その後、DPOで「説明が丁寧な回答」を強化して、
       最後にGGUF Q5_K_Mで変換してほしい。
```

### Step 3: Claudeが自動実行

```
Claude: マルチステージパイプラインを設定します。

        Stage 1: SFT
        ├─ ベース: Qwen/Qwen2.5-7B-Instruct
        ├─ データ: 25,000サンプル
        ├─ 手法: LoRA (r=32)
        └─ 推定時間: 4時間

        Stage 2: DPO
        ├─ 入力: Stage 1の出力
        ├─ 選好データ: 自動生成
        └─ 推定時間: 2時間

        Stage 3: GGUF変換
        ├─ 量子化: Q5_K_M
        └─ 推定時間: 30分

        パイプライン投入完了。
        完了時にSlackで通知します。
```

## 💰 コスト感

### GPU利用料金の目安

| モデルサイズ | GPU | SFT時間 | 概算コスト |
|------------|-----|--------|-----------|
| 3B | A100 40GB | 2-3時間 | $5-10 |
| 7B | A100 40GB | 4-6時間 | $10-20 |
| 14B | A100 80GB | 8-12時間 | $30-50 |

:::message
従来のMLOpsエンジニア人件費と比較すると**95%以上のコスト削減**
:::

## 🚀 Sionic AIの活用事例

Sionic AIは、Hugging Face Skillsを使って**1日1,000以上のML実験**を実行しています。

```
彼らのワークフロー:
────────────────────────────────────
夜間: Claudeが自動でハイパラ探索
      → 100パターンのジョブを投入

朝: 結果を確認
    → 最良モデルを選択

日中: Claude が追加実験を提案・実行
────────────────────────────────────
```

> 「モデル構築の実際の作業において、Claudeはデフォルトのパートナーになった」
> — Sionic AI リサーチチーム

## 📊 従来手法との比較

| 項目 | 従来のMLOps | Hugging Face Skills |
|------|-------------|---------------------|
| スクリプト作成 | 手動（数時間） | 自動（秒） |
| 環境構築 | 手動（数時間） | 不要 |
| GPU管理 | 手動設定 | 自動選択 |
| ジョブ監視 | ダッシュボード確認 | Claudeが報告 |
| エラー対応 | 手動デバッグ | Claudeが自動修正 |
| モデル公開 | 手動アップロード | 自動push |
| **必要なスキル** | Python, PyTorch, DevOps | 自然言語のみ |

## 🔮 これが意味する未来

### データサイエンティストの役割変化

```
Before (2025年)
──────────────────────────
データサイエンティスト:
├─ データ準備: 20%
├─ モデル設計: 20%
├─ 実装・デバッグ: 40% ← ここが消える
└─ 評価・改善: 20%

After (2026年〜)
──────────────────────────
データサイエンティスト:
├─ 問題設計: 30%
├─ データ戦略: 30%
├─ 実験設計（Claudeへの指示）: 20%
└─ 結果解釈・意思決定: 20%
```

### 誰でもカスタムLLMを持てる時代

```
必要なもの:
✅ Hugging Faceアカウント
✅ Claude Code（または対応エージェント）
✅ トレーニングデータ
✅ クレジットカード（GPU費用）

不要になったもの:
❌ PyTorchの深い知識
❌ CUDA/分散学習の経験
❌ MLOpsインフラ構築能力
❌ 専任のMLエンジニア
```

## 📝 まとめ

| 変化 | インパクト |
|------|-----------|
| ファインチューニングの民主化 | 誰でもカスタムLLMを作れる |
| MLOps工数の劇的削減 | 週単位 → 時間単位 |
| 実験速度の向上 | 1日1,000実験が現実に |
| 参入障壁の低下 | 小規模チームでも競争可能 |

Hugging Face Skillsは、**「LLMファインチューニングのGitHub Copilot」**と言えます。

コードを書く必要がなくなったように、モデルを訓練するのにも専門知識が不要になる。

2026年、**「自社専用LLM」を持つことが当たり前**になる時代が始まりました。

---

## 参考リンク

- [We Got Claude to Fine-Tune an Open Source LLM | Hugging Face Blog](https://huggingface.co/blog/hf-skills-training)
- [How We Use Claude Code Skills to Run 1,000+ ML Experiments a Day | Sionic AI](https://huggingface.co/blog/sionic-ai/claude-code-skills-training)
- [Radar Trends to Watch: January 2026 | O'Reilly](https://www.oreilly.com/radar/radar-trends-to-watch-january-2026/)
