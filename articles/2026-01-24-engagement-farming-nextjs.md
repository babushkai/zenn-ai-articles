---
title: "Next.jsでエンゲージメント爆上げUI実装ガイド - バズるサイトの作り方"
emoji: "🔥"
type: "tech"
topics: ["nextjs", "react", "typescript", "ui", "ux"]
published: false
---

## はじめに

「せっかく良いプロダクトを作ったのに、誰にも見てもらえない...」

そんな経験はありませんか？技術的に優れたサイトでも、ユーザーが「シェアしたい」「また来たい」と思わなければ成長しません。

この記事では、[RAG カタログ](https://rag-catalog.vercel.app)で実装した**エンゲージメントを爆上げするUI/UXパターン**を、実際のコードとともに解説します。

## 実装する機能一覧

| 機能 | 効果 |
|------|------|
| アニメーションカウンター | 数字の説得力UP |
| HOT/NEW/Trendingバッジ | FOMO（見逃し不安）を刺激 |
| ランキング表示 | 競争心理を活用 |
| ソーシャルシェアボタン | 拡散のハードルを下げる |
| Featured（注目）セクション | 回遊率UP |
| 人気検索ワード | 探索行動を促進 |

## 1. アニメーションカウンター

数字がカウントアップするアニメーションは、ユーザーの目を引き「すごい！」という印象を与えます。

```tsx
// components/AnimatedCounter.tsx
'use client';

import { useEffect, useRef, useState } from 'react';

interface AnimatedCounterProps {
  end: number;
  duration?: number;
  suffix?: string;
}

export default function AnimatedCounter({
  end,
  duration = 2000,
  suffix = ''
}: AnimatedCounterProps) {
  const [count, setCount] = useState(0);
  const [hasAnimated, setHasAnimated] = useState(false);
  const ref = useRef<HTMLSpanElement>(null);

  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting && !hasAnimated) {
          setHasAnimated(true);
          animateCount();
        }
      },
      { threshold: 0.1 }
    );

    if (ref.current) observer.observe(ref.current);
    return () => observer.disconnect();
  }, [hasAnimated, end, duration]);

  const animateCount = () => {
    const startTime = performance.now();
    const animate = (currentTime: number) => {
      const elapsed = currentTime - startTime;
      const progress = Math.min(elapsed / duration, 1);
      // イージング関数で自然な動きに
      const eased = 1 - Math.pow(1 - progress, 3);
      setCount(Math.floor(eased * end));
      if (progress < 1) requestAnimationFrame(animate);
    };
    requestAnimationFrame(animate);
  };

  return <span ref={ref}>{count}{suffix}</span>;
}
```

**使用例：**
```tsx
<div className="flex items-center gap-2">
  <span className="text-2xl font-bold">
    <AnimatedCounter end={45} suffix="+" />
  </span>
  <span className="text-zinc-500">ツール</span>
</div>
```

**ポイント:**
- `IntersectionObserver`で画面に入ったときだけアニメーション
- イージング関数で最後がゆっくりになる自然な動き
- 一度だけ実行（`hasAnimated`フラグ）

## 2. HOT/NEW/Trendingバッジ

バッジは**FOMO（Fear Of Missing Out）**を刺激する強力なUI要素です。

```tsx
// components/Badge.tsx
interface BadgeProps {
  type: 'hot' | 'new' | 'trending' | 'featured';
  size?: 'sm' | 'md';
}

const badgeConfig = {
  hot: {
    label: 'HOT',
    className: 'bg-gradient-to-r from-orange-500 to-red-500 text-white',
  },
  new: {
    label: 'NEW',
    className: 'bg-gradient-to-r from-green-500 to-emerald-500 text-white',
  },
  trending: {
    label: 'Trending',
    className: 'bg-gradient-to-r from-purple-500 to-pink-500 text-white',
  },
  featured: {
    label: 'Featured',
    className: 'bg-gradient-to-r from-blue-500 to-cyan-500 text-white',
  },
};

export default function Badge({ type, size = 'sm' }: BadgeProps) {
  const config = badgeConfig[type];
  const sizeClass = size === 'sm'
    ? 'px-2 py-0.5 text-xs'
    : 'px-3 py-1 text-sm';

  return (
    <span className={`
      ${config.className}
      ${sizeClass}
      font-bold rounded-full
      shadow-lg animate-pulse
    `}>
      {config.label}
    </span>
  );
}
```

**使用パターン:**

```tsx
// 人気ツールにHOTバッジ
const hotToolIds = [...catalog]
  .sort((a, b) => (b.stars || 0) - (a.stars || 0))
  .slice(0, 5)
  .map(t => t.id);

// 新着ツールにNEWバッジ
const newToolIds = catalog.slice(-5).map(t => t.id);

// 表示
{hotToolIds.includes(item.id) && <Badge type="hot" />}
{newToolIds.includes(item.id) && <Badge type="new" />}
```

**心理学的効果:**
- 🔥 HOT → 「みんなが注目している」
- ✨ NEW → 「最新情報をキャッチしたい」
- 📈 Trending → 「トレンドに乗り遅れたくない」

## 3. ランキング表示

人は「1位」「2位」という序列に自然と惹かれます。

```tsx
{results.map((item, index) => (
  <div className="relative">
    {/* トップ3にランキングバッジ */}
    {index < 3 && (
      <div className="absolute -top-2 -left-2 w-6 h-6 rounded-full
                      bg-zinc-100 text-zinc-900 text-xs font-bold
                      flex items-center justify-center z-10">
        {index + 1}
      </div>
    )}
    <ItemCard item={item} />
  </div>
))}
```

**デザインTips:**
- 1位・2位・3位で色を変える（金・銀・銅）のも効果的
- `absolute`で配置してカードに被せる
- `z-10`で確実に前面に表示

## 4. ソーシャルシェアボタン

シェアのハードルを下げることが重要です。

```tsx
// components/ShareButtons.tsx
'use client';

import { useState } from 'react';

interface ShareButtonsProps {
  title: string;
  url?: string;
  description?: string;
}

export default function ShareButtons({ title, url, description }: ShareButtonsProps) {
  const [copied, setCopied] = useState(false);

  const shareUrl = typeof window !== 'undefined'
    ? (url || window.location.href)
    : '';
  const shareText = description ? `${title} - ${description}` : title;

  const handleCopy = async () => {
    await navigator.clipboard.writeText(shareUrl);
    setCopied(true);
    setTimeout(() => setCopied(false), 2000);
  };

  const twitterUrl = `https://twitter.com/intent/tweet?text=${encodeURIComponent(shareText)}&url=${encodeURIComponent(shareUrl)}`;

  return (
    <div className="flex items-center gap-2">
      <span className="text-xs text-zinc-500">Share:</span>

      {/* X (Twitter) */}
      <a
        href={twitterUrl}
        target="_blank"
        rel="noopener noreferrer"
        className="w-8 h-8 rounded-lg bg-zinc-800 hover:bg-zinc-700
                   flex items-center justify-center transition-colors"
      >
        <XIcon className="w-4 h-4 text-zinc-400" />
      </a>

      {/* コピーボタン */}
      <button
        onClick={handleCopy}
        className="w-8 h-8 rounded-lg bg-zinc-800 hover:bg-zinc-700
                   flex items-center justify-center transition-colors"
      >
        {copied ? (
          <CheckIcon className="w-4 h-4 text-green-400" />
        ) : (
          <LinkIcon className="w-4 h-4 text-zinc-400" />
        )}
      </button>
    </div>
  );
}
```

**配置のコツ:**
- ヘッダー付近（スクロール前に目に入る）
- 検索結果ページ（「この検索結果シェアしたい」）
- 各アイテムの詳細ページ

## 5. Featured（注目）セクション

トップページに「今週の注目」を置くことで、ユーザーの滞在時間を増やせます。

```tsx
// 最もスターの多いツールを特集
const featuredTool = [...catalog]
  .sort((a, b) => (b.stars || 0) - (a.stars || 0))[0];

