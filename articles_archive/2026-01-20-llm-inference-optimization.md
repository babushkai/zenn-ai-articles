---
title: "LLM推論最適化の全技術 - KV Cache/Speculative Decoding/量子化"
emoji: "⚡"
type: "tech"
topics: ["llm", "推論最適化", "kvcache", "量子化", "ai"]
published: false
---

**「LLMの推論コストが高すぎる」**

GPUメモリ不足、レイテンシ、コスト...LLM運用の悩みは尽きません。

この記事では、2026年時点で確立されたLLM推論最適化技術を体系的に解説します。KV Cache、Speculative Decoding、量子化、そしてそれらの組み合わせ方まで。

## LLM推論のボトルネック

### なぜLLM推論は遅いのか

```
Transformer Decoder の推論:

入力: [token_1, token_2, ..., token_n]

Step 1: 全トークンを処理 (Prefill Phase)
Step 2: 1トークン生成
Step 3: 全トークン + 新トークンを処理
Step 4: 1トークン生成
...

→ 生成トークンごとに、全履歴を再計算
→ O(n²) の計算量
```

### 3つの主要ボトルネック

| ボトルネック | 原因 | 影響 |
|-------------|------|------|
| **メモリ帯域** | KV Cacheの読み書き | スループット低下 |
| **計算量** | Attention計算 | レイテンシ増加 |
| **メモリ容量** | モデル重み + KV Cache | バッチサイズ制限 |

---

## KV Cache

### KV Cacheとは

:::message
**KV Cache:** 過去のKey/Value行列を保存し、再計算を避ける最も基本的な最適化
:::

```python
# KV Cacheなし（毎回再計算）
def attention_naive(query, keys, values):
    # keys, values は全履歴を含む
    scores = query @ keys.T
    weights = softmax(scores)
    return weights @ values

# KV Cacheあり
class CachedAttention:
    def __init__(self):
        self.key_cache = []
        self.value_cache = []

    def forward(self, query, new_key, new_value):
        # 新しいKVをキャッシュに追加
        self.key_cache.append(new_key)
        self.value_cache.append(new_value)

        # キャッシュから全KVを取得（再計算不要）
        all_keys = torch.stack(self.key_cache)
        all_values = torch.stack(self.value_cache)

        # Attention計算
        scores = query @ all_keys.T
        weights = softmax(scores)
        return weights @ all_values
```

### KV Cacheのメモリ問題

```
KV Cacheのメモリ使用量:
= 2 × num_layers × batch_size × seq_len × num_heads × head_dim × dtype_size

例: Llama 70B, batch=32, seq=4096
= 2 × 80 × 32 × 4096 × 64 × 128 × 2 (FP16)
= 約 171 GB

→ モデル重み（140GB）より大きい！
```

---

## KV Cache最適化

### 1. KV Cache量子化

**アイデア:** KVをFP16/BF16から低精度形式に圧縮

```python
# vLLMでのKV Cache量子化
from vllm import LLM

llm = LLM(
    model="meta-llama/Llama-3-70B-Instruct",
    kv_cache_dtype="fp8_e4m3",  # FP8量子化
    # または "fp8_e5m2", "int8"
)
```

**効果:**
- FP8: メモリ50%削減、精度低下<1%
- INT8: メモリ50%削減、精度低下1-2%
- INT4: メモリ75%削減、精度低下3-5%

### 2. PagedAttention

**アイデア:** KV Cacheを固定サイズの「ページ」に分割し、必要に応じて割り当て

```python
# 従来: 連続メモリブロック
# → 最大シーケンス長分を事前確保 → メモリ無駄

# PagedAttention: ページ単位で動的割り当て
class PagedKVCache:
    def __init__(self, page_size=16, max_pages=1000):
        self.page_size = page_size
        self.page_pool = PagePool(max_pages)
        self.page_table = {}  # seq_id -> [page_ids]

    def allocate(self, seq_id, num_tokens):
        num_pages = (num_tokens + self.page_size - 1) // self.page_size
        pages = self.page_pool.allocate(num_pages)
        self.page_table[seq_id] = pages
        return pages

    def free(self, seq_id):
        pages = self.page_table.pop(seq_id)
        self.page_pool.free(pages)
```

