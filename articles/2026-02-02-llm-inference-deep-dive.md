---
title: "【深層解説】なぜLLMは遅いのか？KV CacheとSpeculative Decodingの数学的理解"
emoji: "🧠"
type: "tech"
topics: ["llm", "機械学習", "deeplearning", "transformer", "最適化"]
published: false
---

## はじめに

「なぜClaude Codeは時々遅くなるのか？」
「なぜ長いコンテキストになると応答が重くなるのか？」

これらの疑問に答えるには、LLMの推論がどのように動作し、何がボトルネックになっているのかを理解する必要があります。

この記事では、LLM推論の**2大最適化技術**である「**KV Cache**」と「**Speculative Decoding**」を、数学的な背景から実装レベルまで深く解説します。

:::message
**対象読者**: LLMの内部動作に興味があるエンジニア、MLエンジニア、または「なぜ遅いのか」を本質的に理解したい方
:::

## 第1章: LLM推論の本質的なボトルネック

### 1.1 LLMは「計算律速」ではなく「メモリ律速」

多くの人が誤解していますが、LLMの推論が遅い理由は**計算量が多いから**ではありません。

**メモリ帯域幅が足りないから**です。

```
📊 NVIDIA H100 GPUのスペック:
├── 計算能力: 1,979 TFLOPS (FP8)
├── メモリ容量: 80GB HBM3
└── メモリ帯域幅: 3.35 TB/s

問題: GPUは計算能力を持て余している
```

LLM推論では、モデルの重みをメモリから読み込み、計算し、結果を書き戻すというサイクルを繰り返します。このとき、**メモリの読み書き速度が計算速度に追いつかない**のです。

### 1.2 自己回帰生成の呪い

LLMがテキストを生成する方法を「**自己回帰生成（Autoregressive Generation）**」と呼びます。

```python
# 自己回帰生成の疑似コード
def generate(prompt, max_tokens):
    tokens = tokenize(prompt)

    for _ in range(max_tokens):
        # 毎回、全てのトークンに対してAttentionを計算
        logits = model.forward(tokens)
        next_token = sample(logits[-1])
        tokens.append(next_token)

    return tokens
```

問題は、**1トークンを生成するたびに、過去の全トークンに対してAttention計算を行う**ことです。

これは計算量が O(n²) で増加することを意味します。

```
トークン数 | Attention計算回数
----------|------------------
100       | 10,000
1,000     | 1,000,000
10,000    | 100,000,000
100,000   | 10,000,000,000
```

128Kトークンのコンテキストウィンドウを持つモデルでは、これは壊滅的です。

## 第2章: KV Cacheの仕組み

### 2.1 Transformerの復習：Query, Key, Value

Transformerの核心は「**Self-Attention**」です。

```
Attention(Q, K, V) = softmax(QK^T / √d_k) × V
```

各トークンは3つのベクトルに変換されます：

| ベクトル | 役割 |
|---------|------|
| **Query (Q)** | 「何を探しているか」を表す |
| **Key (K)** | 「自分が何であるか」を表す |
| **Value (V)** | 「自分が持っている情報」を表す |

Attentionは、Queryと全てのKeyの類似度を計算し、その重みでValueを集約します。

### 2.2 なぜKeyとValueをキャッシュするのか

自己回帰生成では、新しいトークンを生成するたびに、**過去の全トークンのK,Vを再計算**していました。

```
❌ ナイーブな実装:
Token 1: K1, V1を計算
Token 2: K1, V1, K2, V2を計算  ← K1, V1は再計算
Token 3: K1, V1, K2, V2, K3, V3を計算  ← 全て再計算
```

しかし、過去のトークンのK,Vは**変化しない**ことに気づきます。

```
✅ KV Cache:
Token 1: K1, V1を計算 → キャッシュに保存
Token 2: K2, V2を計算 → キャッシュに追加
Token 3: K3, V3を計算 → キャッシュに追加

Attention計算時はキャッシュから読み込むだけ
```

これにより、計算量は **O(n²) → O(n)** に削減されます。

### 2.3 KV Cacheのメモリ消費量

しかし、KV Cacheには代償があります。**メモリ消費量**です。

```python
# KV Cacheのサイズ計算
def kv_cache_size(
    batch_size: int,
    seq_len: int,
    num_layers: int,
    num_heads: int,
    head_dim: int,
    dtype_bytes: int = 2  # FP16 = 2 bytes
) -> int:
    # K と V それぞれ保存
    return 2 * batch_size * seq_len * num_layers * num_heads * head_dim * dtype_bytes

# Llama-2 70Bの例
size = kv_cache_size(
    batch_size=1,
    seq_len=128_000,  # 128Kコンテキスト
    num_layers=80,
    num_heads=64,
    head_dim=128,
    dtype_bytes=2
)
print(f"KV Cache size: {size / 1e9:.1f} GB")
# → KV Cache size: 167.8 GB
```

**128Kコンテキストで167GB**のKV Cacheが必要です。

これは、最高級のH100 GPU（80GB）でも足りません。

### 2.4 KV Cacheが「本当の」ボトルネック