<section className="bg-gradient-to-r from-purple-500/5 via-transparent to-blue-500/5">
  <div className="flex items-center gap-3 mb-6">
    <Badge type="featured" size="md" />
    <h2 className="text-lg font-semibold">今週の注目ツール</h2>
  </div>

  <Link href={`/item/${featuredTool.id}`} className="block rounded-xl border p-6 hover:border-zinc-700">
    <h3 className="text-2xl font-bold">{featuredTool.name}</h3>
    <p className="text-zinc-400">{featuredTool.description}</p>
    <div className="flex items-center gap-2 mt-4">
      <span>詳細を見る</span>
      <ArrowRightIcon />
    </div>
  </Link>
</section>
```

**グラデーション背景**で視覚的に差別化することで、「特別なセクション」感を演出。

## 6. 人気検索ワード

検索ページに人気ワードを表示すると、ユーザーの探索行動を促進できます。

```tsx
const popularSearches = ['LangChain', 'vector database', 'embedding', 'evaluation'];

<div className="mt-8">
  <p className="text-sm text-zinc-500 mb-3">人気の検索ワード：</p>
  <div className="flex flex-wrap gap-2">
    {popularSearches.map((term) => (
      <button
        key={term}
        onClick={() => router.push(`/search?q=${encodeURIComponent(term)}`)}
        className="rounded-full border border-zinc-800 bg-zinc-900
                   px-4 py-2 text-sm text-zinc-400
                   hover:border-zinc-700 hover:text-zinc-300
                   transition-colors"
      >
        {term}
      </button>
    ))}
  </div>
</div>
```

## 7. OGP / Twitterカードの最適化

シェアされたときの見た目も重要です。

```tsx
// app/layout.tsx
export const metadata: Metadata = {
  title: 'RAG カタログ',
  description: '45+ RAGツールを比較・検索',
  openGraph: {
    title: 'RAG カタログ',
    description: '45+ RAGツールを比較・検索',
    url: 'https://rag-catalog.vercel.app',
    siteName: 'RAG カタログ',
    locale: 'ja_JP',
    type: 'website',
  },
  twitter: {
    card: 'summary_large_image',
    title: 'RAG カタログ',
    description: '45+ RAGツールを比較・検索',
  },
};
```

## まとめ：エンゲージメント設計の原則

| 原則 | 実装 |
|------|------|
| **数字で説得** | AnimatedCounter |
| **希少性を演出** | HOT/NEW/Featuredバッジ |
| **競争心理を活用** | ランキング表示 |
| **行動のハードルを下げる** | シェアボタン、人気検索ワード |
| **回遊を促進** | Featured、関連カテゴリー |

これらの実装を組み合わせることで、**バズりやすく、リピートされやすい**サイトを作ることができます。

## 参考

- [RAG カタログ](https://rag-catalog.vercel.app) - 実際に実装したサイト
- [GitHub](https://github.com/babushkai/ragcatalog) - ソースコード

---

いいねとストックで応援よろしくお願いします！質問はコメント欄へどうぞ。