**効果:** メモリ効率が最大55%向上（vLLM）

### 3. Prefix Caching

**アイデア:** 共通のシステムプロンプトのKV Cacheを共有

```python
class PrefixCache:
    """同じプレフィックスを持つリクエスト間でKV Cacheを共有"""

    def __init__(self):
        self.cache = {}  # prefix_hash -> kv_cache

    def get_or_compute(self, prefix, model):
        prefix_hash = hash(prefix)

        if prefix_hash in self.cache:
            # キャッシュヒット: 再利用
            return self.cache[prefix_hash].clone()
        else:
            # キャッシュミス: 計算してキャッシュ
            kv = model.compute_kv(prefix)
            self.cache[prefix_hash] = kv
            return kv
```

**効果:** 同じシステムプロンプトを使う場合、Prefill時間を大幅削減

### 4. NVFP4 KV Cache（最新）

NVIDIAのBlackwell GPUで利用可能：

```
従来のFP16 KV Cache:
- メモリ: 100%
- 精度: baseline

NVFP4 KV Cache:
- メモリ: 25%（4倍圧縮）
- 精度: <1%低下

→ コンテキスト長を4倍に、またはバッチサイズを4倍に
```

---

## Speculative Decoding

### 問題: Autoregressive生成の遅さ

```
通常の生成:
Token 1 → [Forward Pass] → Token 2 → [Forward Pass] → Token 3 → ...

各トークン生成にフルForward Passが必要
→ シーケンシャルで並列化できない
```

### Speculative Decodingのアイデア

:::message
**Speculative Decoding:** 小さいドラフトモデルで複数トークンを「投機的に」生成し、大きいターゲットモデルで一括検証
:::

```
Speculative Decoding:

1. Draft Model (小/速): [Token 1] → "A B C D E" (5トークン投機生成)

2. Target Model (大/遅): [Token 1, A, B, C, D, E] → 一括検証
   結果: "A B C" は正解、"D E" は却下

3. 採用: "A B C" (3トークンを1 Forward Passで)

4. 続行: "D" の正しいトークンから再開
```

### 実装

```python
class SpeculativeDecoder:
    def __init__(self, target_model, draft_model, k=5):
        self.target = target_model  # 大きい・正確
        self.draft = draft_model    # 小さい・高速
        self.k = k  # 投機的に生成するトークン数

    def generate(self, prompt, max_tokens):
        tokens = self.tokenize(prompt)
        generated = []

        while len(generated) < max_tokens:
            # 1. Draft modelでk個のトークンを投機生成
            draft_tokens = self.draft.generate(tokens, k=self.k)

            # 2. Target modelで一括検証
            # 全候補を一度に評価（1 Forward Pass）
            target_logits = self.target.forward(
                tokens + draft_tokens
            )

            # 3. 受理判定
            accepted = []
            for i, draft_token in enumerate(draft_tokens):
                draft_prob = self.draft.get_prob(draft_token)
                target_prob = self.target.get_prob_from_logits(
                    target_logits[i], draft_token
                )

                # 受理確率: min(1, target_prob / draft_prob)
                accept_prob = min(1.0, target_prob / draft_prob)

                if random.random() < accept_prob:
                    accepted.append(draft_token)
                else:
                    # 却下: このトークンから再サンプリング
                    corrected = self._resample(target_logits[i], draft_prob)
                    accepted.append(corrected)
                    break

            generated.extend(accepted)
            tokens = tokens + accepted

        return generated
```

### Speculative Decodingのバリエーション