ここで重要な洞察があります：

> **LLM推論のボトルネックは、モデルの重みではなく、KV Cacheである**

長いコンテキストになればなるほど、KV Cacheのサイズは線形に増加します。そして、このKV Cacheの読み書きが、メモリ帯域幅を食い尽くすのです。

## 第3章: KV Cache最適化技術

### 3.1 量子化（Quantization）

最もシンプルな解決策は、KV Cacheの精度を下げることです。

```
📊 量子化によるメモリ削減:
├── FP16 (16-bit): 基準
├── FP8 (8-bit): 50%削減
├── INT4 (4-bit): 75%削減
└── INT2 (2-bit): 87.5%削減
```

NVIDIAのTensorRT-LLMは、KV CacheのFP8量子化をサポートしています。

```python
# TensorRT-LLMでのKV Cache量子化設定
config = {
    "kv_cache_dtype": "fp8",  # or "int4"
    "kv_cache_free_gpu_memory_fraction": 0.9
}
```

**FP8量子化では、精度低下は1%未満**と報告されています。

### 3.2 Grouped-Query Attention (GQA)

Llama-2 70Bで採用された技術です。

通常のMulti-Head Attention (MHA)では、各ヘッドが独自のK,Vを持ちます：

```
MHA: Q1,K1,V1 | Q2,K2,V2 | Q3,K3,V3 | Q4,K4,V4
     ↓    ↓    ↓    ↓    ↓    ↓    ↓    ↓
     全て独立
```

GQAでは、複数のQueryヘッドが同じK,Vを共有します：

```
GQA: Q1,Q2 → K1,V1 | Q3,Q4 → K2,V2
     ↓  ↓     ↓  ↓   ↓  ↓     ↓  ↓
     K,Vを共有    K,Vを共有
```

これにより、KV Cacheのサイズを**ヘッド数に比例して削減**できます。

### 3.3 Sliding Window Attention (SWA)

Mistral-7Bで採用された技術です。

全てのトークンのK,Vを保存するのではなく、**直近のWトークン分だけ**を保持します。

```
📦 Sliding Window (W=4096):
Token 1-4096:    [K,V保持]
Token 4097:      Token 1を削除、Token 4097を追加
Token 4098:      Token 2を削除、Token 4098を追加
...
```

これにより、コンテキスト長に関係なく、**KV Cacheサイズを固定**できます。

### 3.4 Multi-Head Latent Attention (MLA)

DeepSeekが開発した最新技術です。

MLAの核心は「**低ランク射影**」です。K,Vをそのまま保存するのではなく、圧縮された「潜在表現」を保存します。

```
従来: K, V をそのまま保存 (d次元)
MLA:  c = compress(K, V) を保存 (r次元, r << d)
      K, V = decompress(c) で復元
```

DeepSeekの報告によると：

- **KV Cacheサイズ: 93%削減**
- **生成スループット: 6倍向上**

## 第4章: Speculative Decoding

### 4.1 基本的なアイデア

Speculative Decoding（投機的デコーディング）は、2022年にGoogleが発表した革命的な技術です。

核心となるアイデア：

> **小さなモデルで「予測」し、大きなモデルで「検証」する**

```
📊 Speculative Decodingの構成:
├── Draft Model（ドラフトモデル）: 小さくて高速
└── Target Model（ターゲットモデル）: 大きくて高精度
```

### 4.2 アルゴリズムの詳細

```python
def speculative_decode(prompt, draft_model, target_model, k=5):
    tokens = tokenize(prompt)

    while not done:
        # Step 1: ドラフトモデルでK個のトークンを予測
        draft_tokens = []
        for _ in range(k):
            next_token = draft_model.generate_one(tokens + draft_tokens)
            draft_tokens.append(next_token)

        # Step 2: ターゲットモデルで一括検証
        # ※ K個のトークンを「並列に」検証できる
        target_probs = target_model.forward(tokens + draft_tokens)

        # Step 3: 受理/棄却の判定
        accepted = 0
        for i, draft_token in enumerate(draft_tokens):
            if accept(draft_token, target_probs[i]):
                accepted += 1
            else:
                break

        # Step 4: 受理されたトークンを追加
        tokens.extend(draft_tokens[:accepted])

        # Step 5: 棄却された位置で正しいトークンを生成
        if accepted < k:
            correct_token = sample(target_probs[accepted])
            tokens.append(correct_token)

    return tokens
```

### 4.3 なぜこれが速いのか

ポイントは、**ターゲットモデルの呼び出し回数が減る**ことです。

```
❌ 通常の生成 (10トークン):
Target呼び出し: 10回

✅ Speculative Decoding (k=5, 受理率80%):
Draft呼び出し: 12回（速い）
Target呼び出し: 3回（遅いが回数減）
```

ドラフトモデルはターゲットモデルの**10倍以上速い**ことが多いため、全体として高速化されます。

### 4.4 数学的保証：出力品質は同一

驚くべきことに、Speculative Decodingは**出力品質を一切犠牲にしません**。

