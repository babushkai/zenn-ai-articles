---
title: "LLM推論の全貌 - CoT/ToT/o1モデルの仕組みと実装"
emoji: "🧩"
type: "tech"
topics: ["ai", "llm", "推論", "chainofthought", "openai"]
published: false
---

**「なぜo1は数学が得意なのか？」**

答えは**Long Chain-of-Thought（長い思考連鎖）**にあります。

この記事では、LLMの推論能力を引き出す技術の全体像を解説します。基本的なCoTから、Tree of Thoughts、そしてo1/o3/DeepSeek-R1に至る進化の系譜を辿ります。

## 推論技術の進化

```
2022: Chain-of-Thought (CoT) - 「ステップバイステップで考えよう」
  ↓
2023: Tree of Thoughts (ToT) - 「複数の思考経路を探索しよう」
  ↓
2024: o1 (OpenAI) - 「強化学習で思考を最適化」
  ↓
2025: DeepSeek-R1, QwQ - 「オープンソースのLong CoT」
  ↓
2026: 長文脈推論の標準化
```

---

## Chain-of-Thought (CoT)

### 基本概念

:::message
**CoTの本質:** LLMに「考えるステップを明示的に出力させる」ことで、複雑な問題を解けるようにする
:::

#### Before CoT

```
Q: ロジャーはテニスボールを5個持っています。
   さらに2缶買いました。各缶には3個入っています。
   今、何個のボールを持っていますか？

A: 11個
```

直接答えを出すと間違いやすい。

#### After CoT

```
Q: ロジャーはテニスボールを5個持っています。
   さらに2缶買いました。各缶には3個入っています。
   今、何個のボールを持っていますか？

A: ステップバイステップで考えます。
   1. 最初に5個持っている
   2. 2缶買った、各缶に3個入り
   3. 追加: 2 × 3 = 6個
   4. 合計: 5 + 6 = 11個

   答え: 11個
```

### 実装パターン

#### Zero-shot CoT

最もシンプル。「ステップバイステップで考えてください」を追加するだけ：

```python
def zero_shot_cot(question):
    prompt = f"""
    {question}

    ステップバイステップで考えてください。
    """
    return llm.generate(prompt)
```

#### Few-shot CoT

推論例を提供：

```python
def few_shot_cot(question):
    examples = """
    Q: 駐車場に車が3台あります。さらに2台来ました。
       駐車場には今何台の車がありますか？
    A: 最初に3台ありました。さらに2台来たので、
       3 + 2 = 5台になりました。答え: 5台

    Q: レアには32個のチョコレートがあり、姉は42個持っています。
       35個食べたら、合計で何個残りますか？
    A: 最初の合計: 32 + 42 = 74個
       35個食べた後: 74 - 35 = 39個
       答え: 39個
    """

    prompt = f"{examples}\n\nQ: {question}\nA:"
    return llm.generate(prompt)
```

#### Self-Consistency

複数の推論パスを生成し、多数決で最終回答を決定：

```python
def self_consistency(question, n_paths=5):
    answers = []

    for _ in range(n_paths):
        # 温度を上げて多様な推論パスを生成
        response = llm.generate(
            prompt=cot_prompt(question),
            temperature=0.7,
        )
        answer = extract_answer(response)
        answers.append(answer)

    # 多数決
    return most_common(answers)
```

**効果:** 単一パスより10-20%精度向上（算術推論タスク）

---

## Tree of Thoughts (ToT)

### CoTの限界

CoTは**線形**。一度間違えると、修正が難しい：

```
問題: 24ゲーム（4つの数字で24を作る）
数字: 4, 5, 6, 8

CoT:
「4 + 5 = 9、9 + 6 = 15、15 + 8 = 23... ダメだ」
→ 行き詰まり
```

### ToTの解決策

**複数の思考パスを木構造で探索**：

```
                    [初期状態: 4, 5, 6, 8]
                          /    |    \
                         /     |     \
              [4+5=9]    [4×5=20]    [6-4=2]
                /           |            \
               /            |             \
         [9+6=15]      [20+8=28]       [2×8=16]
            |              ✗              |
            |                             |
        [15+8=23]                    [16+5=21]
            ✗                            ...

探索を続けて: (6-4)×(5+8) = 2×13 ✗
さらに: 4×(8-5+6) = 4×9 = 36 ✗
最終的に: (4+8)×(6-4) = 12×2 = 24 ✓
```

### 実装

