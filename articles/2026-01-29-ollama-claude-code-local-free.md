---
title: "【衝撃】Claude Codeが無料で動く。Ollamaがやってくれた。"
emoji: "🦙"
type: "tech"
topics: ["ollama", "claudecode", "ローカルllm", "無料", "ai"]
published: true
---

**待って。**

**これマジで言ってる？**

Ollamaの最新アップデート（v0.14.0）で、**Claude CodeがローカルLLMで動くようになった。**

つまり：

:::message alert
**月額数万円のAPI費用 → 0円**
**クラウド依存 → 完全ローカル**
**データ送信 → 一切なし**
:::

「いやいや、ローカルモデルじゃClaude Codeの機能使えないでしょ」

**使えます。ツール呼び出しも、ビジョンも、拡張思考も。**

この記事では、セットアップから実践的な使い方まで、**完全に無料でClaude Code体験を手に入れる方法**を解説します。

## 🔥 何が起きたのか

### Ollamaがやったこと

```
┌─────────────────────────────────────────────────────────┐
│                    Ollama v0.14.0                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  新機能: Anthropic Messages API 互換レイヤー             │
│                                                         │
│  ┌─────────────┐      ┌─────────────┐                  │
│  │ Claude Code │ ──── │   Ollama    │                  │
│  │  (クライアント) │      │  (APIサーバー)  │                  │
│  └─────────────┘      └─────────────┘                  │
│         │                    │                          │
│         │  Anthropic API     │  ローカルLLM             │
│         │  形式のリクエスト    │  で処理                  │
│         └────────────────────┘                          │
│                                                         │
│  Claude Codeは「Anthropic APIと話してる」と思ってる       │
│  実際は「ローカルのOllamaと話してる」                      │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**要するに、OllamaがAnthropicのふりをしてくれる。**

### なぜこれが革命的なのか

| 項目 | Anthropic API | Ollama + ローカル |
|------|--------------|------------------|
| 月額コスト | $20〜$200+ | **$0** |
| データプライバシー | クラウド送信 | **完全ローカル** |
| オフライン動作 | 不可 | **可能** |
| レート制限 | あり | **なし** |
| カスタマイズ | 不可 | **自由自在** |

## 🛠️ 5分でセットアップ

### Step 1: Ollamaのインストール

```bash
# macOS / Linux / WSL
curl -fsSL https://ollama.com/install.sh | bash

# Windows PowerShell
irm https://ollama.com/install.ps1 | iex
```

### Step 2: 推奨モデルのダウンロード

```bash
# ローカル実行用（推奨）
ollama pull gpt-oss:20b

# または、コーディング特化
ollama pull qwen3-coder
```

:::message
**重要**: 32K以上のコンテキスト長を持つモデルを推奨。これより短いと、Claude Codeの複雑なタスクで問題が発生する可能性あり。
:::

### Step 3: 環境変数の設定

```bash
# bashrc / zshrc に追加
export ANTHROPIC_AUTH_TOKEN=ollama
export ANTHROPIC_BASE_URL=http://localhost:11434
```

**これだけ。**

### Step 4: Claude Codeを起動

```bash
# モデルを指定して起動
claude --model gpt-oss:20b
```

**おめでとうございます。あなたは今、無料でClaude Codeを使っています。**

## 📊 推奨モデル完全ガイド

### ローカル実行向け

| モデル | パラメータ | VRAM要件 | 特徴 |
|--------|-----------|---------|------|
| **gpt-oss:20b** | 20B | 16GB+ | バランス型、汎用性高い |
| **qwen3-coder** | 32B | 24GB+ | コーディング特化、精度高い |
| **deepseek-coder-v2** | 16B | 12GB+ | コスパ最強 |
| **codestral** | 22B | 16GB+ | Mistral製、高速 |

### クラウド経由（Ollamaクラウド）

```bash
# クラウドモデルを使う場合
claude --model glm-4.7:cloud
claude --model minimax-m2.1:cloud
```

| モデル | 特徴 | コスト |
|--------|------|--------|
| **glm-4.7:cloud** | 128K context、高精度 | 従量課金（安い） |
| **minimax-m2.1:cloud** | 超長文対応 | 従量課金（安い） |

:::message
クラウドモデルはAnthropicの1/10以下の価格で利用可能
:::

## 🎯 実際に動く機能

### 1. ツール呼び出し（Function Calling）

```python
# Ollama経由でもツール呼び出しが動作
from anthropic import Anthropic

client = Anthropic(
    api_key="ollama",
    base_url="http://localhost:11434"
)

response = client.messages.create(
    model="gpt-oss:20b",
    max_tokens=1024,
    tools=[{
        "name": "get_weather",
        "description": "Get current weather for a location",
        "input_schema": {
            "type": "object",
            "properties": {
                "location": {
                    "type": "string",
                    "description": "City name"
                }
            },
            "required": ["location"]
        }
    }],
    messages=[{
        "role": "user",
        "content": "東京の天気を教えて"
    }]
)
```

**Claude Codeのファイル編集、コマンド実行、検索機能がすべて動作。**

### 2. ビジョン（画像入力）

```bash
# 画像を含むタスクも対応
claude --model llava:34b

> このスクリーンショットのUIを実装して
```

### 3. 拡張思考（Extended Thinking）

```bash
# 複雑な推論タスク
claude --model qwen3-coder

