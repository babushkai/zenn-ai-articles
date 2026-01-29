---
title: "【実践】LoRA/QLoRAファインチューニング完全ガイド｜Google Colabの無料GPUでLLMを調整する方法"
emoji: "🎯"
type: "tech"
topics: ["lora", "llm", "機械学習", "finetuning", "ai"]
published: false
---

**「LLMをファインチューニングしたいけど、GPUが高すぎる...」**

A100を1時間使うと$2〜4。1日回すと$50〜100。個人には厳しい。

でも、**LoRA**と**QLoRA**を使えば、**Google Colabの無料T4 GPU**でもファインチューニングできます。

## 結論から言うと

:::message
- **LoRA**: パラメータの0.1%だけを学習。メモリ使用量1/10。
- **QLoRA**: さらに4bit量子化で1/3に。T4でも65Bモデルが動く。
- 品質低下はほぼなし。コスパ最強のファインチューニング手法。
:::

## LoRAとは？

### 基本概念

**Low-Rank Adaptation**（低ランク適応）の略。

```
フルファインチューニング:
- 全パラメータを更新
- 7Bモデルで28GB+ VRAM必要
- コスト高、時間かかる

LoRA:
- 追加の小さな行列だけを学習
- 元のモデルは凍結（変更しない）
- 7Bモデルで8GB VRAMで動く
```

### 数学的な仕組み

```
元の重み行列: W (d × k)
LoRA行列: A (d × r) と B (r × k)

更新後: W' = W + BA

r（ランク）= 8〜64（通常）
→ パラメータ数が劇的に減少
```

**例**: 4096×4096の行列
- フル: 16,777,216パラメータ
- LoRA (r=8): 65,536パラメータ（**0.4%**）

## QLoRAとは？

### LoRA + 量子化

```
LoRA: パラメータ効率化
QLoRA: LoRA + 4bit量子化 + ページドオプティマイザ

メモリ使用量:
フル: 100%
LoRA: 30%
QLoRA: 10%
```

### 技術的ブレイクスルー

1. **4-bit NormalFloat**: 新しい量子化形式
2. **Double Quantization**: 量子化定数も量子化
3. **Paged Optimizers**: メモリスパイクを防ぐ

**結果**: LLaMA 65Bを**単一の48GB GPU**でファインチューニング可能

## 実践：Google Colabでファインチューニング

### 環境設定

```python
!pip install -q transformers peft bitsandbytes accelerate datasets trl

import torch
from transformers import AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig
from peft import LoraConfig, get_peft_model, prepare_model_for_kbit_training
from trl import SFTTrainer
```

### モデルのロード（QLoRA用4bit）

```python
# 4bit量子化設定
bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_use_double_quant=True,  # 二重量子化
    bnb_4bit_quant_type="nf4",       # NormalFloat4
    bnb_4bit_compute_dtype=torch.bfloat16
)

# モデルロード
model_name = "meta-llama/Llama-2-7b-hf"
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    quantization_config=bnb_config,
    device_map="auto"
)

tokenizer = AutoTokenizer.from_pretrained(model_name)
tokenizer.pad_token = tokenizer.eos_token
```

### LoRA設定

```python
# LoRA設定
lora_config = LoraConfig(
    r=16,                      # ランク（8-64が一般的）
    lora_alpha=32,             # スケーリング係数
    target_modules=[           # 適用するレイヤー
        "q_proj", "k_proj", "v_proj", "o_proj",  # Attention
        "gate_proj", "up_proj", "down_proj"       # FFN
    ],
    lora_dropout=0.05,
    bias="none",
    task_type="CAUSAL_LM"
)

# モデルにLoRAを適用
model = prepare_model_for_kbit_training(model)
model = get_peft_model(model, lora_config)

# 学習可能パラメータを確認
model.print_trainable_parameters()
# 出力例: trainable params: 4,194,304 || all params: 6,742,609,920 || trainable%: 0.06%
```

### データセット準備

```python
from datasets import load_dataset

# 例：日本語指示データセット
dataset = load_dataset("kunishou/databricks-dolly-15k-ja")

def format_instruction(sample):
    return f"""### 指示:
{sample['instruction']}

### 入力:
{sample['input']}

### 回答:
{sample['output']}"""

# フォーマット適用
dataset = dataset.map(lambda x: {"text": format_instruction(x)})
```

### トレーニング

