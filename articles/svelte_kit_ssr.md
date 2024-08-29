![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ba18eb85-4fb0-490b-ae27-7d8dd81cbfdc/f7253231-b578-4ad6-bd1e-76d0e6cd52a4/image.png)

## ブログサイトやナレッジサイトで、URL から自動で OGP と画像を生成して欲しい

最近、[ビジネスアイデアラボ](https://new-giants.breakai.ai) というアプリを作りました。

そこでは起業家秘話やユーザーが作成したアイデアが公開されています。ポストとして一つ一つに URL が振られていて、内容はそれぞれ異なります。

そのポストを Facebook, X とか 外部のサイトに公開したいときに、その内容に基づいた画像が OGP として入っていたら嬉しいです。

しかし、ポスト一つ一つに静的な画像を作成するのは手間がかかりますし、自動でやってほしいですよね

今回は、SvelteKit で OGP 画像を自動生成する方法を書きます。

## 画像生成ライブラリ

- satori
- sharp
- google font api

画像生成コード

```tsx
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

## 画像自動生成 API

```tsx
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

## 各コンテンツの OGP として設定する

```tsx
// src/routes//posts/[id]/+page.svelte
<svelte:head>
	<title>{data.post.title} | {$LL.POSTPAGE.SITE_TITLE({ title: data.post.title })}</title>
	<meta name="description" content={data.post.content} />

	<!-- Open Graph / Facebook -->
	<meta property="og:type" content="article" />
	<meta property="og:url" content={`https://new-giants.breakai.ai/posts/${data.post.id}`} />
	<meta property="og:title" content={data.post.title} />
	<meta property="og:description" content={data.post.content} />
	<meta
		property="og:image"
		content={`https://new-giants.breakai.ai/posts/ogp/large/${encodeURIComponent(data.post.title ?? 'breakai')}.png`}
	/>
	<meta property="og:image:alt" content={data.post.title} />

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
	<meta name="twitter:site" content={$LL.META.TWITTER_CREATOR()} />
	<meta name="twitter:creator" content={$LL.META.TWITTER_CREATOR()} />
	<meta name="twitter:title" content={data.post.title} />
	<meta name="twitter:description" content={data.post.content} />
	<meta
		name="twitter:image"
		content={`https://new-giants.breakai.ai/posts/ogp/large/${encodeURIComponent(data.post.title ?? 'breakai')}.png`}
	/>
	<meta name="twitter:image:width" content="1423" />
	<meta name="twitter:image:height" content="771" />
	<meta
		name="twitter:image"
		content={`https://new-giants.breakai.ai/posts/ogp/small/${encodeURIComponent(data.post.title ?? 'breakai')}.png`}
	/>
	<meta name="twitter:image:width" content="1200" />
	<meta name="twitter:image:height" content="630" />

	<meta name="twitter:card" content={$LL.META.TWITTER_CARD()} />
	<meta name="twitter:site" content={$LL.META.TWITTER_CREATOR()} />
	<meta name="twitter:creator" content={$LL.META.TWITTER_CREATOR()} />
	<meta name="twitter:title" content={data.post.title} />
	<meta name="twitter:description" content={data.post.content} />
</svelte:head>
```

## パフォーマンス

Vercel 上にデプロイしていて、大体 1-2 秒程度かかります

一度生成した画像は、キャッシュするとかしたほうが良さげですね

```bash
󰕈 ekusiadadus  ~   01:05 
 curl -w"time_total: %{time_total}\n" "https://new-giants.breakai.ai/posts/ogp/How%20To%20Perfectly%20Pitch%20Your%20Seed%20Stage%20Startup%20With%20Y%20Combinator's%20Michael%20Seibel.png"
Warning: Binary output can mess up your terminal. Use "--output -" to tell
Warning: curl to output it to your terminal anyway, or consider "--output
Warning: <FILE>" to save to a file.
time_total: 1.204007
```

### 参考

https://azukiazusa.dev/blog/satori-sveltekit-ogp-image/
https://zenn.dev/de_teiu_tkg/articles/677dabfb5c739c