> このアーキテクチャの問題点を分析して、
> 改善案を3つ提案して
```

### 4. マルチターン会話 + ストリーミング

```python
# ストリーミングレスポンス
with client.messages.stream(
    model="gpt-oss:20b",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

## 💻 実践的なユースケース

### ケース1: 完全オフライン開発

```
シチュエーション:
- 飛行機の中で8時間のフライト
- 機密性の高いコードを扱う
- インターネット接続なし

解決策:
1. 事前にモデルをダウンロード
2. Ollamaをローカルで起動
3. Claude Codeを使って開発

結果: 8時間、完全オフラインでAIペアプログラミング
```

### ケース2: 機密データの処理

```
シチュエーション:
- 顧客データを含むコードベース
- クラウドに送信できない
- でもAI支援は欲しい

解決策:
export ANTHROPIC_BASE_URL=http://localhost:11434

結果: 一切のデータがローカルから出ない
```

### ケース3: 無制限の実験

```
シチュエーション:
- 新しいプロンプト手法を100回試したい
- API費用が気になる
- レート制限に引っかかる

解決策:
Ollamaローカル実行

結果: 何回実行しても$0、レート制限なし
```

## ⚠️ 正直に言う：制限事項

### 性能差は存在する

```
┌─────────────────────────────────────────────────────────┐
│              性能比較（正直なやつ）                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  タスク          │ Claude 3.5 │ gpt-oss:20b │ 差     │
│  ────────────────┼────────────┼─────────────┼───────  │
│  簡単なコード生成  │    95%     │    90%      │  -5%   │
│  複雑なリファクタ  │    90%     │    75%      │ -15%   │
│  アーキテクチャ設計│    95%     │    70%      │ -25%   │
│  バグ修正        │    92%     │    85%      │  -7%   │
│                                                         │
│  ※ 体感値。タスクとモデルにより大きく変動               │
└─────────────────────────────────────────────────────────┘
```

**複雑なタスクほど差が出る。これは事実。**

### ハードウェア要件

| モデルサイズ | 必要VRAM | 推奨GPU |
|------------|---------|---------|
| 7B | 8GB | RTX 3070以上 |
| 14B | 12GB | RTX 3080以上 |
| 20B | 16GB | RTX 4080以上 |
| 32B+ | 24GB+ | RTX 4090 / A100 |

**GPUがなければ、CPUでも動くが遅い（10-30トークン/秒）。**

### 一部機能の互換性

```
✅ 完全動作
- ファイル読み書き
- Bash実行
- 基本的なコード生成
- 会話の継続

⚠️ モデル依存
- ツール呼び出しの精度
- 複雑な推論
- 長文コンテキストの維持

❌ 非対応（現時点）
- Anthropicの最新機能（即座の反映なし）
- Computer Use（特殊なセットアップ必要）
```

## 🔧 プロTips

### 1. コンテキスト長を最大化

```bash
# Modelfileでコンテキスト長を指定
FROM gpt-oss:20b
PARAMETER num_ctx 65536
```

### 2. システムプロンプトのカスタマイズ

```bash
# カスタムシステムプロンプトを設定
FROM qwen3-coder
SYSTEM """
あなたは経験豊富なシニアエンジニアです。
コードは常に以下の原則に従います：
- 可読性を最優先
- エラーハンドリングを怠らない
- テストを意識した設計
"""
```

### 3. 複数モデルの使い分け

```bash
# タスクに応じてモデルを切り替え
alias claude-fast='claude --model deepseek-coder-v2'
alias claude-smart='claude --model qwen3-coder'
alias claude-vision='claude --model llava:34b'
```

### 4. GPU メモリ最適化

```bash
# メモリ使用量を制限
OLLAMA_MAX_LOADED_MODELS=1 ollama serve
```

## 📈 コスト比較：1ヶ月使った場合

### ヘビーユーザー（1日10万トークン）

| 方式 | 月間コスト |
|------|-----------|
| Claude API（Sonnet） | **$90〜$150** |
| Claude API（Opus） | **$450〜$750** |
| Ollama ローカル | **$0**（電気代のみ） |
| Ollama クラウド | **$5〜$15** |

### 年間で考えると

```
Claude API（Sonnet平均）: $120/月 × 12 = $1,440/年
Ollamaローカル: $0/年

差額: 約21万円/年
```

:::message
**21万円あったら、RTX 4080が買える。そしてそれで永久に無料。**
:::

## 🎯 結論：誰が使うべきか

### Ollamaローカルが向いている人

```
✅ プライバシーを重視する
✅ オフライン環境で作業する
✅ コストを最小化したい
✅ 実験を大量に回したい
✅ GPUを持っている（または買う予定）
✅ 「まあまあの性能」で十分
```

### Anthropic APIを使い続けるべき人

```
✅ 最高精度が必要
✅ 複雑なアーキテクチャ設計が多い
✅ GPUを持っていない
✅ セットアップに時間をかけたくない
✅ 企業のサポートが必要
```

## 🚀 今すぐ試す

```bash
# 1. Ollamaインストール
curl -fsSL https://ollama.com/install.sh | bash

# 2. モデルダウンロード
ollama pull gpt-oss:20b

# 3. 環境変数設定
export ANTHROPIC_AUTH_TOKEN=ollama
export ANTHROPIC_BASE_URL=http://localhost:11434

# 4. 起動
claude --model gpt-oss:20b
```

**5分後、あなたは無料でClaude Codeを使っている。**

---

Ollamaチームに感謝。

これで、AIコーディングアシスタントは**「持てる者だけの特権」から「全員のツール」**になった。

2026年、AIの民主化がまた一歩進んだ。

---

## 参考リンク

- [Ollama Blog: Claude](https://ollama.com/blog/claude)
- [Ollama GitHub](https://github.com/ollama/ollama)
- [Anthropic Messages API Reference](https://docs.anthropic.com/claude/reference/messages)