| 手法 | Draft Model | 特徴 |
|------|-------------|------|
| **Standard** | 小さいLLM | 汎用的 |
| **Self-Speculative** | 同じモデルの量子化版 | モデル1つで完結 |
| **EAGLE** | 学習済み軽量ヘッド | 高受理率 |
| **N-gram** | N-gramマッチング | 計算コストほぼゼロ |
| **Medusa** | 複数の予測ヘッド | 並列予測 |

### 実践的な設定（vLLM）

```python
from vllm import LLM, SamplingParams

# EAGLE-3でSpeculative Decoding
llm = LLM(
    model="meta-llama/Llama-3-70B-Instruct",
    speculative_model="eagle-3-llama-70b",
    speculative_draft_tensor_parallel_size=1,
    num_speculative_tokens=5,  # k値
)

# 生成
output = llm.generate(
    "Explain quantum computing",
    SamplingParams(temperature=0.7, max_tokens=500)
)
```

**注意:** k値のチューニングが重要。不適切なkは逆効果になりうる。

---

## 量子化

### 量子化の種類

```
精度レベル:
FP32 (32bit) → FP16/BF16 (16bit) → FP8 (8bit) → INT8 (8bit) → INT4 (4bit)

メモリ削減:
100%        → 50%              → 25%       → 25%        → 12.5%
```

### 重み量子化 vs KV Cache量子化

| 対象 | 効果 | 最適なケース |
|------|------|-------------|
| **重み量子化** | モデルサイズ削減 | 短いコンテキスト |
| **KV Cache量子化** | 動的メモリ削減 | 長いコンテキスト |

```
短いコンテキスト (< 2K tokens):
- KV Cacheは小さい
- 重み量子化の効果が大きい

長いコンテキスト (> 8K tokens):
- KV Cacheがメモリの大部分を占める
- KV Cache量子化が必須
```

### 量子化手法

#### PTQ (Post-Training Quantization)

学習なしで量子化：

```python
# GPTQ (Post-Training Quantization)
from transformers import AutoModelForCausalLM

model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-3-70B-Instruct",
    device_map="auto",
    load_in_4bit=True,  # INT4量子化
    bnb_4bit_compute_dtype=torch.bfloat16,
)
```

#### AWQ (Activation-aware Weight Quantization)

活性化の重要度を考慮：

```python
# AWQは重要な重みを高精度で保持
from awq import AutoAWQForCausalLM

model = AutoAWQForCausalLM.from_quantized(
    "TheBloke/Llama-3-70B-AWQ",
    fuse_layers=True,
)
```

---

## FlashAttention

### 問題: Attention計算のメモリ効率

```
標準的なAttention:
1. Q @ K.T → Attention scores (N×N行列)
2. Softmax → Attention weights
3. Weights @ V → Output

問題: N×N行列を全てメモリに保持
→ 長いシーケンスでメモリ爆発
```

### FlashAttentionの解決策

**アイデア:** ブロック単位で計算し、HBM（高帯域メモリ）へのアクセスを最小化

```
FlashAttention:
- Q, K, V をブロックに分割
- 各ブロックをSRAM（オンチップメモリ）で処理
- 中間結果をHBMに書き戻さない
- オンラインSoftmax計算で数値安定性を確保
```

### 効果

| 指標 | 標準Attention | FlashAttention |
|------|--------------|----------------|
| メモリ使用量 | O(N²) | O(N) |
| 速度 | baseline | 2-4x 高速 |
| 長いシーケンス | 不可能 | 可能 |

### 使用方法

```python
# PyTorch 2.0+で標準サポート
import torch

# SDPAを有効化（FlashAttentionを自動選択）
with torch.backends.cuda.sdp_kernel(
    enable_flash=True,
    enable_math=False,
    enable_mem_efficient=False
):
    output = F.scaled_dot_product_attention(q, k, v)
```

---

## Continuous Batching

### 問題: 静的バッチングの非効率