```python
class TreeOfThoughts:
    def __init__(self, llm, evaluator):
        self.llm = llm
        self.evaluator = evaluator

    def solve(self, problem, max_depth=5, beam_width=3):
        # 初期状態
        root = ThoughtNode(state=problem, depth=0)
        frontier = [root]

        for depth in range(max_depth):
            candidates = []

            for node in frontier:
                # 次の思考を生成
                next_thoughts = self._generate_thoughts(node)

                for thought in next_thoughts:
                    # 評価
                    score = self._evaluate_thought(thought, problem)
                    child = ThoughtNode(
                        state=thought,
                        parent=node,
                        depth=depth + 1,
                        score=score,
                    )
                    candidates.append(child)

                    # 解が見つかったか確認
                    if self._is_solution(child, problem):
                        return self._extract_path(child)

            # ビームサーチ: 上位のみ保持
            frontier = sorted(candidates, key=lambda x: x.score, reverse=True)[:beam_width]

        return None  # 解が見つからなかった

    def _generate_thoughts(self, node):
        """次の可能な思考ステップを生成"""
        prompt = f"""
        現在の状態: {node.state}

        次に取りうるステップを3つ提案してください。
        各ステップは独立した選択肢です。
        """
        response = self.llm.generate(prompt)
        return self._parse_thoughts(response)

    def _evaluate_thought(self, thought, problem):
        """思考の有望さを評価 (0-1)"""
        prompt = f"""
        問題: {problem}
        現在の思考: {thought}

        この思考が解決に向かっている可能性を0-1で評価してください。
        """
        response = self.llm.generate(prompt)
        return float(response)
```

### ToTのバリエーション

| 手法 | 探索戦略 | 特徴 |
|------|----------|------|
| BFS-ToT | 幅優先 | 浅い解を優先 |
| DFS-ToT | 深さ優先 | メモリ効率が良い |
| Beam-ToT | ビームサーチ | バランス型 |
| A*-ToT | ヒューリスティック | 評価関数が鍵 |

---

## o1モデル: Long Chain-of-Thought

### o1の革新

:::message
**o1の本質:** 強化学習（RL）でCoTを最適化し、「考える時間」を動的に調整
:::

従来のCoTとの違い：

| 項目 | 従来CoT | o1 |
|------|---------|-----|
| 思考の長さ | 固定的 | 問題に応じて可変 |
| 学習方法 | プロンプト | 強化学習 |
| 自己修正 | 限定的 | 積極的 |
| 計算コスト | 低 | 高（思考に時間をかける） |

### o1の動作原理

```
入力: 複雑な数学問題

o1の内部処理:
1. 問題の難易度を暗黙的に評価
2. 必要な「思考トークン」数を決定
3. 推論チェーンを生成
4. 各ステップで自己評価・修正
5. 行き詰まったら別アプローチを試行
6. 最終回答を出力
```

### 実装イメージ（擬似コード）

```python
class ReasoningModel:
    """o1スタイルの推論モデル（概念的な実装）"""

    def __init__(self, base_model, reward_model):
        self.model = base_model
        self.reward = reward_model

    def reason(self, problem, max_thinking_tokens=10000):
        thinking_chain = []
        current_state = problem

        while len(thinking_chain) < max_thinking_tokens:
            # 次の思考ステップを生成
            next_thought = self._generate_step(current_state, thinking_chain)

            # 自己評価
            evaluation = self._self_evaluate(next_thought, problem)

            if evaluation.is_mistake:
                # 間違いを認識したら修正
                correction = self._correct(next_thought, evaluation.reason)
                thinking_chain.append(f"Wait, that's wrong. {correction}")
                current_state = correction
            elif evaluation.is_stuck:
                # 行き詰まったら別アプローチ
                alternative = self._try_alternative(problem, thinking_chain)
                thinking_chain.append(f"Let me try a different approach. {alternative}")
                current_state = alternative
            elif evaluation.is_solution:
                # 解に到達
                break
            else:
                # 続行
                thinking_chain.append(next_thought)
                current_state = next_thought

        return self._extract_final_answer(thinking_chain)

    def _self_evaluate(self, thought, problem):
        """自己評価: 間違い検出、行き詰まり検出、解到達判定"""
        prompt = f"""
        問題: {problem}
        現在の思考: {thought}

        評価してください:
        1. この推論に間違いはありますか？
        2. 行き詰まっていますか？
        3. 解に到達しましたか？
        """
        return self.model.generate(prompt, response_format=Evaluation)
```

### o1 vs o3 vs o4

| モデル | 特徴 | ベンチマーク |
|--------|------|-------------|
| o1 | 初代推論モデル | AIME: 74% |
| o3 | より深い推論、効率改善 | AIME: 87% |
| o4 (予想) | マルチモーダル推論 | - |

---

