---
title: "【完全解説】JitRL：勾配更新なしでLLMエージェントを継続学習させる革命的手法のコード解析"
emoji: "🧪"
type: "tech"
topics: ["ai", "llm", "強化学習", "継続学習", "python"]
published: true
---

**「LLMを学習させずに、学習させる」**

これは矛盾しているように聞こえるが、2026年1月26日に公開された[JitRL（Just-In-Time Reinforcement Learning）](https://github.com/liushiliushi/JitRL)はまさにこれを実現した。

この記事では、JitRLのGitHubリポジトリを完全解析し、**Policy Tripletとは何か**、**どのようにして勾配更新なしで継続学習を実現しているのか**を、コードレベルで深掘りする。

## 結論から言うと

:::message
**JitRL** = LLMのパラメータを一切更新せずに、過去の経験を**非パラメトリックメモリ**に保存し、推論時に類似経験を検索して**ロジット（出力確率）を調整**することで、継続学習を実現する手法。

**Policy Triplet** = `<state, action, reward>` の3つ組。これをメモリに保存し、類似状況で過去のaction-rewardを参照してポリシーを改善する。
:::

---

## 1. 従来の強化学習 vs JitRL：何が違うのか？

### 従来のRL（Reinforcement Learning）

```
経験収集 → 勾配計算 → パラメータ更新 → 古い知識が壊れる（破滅的忘却）
```

### JitRL

```
経験収集 → メモリに保存 → 推論時に検索 → ロジット調整 → パラメータは変わらない
```

**決定的な違い**：JitRLはLLMの重みを**一切変更しない**。代わりに、過去の経験を外部メモリに保存し、推論時にそのメモリから関連する経験を検索して、出力の確率分布を調整する。

---

## 2. Policy Tripletとは何か？

### 数学的定義

Policy Tripletは強化学習の基本単位で、以下の3要素から構成される：

```python
triplet = {
    'state': str,      # 現在の状態（環境の観測）
    'action': str,     # エージェントが取った行動
    'reward': float    # その行動に対する報酬
}
```

JitRLでは、これを拡張した**Step Metadata**として保存する：

```python
# cross_episode_memory.py より
step_metadata = {
    'episode_timestamp': episode_data.get('timestamp'),
    'episode_number': self.current_episode_number,
    'step_data': step_data.copy(),  # state, action, rewardを含む
    'episode_final_score': episode_data.get('final_score'),
    'episode_success': episode_data.get('success'),
    'trajectory_context': trajectory_context,  # 履歴サマリー
    'current_env_info': current_env_info,      # 現在の環境情報
    'future_rewards': future_rewards           # 将来の報酬系列（割引計算用）
}
```

### なぜTripletが重要なのか？

Policy Tripletは「**この状態で、この行動を取ったら、この報酬が得られた**」という経験の記録だ。

従来のRLではこの経験を使って**パラメータを更新**するが、JitRLでは**メモリに保存して推論時に参照**する。

---

## 3. JitRLのアーキテクチャ詳解

### リポジトリ構造

```
JitRL/
├── Jericho/                          # テキストアドベンチャーゲーム用
│   └── src/
│       ├── memory_agent.py           # メインエージェント（850行）
│       ├── cross_episode_memory.py   # コアメモリシステム（900行）
│       └── utils.py                  # LLMユーティリティ（1150行）
│
└── WebArena/                         # Web自動化用
    └── memory_agents/
        └── utils/
            └── cross_episode_memory.py  # メモリシステム（1200行）
```

### 核心：CrossEpisodeMemoryクラス

```python
class CrossEpisodeMemory:
    def __init__(self, base_dir: str, gamma: float = 0.95, ...):
        self.gamma = gamma  # 割引率

        # デュアルFaissインデックス（履歴 + 状態）
        self.history_index = faiss.IndexFlatIP(self.vector_dim)
        self.state_index = faiss.IndexFlatIP(self.vector_dim)

        # メタデータ保存
        self.step_metadata = []
```

**ポイント**：
- **デュアルインデックス**：履歴ベクトルと現在状態ベクトルを別々に管理
- **Faiss**：高速な類似度検索ライブラリ
- **gamma（割引率）**：将来の報酬をどれだけ重視するか

---

## 4. 経験の保存：Tripletからベクトルへ

### ステップ1：状態のエンコード

```python
def _encode_trajectory_context(self, states, actions, current_step, current_summary, info=None):
    # LLMで履歴サマリーを生成
    trajectory_context, current_env_info = generate_trajectory_context_for_vector(
        states=states,
        actions=actions,
        earlier_summary=earlier_summary,
        current_step=current_step,
        episode_number=self.current_episode_number,
        llm_model=self.llm_model,
        temperature=0.8,
        max_tokens=1000,
        info=info
    )

    # 履歴サマリーをベクトル化（OpenAI Embedding）
    history_vector = get_embedding_with_retries(current_summary, model=self.embedding_model)

    # 現在状態をベクトル化
    current_state_text = f"step {current_step}: State: {current_env_info}"
    state_vector = get_embedding_with_retries(current_state_text, model=self.embedding_model)

    # 正規化（コサイン類似度用）
    history_vector = history_vector / (np.linalg.norm(history_vector) + 1e-8)
    state_vector = state_vector / (np.linalg.norm(state_vector) + 1e-8)

    return trajectory_context, current_env_info, history_vector, state_vector
```

### ステップ2：メモリへの保存

```python
def _store_step_in_vector_db(self, states, actions, step_index, episode_data):
    # 将来の報酬系列を抽出（割引報酬計算用）
    future_rewards = []
    for u in range(step_index, len(steps)):
        future_rewards.append(steps[u].get('llm_step_score', 0))

    # メタデータを作成
    step_metadata = {
        'step_data': step_data.copy(),
        'trajectory_context': trajectory_context,
        'current_env_info': current_env_info,
        'future_rewards': future_rewards  # ★重要：割引報酬計算に使用
    }

    # デュアルFaissインデックスに追加
    self.history_index.add(history_vector.reshape(1, -1))
    self.state_index.add(state_vector.reshape(1, -1))
    self.step_metadata.append(step_metadata)
```

---

## 5. 経験の検索：類似状況の発見

### デュアルベクトル検索

```python
def retrieve_similar_with_vector(self, game_history, current_state, current_summary, k=3, r=0.7, info=None):
    # 現在の状態をデュアルベクトルにエンコード
    trajectory_context, current_env_info, query_history_vec, query_state_vec = \
        self._encode_trajectory_context(query_states, query_actions, len(query_states)-1, current_summary, info)

    # 両方のインデックスで検索
    recall_size = min(max(k * 10, 100), self.history_index.ntotal)
    history_scores, history_indices = self.history_index.search(query_history_vec.reshape(1,-1), recall_size)
    state_scores, state_indices = self.state_index.search(query_state_vec.reshape(1,-1), recall_size)
```

### Jaccard類似度による精緻化

ベクトル検索だけでなく、**N-gram Jaccard類似度**も使用：

```python
def _jaccard(self, a_tokens, b_tokens, ngram=3):
    # N-gramを生成
    a_ngrams = self._get_ngrams(a_tokens, ngram)
    b_ngrams = self._get_ngrams(b_tokens, ngram)

    # 多重集合でJaccard係数を計算
    from collections import Counter
    counter_a = Counter(a_ngrams)
    counter_b = Counter(b_ngrams)

    inter = sum((counter_a & counter_b).values())
    union = sum((counter_a | counter_b).values())

    return inter / union if union > 0 else 0.0
```

### 最終類似度スコア

```python
# 履歴類似度（1-gram）と状態類似度（4-gram）の加重平均
sim1 = self._jaccard(query_history_tokens, history_tokens, ngram=1)
sim2 = self._jaccard(query_current_tokens, current_tokens, ngram=4)
similarity = sim1 * 0.3 + sim2 * 0.7  # 現在状態を重視
```

---

## 6. ロジット調整：JitRLの核心

### Advantage（アドバンテージ）の計算

検索で見つかった類似経験から、各行動の**割引報酬**を計算：

```python
# memory_agent.py より
def update_scores(self, state_node, options_with_logits, k, r, memory_text, info=None):
    nearest_trajectories = self.cross_mem.retrieve_similar(...)

    # 行動ごとの報酬を集計
    action_rewards = {}
    for sim, discounted_return, result_dict in nearest_trajectories:
        action = result_dict.get('action', '').strip()
        discounted_reward = result_dict.get('discounted_reward', 0)
        if action not in action_rewards:
            action_rewards[action] = []
        action_rewards[action].append(discounted_reward)

    # 行動ごとの平均報酬
    action_avg_rewards = {}
    for action, rewards in action_rewards.items():
        action_avg_rewards[action] = sum(rewards) / len(rewards)

    # 全体の平均報酬（ベースライン）
    all_rewards = []
    for rewards_list in action_rewards.values():
        all_rewards.extend(rewards_list)
    overall_avg_reward = sum(all_rewards) / len(all_rewards)

    # アドバンテージ = 行動の報酬 - ベースライン
    action_advantages = {}
    for action, avg_reward in action_avg_rewards.items():
        action_advantages[action] = avg_reward - overall_avg_reward
```

### アドバンテージの正規化

```python
# 正のアドバンテージで正規化
positive_advs = [adv for adv in adv_values if adv > 0]
if positive_advs:
    max_positive = max(positive_advs)
    normalized_advantages = {
        action: adv / max_positive
        for action, adv in action_advantages.items()
    }
```

### ロジットの調整（核心部分）

```python
# エピソードベースの重み付け（学習が進むほど経験を重視）
current_episode = self.cross_mem.current_episode_number
episode_weight = min(1.0 + (current_episode / 50.0) * 0.5, 1.5)

# 重み付き正規化アドバンテージ
weighted_normalized_advantage = normalized_advantage * episode_weight

# ★★★ ロジット調整 ★★★
# LLMの出力確率に、過去の経験から計算したアドバンテージを加算
corrected_logprob = normalized_prob + weighted_normalized_advantage
```

:::message alert
**これがJitRLの核心**：LLMのパラメータは変えずに、出力のログ確率（logit）に経験ベースのアドバンテージを**加算**することで、ポリシーを改善する。
:::

---

## 7. 数学的背景：なぜこれが機能するのか？

論文によると、このロジット加算ルールは**KL制約付きポリシー最適化問題の閉形式解**である。

### 最適化問題

```
max_π E[A(s,a)] - β × KL(π || π_ref)
```

- `π`: 新しいポリシー
- `π_ref`: 参照ポリシー（元のLLM）
- `A(s,a)`: アドバンテージ関数
- `β`: KL制約の強さ

### 閉形式解

```
log π(a|s) = log π_ref(a|s) + A(s,a) / β
```

これは**ソフトmax方程式**の一種で、JitRLのロジット加算：

```python
corrected_logprob = normalized_prob + weighted_normalized_advantage
```

と数学的に等価である。

---

## 8. 探索と活用のバランス

### 適応的探索確率

```python
def calculate_exploration_probability(self, nearest_trajectories, action_rewards):
    """
    安定した高報酬行動が存在するかどうかで探索確率を調整
    """
    for action, rewards in action_rewards.items():
        n_samples = len(rewards)
        mean_reward = np.mean(rewards)
        std_reward = np.std(rewards)

        # 変動係数（CV）: 相対的な安定性を測定
        cv = std_reward / mean_reward if mean_reward > 0 else float('inf')

        # 信頼度スコア
        performance_score = max(0, mean_reward)
        stability_score = 1.0 / (1.0 + cv)
        sample_confidence = max(n_samples / 4.0, 1)

        action_confidence = (performance_score * stability_score) ** self.exploration_rate * sample_confidence

    # 探索確率 = 信頼度の逆数
    exploration_prob = 1 / (1 + max_confidence)
    return max(0.05, min(0.8, exploration_prob))  # 0.05〜0.8にクランプ
```

**ポイント**：
- **安定した高報酬行動がある** → 探索を減らして活用
- **不確実な状況** → 探索を増やして新しい行動を試す

---

## 9. LLMによるステップスコアリング

JitRLでは、ゲームの報酬だけでなく、**LLMによる行動評価スコア**も使用：

```python
def evaluate_step_scores_with_llm(game_history, state, final_score, success, llm_model):
    sys_prompt = """You are scoring game actions to build training data for future gameplay.

    SCORING RULES:
    - Positive: Action led to progress or useful discoveries
    - Negative: Action wasted time, caused loops, or had no benefit
    """

    user_prompt = f"""Score each action in this game session.

    Final Result: {"SUCCESS" if success else "FAILURE"}, Final Score: {final_score}

    Trajectory:
    {trajectory_text}

    JSON FORMAT:
    {{
      "step_analysis": [
        {{
          "step": 0,
          "action": "exact action taken",
          "detailed_reasoning": "What happened after this action...",
          "score": 5,
        }}
      ]
    }}
    """
```

これにより、**スパース（まばら）な環境報酬**を補完する**密な報酬信号**が得られる。

---

## 10. 実験結果

### Jerichoテキストアドベンチャー

| ゲーム | ナイーブ | JitRL | 改善率 |
|--------|----------|-------|--------|
| Zork1 | 35.2 | 52.8 | **+50%** |
| Library | 18.5 | 28.3 | **+53%** |
| Detective | 180 | 265 | **+47%** |

### WebArena-Lite（Web自動化）

| モデル | メモリなし | JitRL | 改善率 |
|--------|------------|-------|--------|
| GPT-4o | 32.5% | 41.2% | **+27%** |
| Claude-3.5 | 28.8% | 38.5% | **+34%** |
| Gemini-2.5 | 25.3% | 34.7% | **+37%** |

---

## 11. JitRLを自分で試す

### インストール

```bash
git clone https://github.com/liushiliushi/JitRL.git
cd JitRL/Jericho

pip install jericho openai tiktoken numpy python-dotenv faiss-cpu
```

### 実行

```bash
# Zork1で10エピソード実行
python main.py --game_name zork1 --agent_type memory --eval_runs 10

# メモリなしのベースライン
python main.py --game_name zork1 --agent_type memory --no-enable_cross_mem
```

### 環境設定

```bash
# .env
OPENROUTER_API_KEY=your_api_key
```

---

## 12. まとめ：JitRLが開く未来

### JitRLの革新性

1. **勾配更新なし**：LLMのパラメータを一切変更しない
2. **テスト時学習**：推論時にリアルタイムでポリシーを改善
3. **破滅的忘却の回避**：経験は外部メモリに保存されるため、知識が失われない
4. **数学的根拠**：KL制約付き最適化の閉形式解

### 今後の可能性

- **Claude CodeやCursorへの応用**：プロジェクト固有の知識を蓄積
- **企業向けAIエージェント**：顧客対応の経験を蓄積して品質向上
- **自動運転**：異常状況への対応経験を蓄積

---

:::message
この記事が参考になったら、**いいね**と**保存**をお願いします！

**質問**：あなたのプロジェクトでJitRLのような経験蓄積システムを使ってみたいと思いますか？どんなユースケースを考えていますか？コメントで教えてください。
:::

## 参考文献

- [JitRL GitHub Repository](https://github.com/liushiliushi/JitRL)
- [JitRL Paper (arXiv:2601.18510)](https://www.arxiv.org/pdf/2601.18510)
- [Jericho: Text Adventure Game Framework](https://github.com/microsoft/jericho)
- [WebArena Benchmark](https://webarena.dev/)
- [Faiss: Vector Similarity Search](https://github.com/facebookresearch/faiss)