```python
from transformers import TrainingArguments

training_args = TrainingArguments(
    output_dir="./lora-output",
    num_train_epochs=3,
    per_device_train_batch_size=4,
    gradient_accumulation_steps=4,
    learning_rate=2e-4,
    fp16=True,
    logging_steps=10,
    save_strategy="epoch",
    optim="paged_adamw_8bit",  # ページドオプティマイザ
)

trainer = SFTTrainer(
    model=model,
    train_dataset=dataset["train"],
    args=training_args,
    tokenizer=tokenizer,
    max_seq_length=512,
    dataset_text_field="text",
)

# 学習開始
trainer.train()
```

### 保存と読み込み

```python
# LoRAアダプターのみ保存（数十MB）
model.save_pretrained("./my-lora-adapter")

# 推論時のロード
from peft import PeftModel

base_model = AutoModelForCausalLM.from_pretrained(model_name)
model = PeftModel.from_pretrained(base_model, "./my-lora-adapter")
```

## パラメータチューニングガイド

### ランク（r）の選び方

| r | パラメータ数 | 用途 |
|---|-------------|------|
| 4 | 最小 | シンプルなタスク |
| 8 | 小 | 一般的な微調整 |
| 16 | 中 | **推奨**（バランス良い） |
| 32 | 大 | 複雑なタスク |
| 64 | 最大 | 大規模な適応 |

### alpha値の設定

```
推奨: alpha = 2 × r

例:
r=16 → alpha=32
r=32 → alpha=64
```

### target_modules

```python
# 最小構成（Attentionのみ）
target_modules = ["q_proj", "v_proj"]

# 推奨構成（全Linear層）
target_modules = [
    "q_proj", "k_proj", "v_proj", "o_proj",
    "gate_proj", "up_proj", "down_proj"
]

# 研究結果: 全Linear層を対象にすると精度向上
```

## LoRA vs QLoRA：どちらを使う？

| 条件 | 推奨 |
|------|------|
| VRAM 24GB以上 | LoRA |
| VRAM 16GB以下 | **QLoRA** |
| 学習速度重視 | LoRA |
| メモリ制約厳しい | QLoRA |
| 品質最優先 | LoRA |

### 速度比較

```
LoRA:   100%（基準）
QLoRA:  139%（39%遅い）

理由: 量子化/逆量子化のオーバーヘッド
```

### 品質比較

```
研究結果: QLoRAとLoRAの品質差は「無視できるレベル」
```

## 2026年最新：LoRAFusion

4月のEUROSYS '26で発表予定の新技術。

**改善点**:
- カーネルレベルの最適化
- メモリアクセスの効率化
- QLoRAにも適用可能

**効果**:
- 学習速度2倍
- メモリ効率さらに向上

## よくある問題と解決策

### CUDA Out of Memory

```python
# 解決策1: バッチサイズを下げる
per_device_train_batch_size=2
gradient_accumulation_steps=8

# 解決策2: max_seq_lengthを下げる
max_seq_length=256

# 解決策3: gradient_checkpointingを有効化
model.gradient_checkpointing_enable()
```

### 過学習

```python
# 解決策: ドロップアウトを上げる
lora_dropout=0.1

# エポック数を減らす
num_train_epochs=1
```

## まとめ

| 項目 | フルFT | LoRA | QLoRA |
|------|--------|------|-------|
| VRAM（7B） | 28GB+ | 8GB | **6GB** |
| 学習時間 | 100% | 30% | 42% |
| 品質 | 100% | 99% | 98% |
| コスト | $$$ | $ | **$** |

:::message
2026年、LLMファインチューニングのデファクトは**LoRA/QLoRA**。フルファインチューニングは研究機関でも減少傾向。個人開発者でも高品質なカスタムLLMを作れる時代です。
:::

---

この記事が役に立ったら、**いいね**と**保存**をお願いします！

**質問**: LoRA/QLoRAでどんなモデルを作りましたか？コメントで教えてください！

## 参考リンク

- [LoRA vs QLoRA - Modal](https://modal.com/blog/lora-qlora)
- [Practical Tips for LoRA - Sebastian Raschka](https://magazine.sebastianraschka.com/p/practical-tips-for-finetuning-llms)
- [LoRAFusion Paper - arXiv](https://arxiv.org/html/2510.00206v1)
- [Efficient Fine-Tuning Guide - Databricks](https://www.databricks.com/blog/efficient-fine-tuning-lora-guide-llms)