## DeepSeek-R1: オープンソースの衝撃

### なぜ重要か

:::message
**DeepSeek-R1の意義:** o1レベルの推論能力を**オープンソース**で実現
:::

### アーキテクチャの特徴

```python
# DeepSeek-R1の推論プロセス（概念）
class DeepSeekR1:
    def reason(self, problem):
        # 1. 長い思考チェーンを生成
        thinking = self._long_cot(problem)

        # 2. 思考の中で自己検証
        # "<think>" タグ内で推論、"</think>" 後に回答
        response = f"""
        <think>
        {thinking}
        </think>

        Based on my reasoning above, the answer is: {self._extract_answer(thinking)}
        """
        return response
```

### Long CoTの特徴

DeepSeek-R1やQwQなどの特徴：

1. **深い推論深度**: 数千トークンの思考チェーン
2. **広範な探索**: 複数の解法を試行
3. **実現可能性の反省**: 各ステップで「これは正しいか？」を確認

```
例: 複雑な証明問題

<think>
まず問題を理解しよう。
条件は...

アプローチ1を試してみる:
...
うーん、これだと矛盾が生じる。

別のアプローチ2:
...
これは有望だ。続けてみよう。
...
待って、ここで仮定が間違っている。

修正して:
...
よし、これで証明できた。
</think>

証明: [最終的な証明]
```

---

## 実践: 推論能力を引き出すプロンプト

### レベル別プロンプト

#### Level 1: 基本CoT

```
問題を解いてください。ステップバイステップで考えてください。

{problem}
```

#### Level 2: 構造化CoT

```
以下の問題を解いてください。

問題: {problem}

以下の形式で回答してください:
1. 問題の理解: [問題を自分の言葉で言い換え]
2. 必要な情報: [問題から抽出した情報]
3. 解法の選択: [どのアプローチを使うか]
4. ステップバイステップの解答:
   - Step 1: ...
   - Step 2: ...
5. 検証: [答えが正しいか確認]
6. 最終回答: [答え]
```

#### Level 3: 自己批判CoT

```
問題: {problem}

以下のプロセスで解いてください:

1. 最初の解答を試みる
2. その解答を批判的に検証する
3. 間違いがあれば修正する
4. 別のアプローチも検討する
5. 最も確実な解答を選ぶ

考えるプロセスをすべて出力してください。
```

#### Level 4: Think Hard（拡張思考）

```
この問題は非常に難しいです。十分な時間をかけて、
深く考えてから回答してください。

問題: {problem}

think hard about this problem.
```

### Claude/GPTでの使い分け

| モデル | 推奨プロンプト |
|--------|---------------|
| Claude Opus 4.5 | 「think」「think hard」「ultrathink」 |
| GPT-4o | 構造化CoT |
| o1/o3 | シンプルな問題文（内部で自動推論） |
| DeepSeek-R1 | シンプルな問題文 |

---

## 推論モデルの限界

### 注意点

1. **コスト**: 思考トークンも課金対象
2. **レイテンシ**: 深い推論は時間がかかる
3. **万能ではない**: 知識の欠如は推論で補えない
4. **脆さ**: 推論が正しく「見える」だけで、論理的に正しいとは限らない

### いつ使うべきか

| 推論が有効 | 推論が不要 |
|-----------|-----------|
| 数学的問題 | 事実の検索 |
| 論理パズル | 翻訳 |
| コード生成 | 要約 |
| 計画立案 | 分類 |
| 複雑な分析 | 単純なQ&A |

---

## まとめ

```
推論技術の使い分け:

シンプルな問題 → Zero-shot CoT
中程度の問題 → Few-shot CoT + Self-Consistency
複雑な問題 → Tree of Thoughts
非常に複雑 → o1/o3/DeepSeek-R1
```

### 重要ポイント

1. **CoTは基本**: 「ステップバイステップで」は今でも有効
2. **ToTは探索**: 複数の可能性を検討する必要がある問題に
3. **o1/R1は自動化**: 推論プロセス自体を最適化
4. **コストとのトレードオフ**: 深い推論 = 高コスト

---

## 参考リンク

- [Chain-of-Thought Prompting (arXiv)](https://arxiv.org/abs/2201.11903)
- [Tree of Thoughts | Prompt Engineering Guide](https://www.promptingguide.ai/techniques/tot)
- [Learning to Reason with LLMs | OpenAI](https://openai.com/index/learning-to-reason-with-llms/)
- [Awesome LLM Reasoning (GitHub)](https://github.com/atfortes/Awesome-LLM-Reasoning)

---

**推論技術を活用した面白いユースケースがあれば、コメントで教えてください！**