これは、受理/棄却のアルゴリズムが**ターゲットモデルの出力分布を完全に保存**するよう設計されているからです。

受理確率は以下で計算されます：

```
accept_prob = min(1, target_prob / draft_prob)
```

この式により、最終的な出力分布はターゲットモデル単体と**数学的に同一**になることが証明されています。

### 4.5 実際のパフォーマンス

```
📊 Speculative Decodingの効果:
├── 最適なk値を選んだ場合: 20-54%のコスト削減
├── 間違ったk値を選んだ場合: 175%のコスト増加（逆効果）
└── N-gram + FP8の組み合わせ: 最大3倍の高速化
```

重要な点：**Speculative Decodingは「Plug and Play」ではない**

最適なk値（予測トークン数）は、タスクとモデルの組み合わせによって大きく異なります。チューニングが必須です。

## 第5章: 最先端の組み合わせ技術

### 5.1 QuantSpec: 量子化 + Speculative Decoding

2025年に発表されたQuantSpecは、両方の技術を組み合わせます。

```
📦 QuantSpecの構成:
├── ドラフトモデル: 4-bit量子化されたKV Cache
├── ターゲットモデル: フル精度
└── 階層的量子化: 重要なトークンは高精度で保持
```

効果：

- 短いコンテキスト: 重み量子化が効果的
- 長いコンテキスト: KV Cache量子化が効果的

### 5.2 BitDecoding: Tensor Coreの活用

従来、低ビットKV Cacheの処理はCUDA Coreのみで行われていました。

BitDecodingは、**Tensor Coreも活用**することで、さらなる高速化を実現します。

```
📊 BitDecodingの効果:
└── 128Kコンテキストで最大3.3倍の高速化
```

### 5.3 N-gram Speculative Decoding

ドラフトモデルを使う代わりに、**過去のトークンパターン（N-gram）**から次のトークンを予測する手法です。

```
利点:
├── ドラフトモデル不要（メモリ節約）
├── 計算オーバーヘッドほぼゼロ
└── FP8量子化との相性が抜群
```

## 第6章: 実践的な適用

### 6.1 vLLMでの設定例

```python
from vllm import LLM, SamplingParams

# Speculative Decodingの設定
llm = LLM(
    model="meta-llama/Llama-2-70b-hf",
    speculative_model="meta-llama/Llama-2-7b-hf",
    num_speculative_tokens=5,
    speculative_draft_tensor_parallel_size=1,
)

# KV Cache量子化の設定
llm = LLM(
    model="meta-llama/Llama-2-70b-hf",
    kv_cache_dtype="fp8",
    quantization="fp8",
)
```

### 6.2 TensorRT-LLMでの設定例

```python
import tensorrt_llm

# KV Cache量子化
config = tensorrt_llm.BuildConfig(
    max_batch_size=8,
    max_input_len=2048,
    max_output_len=512,
    kv_cache_config=tensorrt_llm.KvCacheConfig(
        dtype="fp8",
        free_gpu_memory_fraction=0.9,
    )
)
```

### 6.3 最適化の指針

| シナリオ | 推奨技術 |
|----------|----------|
| 短いコンテキスト、バッチ処理 | 重み量子化 |
| 長いコンテキスト、単一リクエスト | KV Cache量子化 |
| 低レイテンシ重視 | Speculative Decoding |
| メモリ制約あり | GQA + SWA |
| 最大スループット | 全て組み合わせ |

## まとめ

### 核心的な理解

1. **LLM推論はメモリ律速** - GPUの計算能力ではなく、メモリ帯域幅がボトルネック
2. **KV Cacheは諸刃の剣** - 計算量を削減するが、メモリを大量消費
3. **量子化は万能薬に近い** - FP8で精度低下1%未満、メモリ50%削減
4. **Speculative Decodingは魔法** - 品質を犠牲にせず、最大3倍高速化

### 2026年のトレンド

- **MLA（Multi-Head Latent Attention）**の普及
- **N-gram + 量子化**の組み合わせ
- **128K+ コンテキスト**の標準化
- **分散推論**の民主化

---

**この記事が参考になったら、いいね👍とストックをお願いします！**

「ここが分からない」「もっと詳しく知りたい」という点があれば、コメントで教えてください。

---

## 参考文献

- [Fast Inference from Transformers via Speculative Decoding (Google, 2022)](https://arxiv.org/abs/2211.17192)
- [A Survey on Large Language Model Acceleration based on KV Cache Management](https://arxiv.org/html/2412.19442v3)
- [QuantSpec: Self-Speculative Decoding with Hierarchical Quantized KV Cache](https://arxiv.org/html/2502.10424v1)
- [Optimizing Inference with NVFP4 KV Cache - NVIDIA](https://developer.nvidia.com/blog/optimizing-inference-for-long-context-and-large-batch-sizes-with-nvfp4-kv-cache)
- [Decoding Multi-Head Latent Attention - Vizuara](https://vizuara.substack.com/p/decoding-multi-head-latent-attention)
- [Speculative Decoding - BentoML](https://bentoml.com/llm/inference-optimization/speculative-decoding)
