---
title: SvelteKit で OGP 画像を自動生成する
tags: SvelteKit, OGP, Satori, Sharp, Google Font API
author: ekusiadadus
slide: false
---

# SvelteKit で OGP 画像を自動生成する

![OGP Image](/images/sveltekit_ogp.png)

## はじめに

最近、[ビジネスアイデアラボ](https://new-giants.breakai.ai) というアプリを作りました。このアプリでは、AI がアイデアを評価したり、市場調査・競合調査をしてくれます。各ポストにはユニークな URL が振られており、内容も異なります。これらのポストを Facebook や X などの外部サイトに公開する際に、内容に基づいた OGP（Open Graph Protocol）画像があれば理想的です。

しかし、ポスト毎に静的な画像を作成するのは手間がかかります。そこで、自動で OGP 画像を生成する方法を探りました。この記事では、SvelteKit で OGP 画像を自動生成する方法について解説します。

## 画像生成の流れ

1. 画像生成コードの作成
2. 画像生成 API の作成
3. 各コンテンツの OGP として設定

## 1. 画像生成コードの作成

画像生成には以下のライブラリを使用します：

- satori
- sharp
- Google Font API

Vercel 製の satori を選択し、sharp は画像処理に使用します。フォントは Google Font API を利用します。

<details>
<summary>画像生成コード（全体）</summary>

```typescript
// src/lib/generateOGPImage.ts
import satori, { type SatoriOptions } from "satori";
import sharp from "sharp";

export const generateOgpImage = async (
  title: string,
  width: number,
  height: number
) => {
  if (!process.env.GOOGLE_FONTS_API_KEY) {
    throw new Error("GOOGLE_FONTS_API_KEY is not set");
  }
  // Fetch Google Font
  const endpoint = new URL("https://www.googleapis.com/webfonts/v1/webfonts");
  endpoint.searchParams.set("family", "Noto Sans JP");
  endpoint.searchParams.set("key", process.env.GOOGLE_FONTS_API_KEY ?? "");

  try {
    const response = await fetch(endpoint);
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const info = await response.json();
    console.log("API Response:", JSON.stringify(info, null, 2));

    if (!info.items || info.items.length === 0) {
      throw new Error("No font items found in the API response");
    }

    const fontItem = info.items[0];
    if (!fontItem.files || !fontItem.files.regular) {
      throw new Error("Regular font file not found in the API response");
    }

    const fontUrl = fontItem.files.regular;
    const fontResponse = await fetch(fontUrl, { cache: "no-cache" });
    if (!fontResponse.ok) {
      throw new Error(`Failed to fetch font file: ${fontResponse.status}`);
    }
    const fontBuffer = await fontResponse.arrayBuffer();

    const options: SatoriOptions = {
      width,
      height,
      fonts: [
        {
          name: "Noto Sans JP",
          data: fontBuffer,
          weight: 400,
          style: "normal",
        },
      ],
    };

    const svg = await satori(
      {
        type: "div",
        props: {
          style: {
            height: "100%",
            width: "100%",
            display: "flex",
            flexDirection: "column",
            alignItems: "center",
            justifyContent: "center",
            backgroundColor: "#0ea5e9",
            backgroundImage:
              "linear-gradient(to bottom right, #0ea5e9, #a855f7, #ec4899)",
            fontSize: 64,
            fontWeight: 700,
            color: "white",
            textAlign: "center",
            padding: "40px",
          },
          children: [
            {
              type: "div",
              props: {
                children: title,
                style: {
                  textShadow: "0 2px 4px rgba(0,0,0,0.2)",
                },
              },
            },
          ],
        },
      },
      options
    );

    const png = await sharp(Buffer.from(svg)).png().toBuffer();
    return png;
  } catch (error) {
    console.error("Error in generateOgpImage:", error);
    throw error;
  }
};
```

</details>

## 2. 画像生成 API の作成

`/posts/ogp/[title].png` というパスで API を準備します。Twitter と Facebook で推奨サイズが異なるため、`/ogp/small/[title].png` と `/ogp/large/[title].png` のように分けるのも良いでしょう。

<details>
<summary>画像生成API（全体）</summary>

```typescript
// src/routes/posts/ogp/[title].png/+server.ts
import { generateOgpImage } from "$lib/generateOgpImage";
import type { RequestHandler } from "@sveltejs/kit";

export const GET: RequestHandler = async ({ params }) => {
  const { title } = params;
  const width = 1200;
  const height = 630;
  const png = await generateOgpImage(
    title ?? "Giants | BreakAI | AI for Business",
    width,
    height
  );

  return new Response(png, {
    headers: {
      "Content-Type": "image/png",
    },
  });
};
```

</details>

## 3. 各コンテンツの OGP として設定

SvelteKit で各コンテンツに OGP を設定するには、`<svelte:head>` を使用します。

<details>
<summary>OGP設定コード</summary>

```svelte
<svelte:head>
  <title>
    {data.post.title} | {$LL.POSTPAGE.SITE_TITLE({ title: data.post.title })}
  </title>
  <meta name="description" content="{data.post.content}" />

  <!-- Open Graph / Facebook -->
  <meta property="og:type" content="article" />
  <meta property="og:url"
  content={`https://new-giants.breakai.ai/posts/${data.post.id}`} />
  <meta property="og:title" content="{data.post.title}" />
  <meta property="og:description" content="{data.post.content}" />
  <meta property="og:image"
  content={`https://new-giants.breakai.ai/posts/ogp/large/${encodeURIComponent(data.post.title
  ?? 'breakai')}.png`} />
  <meta property="og:image:alt" content="{data.post.title}" />

  <!-- Open Graph / Facebook - Multiple image sizes -->
  <meta
    property="og:image"
    content="https://new-giants.breakai.ai/posts/ogp/large/${encodeURIComponent(
			data.post.title ?? 'breakai'
		)}.png"
  />
  <meta property="og:image:width" content="1423" />
  <meta property="og:image:height" content="771" />
  <meta
    property="og:image"
    content="https://new-giants.breakai.ai/posts/ogp/small/${encodeURIComponent(
			data.post.title ?? 'breakai'
		)}.png"
  />
  <meta property="og:image:width" content="1200" />
  <meta property="og:image:height" content="630" />

  <!-- Twitter -->
  <meta name="twitter:card" content="summary_large_image" />
  <meta name="twitter:site" content="{$LL.META.TWITTER_CREATOR()}" />
  <meta name="twitter:creator" content="{$LL.META.TWITTER_CREATOR()}" />
  <meta name="twitter:title" content="{data.post.title}" />
  <meta name="twitter:description" content="{data.post.content}" />
  <meta name="twitter:image"
  content={`https://new-giants.breakai.ai/posts/ogp/large/${encodeURIComponent(data.post.title
  ?? 'breakai')}.png`} />
  <meta name="twitter:image:width" content="1423" />
  <meta name="twitter:image:height" content="771" />
  <meta name="twitter:image"
  content={`https://new-giants.breakai.ai/posts/ogp/small/${encodeURIComponent(data.post.title
  ?? 'breakai')}.png`} />
  <meta name="twitter:image:width" content="1200" />
  <meta name="twitter:image:height" content="630" />

  <meta name="twitter:card" content="{$LL.META.TWITTER_CARD()}" />
  <meta name="twitter:site" content="{$LL.META.TWITTER_CREATOR()}" />
  <meta name="twitter:creator" content="{$LL.META.TWITTER_CREATOR()}" />
  <meta name="twitter:title" content="{data.post.title}" />
  <meta name="twitter:description" content="{data.post.content}" />
</svelte:head>
```

</details>

## パフォーマンス

Vercel 上にデプロイした場合、画像生成に 1-2 秒程度かかります。パフォーマンス向上のために、一度生成した画像をキャッシュすることをおすすめします。

```bash
󰕈 ekusiadadus  ~   01:05
 curl -w"time_total: %{time_total}\n" "https://new-giants.breakai.ai/posts/ogp/How%20To%20Perfectly%20Pitch%20Your%20Seed%20Stage%20Startup%20With%20Y%20Combinator's%20Michael%20Seibel.png"
Warning: Binary output can mess up your terminal. Use "--output -" to tell
Warning: curl to output it to your terminal anyway, or consider "--output
Warning: <FILE>" to save to a file.
time_total: 1.204007
```

## 参考

- [Satori + SvelteKit で OGP 画像を動的生成する](https://azukiazusa.dev/blog/satori-sveltekit-ogp-image/)
- [Satori で OGP 画像を動的生成する](https://zenn.dev/de_teiu_tkg/articles/677dabfb5c739c)