```
静的バッチング:
Request 1: "Hello" → [生成中...100トークン]
Request 2: "Hi"    → [生成中...50トークン] → [待機...] → [待機...]
Request 3: "Hey"   → [生成中...30トークン] → [待機...] → [待機...]

→ 短いリクエストが長いリクエストを待つ
```

### Continuous Batching

```
Continuous Batching:
Step 1: [R1, R2, R3] → 全て処理
Step 2: [R1, R2, R3] → R3完了 → 空きスロット
Step 3: [R1, R2, R4] → 新リクエストR4を即座に投入
Step 4: [R1, R4, R5] → R2完了 → R5を投入
...

→ GPUが常にフル稼働
```

### 実装（概念）

```python
class ContinuousBatcher:
    def __init__(self, model, max_batch_size):
        self.model = model
        self.max_batch = max_batch_size
        self.active_requests = []
        self.pending_queue = []

    async def process_iteration(self):
        # 1. 完了したリクエストを除去
        self.active_requests = [
            r for r in self.active_requests
            if not r.is_complete
        ]

        # 2. 空きスロットに新リクエストを追加
        while (len(self.active_requests) < self.max_batch
               and self.pending_queue):
            new_req = self.pending_queue.pop(0)
            self.active_requests.append(new_req)

        # 3. バッチ処理
        if self.active_requests:
            batch = self._prepare_batch(self.active_requests)
            outputs = self.model.forward(batch)
            self._update_requests(outputs)
```

---

## 組み合わせ戦略

### 2026年の最適構成

```python
# vLLMでの最適化フルスタック
from vllm import LLM, SamplingParams

llm = LLM(
    model="meta-llama/Llama-3-70B-Instruct",

    # 量子化
    quantization="awq",
    kv_cache_dtype="fp8_e4m3",

    # Speculative Decoding
    speculative_model="eagle-3-llama-70b",
    num_speculative_tokens=5,

    # メモリ最適化
    enable_prefix_caching=True,
    max_num_seqs=256,  # Continuous Batching

    # GPU設定
    tensor_parallel_size=4,
    gpu_memory_utilization=0.95,
)
```

### 最適化の優先順位

```
1. KV Cache量子化 (FP8)
   → 長いコンテキストで最も効果的

2. FlashAttention
   → 標準で有効化（PyTorch 2.0+）

3. Continuous Batching
   → スループット向上

4. Speculative Decoding
   → レイテンシ改善（要チューニング）

5. 重み量子化 (INT4/AWQ)
   → メモリ制約がある場合
```

---

## まとめ

### 最適化技術マトリックス

| 技術 | メモリ削減 | 速度向上 | 精度影響 |
|------|-----------|----------|----------|
| KV Cache量子化 | ◎ | ○ | △ |
| PagedAttention | ○ | - | なし |
| Speculative Decoding | - | ◎ | なし |
| FlashAttention | ◎ | ◎ | なし |
| Continuous Batching | - | ◎ | なし |
| 重み量子化 (INT4) | ◎ | ○ | △ |

### コンテキスト長別推奨

| コンテキスト長 | 推奨最適化 |
|---------------|-----------|
| < 2K | 重み量子化 + FlashAttention |
| 2K - 8K | 上記 + KV Cache量子化 |
| 8K - 32K | 上記 + PagedAttention |
| > 32K | 上記 + Prefix Caching |

---

## 参考リンク

- [vLLM Documentation](https://docs.vllm.ai/)
- [KV Cache Compression Survey (arXiv)](https://arxiv.org/html/2508.06297v1)
- [NVIDIA - NVFP4 KV Cache](https://developer.nvidia.com/blog/optimizing-inference-for-long-context-and-large-batch-sizes-with-nvfp4-kv-cache)
- [FlashAttention Paper](https://arxiv.org/abs/2205.14135)

---

**あなたのLLM推論環境ではどんな最適化をしていますか？コメントで教えてください！**
