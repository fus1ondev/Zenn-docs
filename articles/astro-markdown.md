---
title: "Astroで爆速Markdownブログ構築"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["astro","jamstack","markdown"]
published: true
---

## はじめに

先日、新鋭のWebフレームワーク「Astro」の存在を知り、自分のポートフォリオサイトをNext.jsからAstroに移行することにしました。

https://astro.build/

実際にサイトを作り始めてみたところ、Markdownの表示がとても楽だったので簡単に手順を載せておきます。

## Astroのプロジェクトを作成

パッケージマネージャーにはpnpmを使用していきます。

```shell
$ pnpm create astro@latest
```

色々聞かれますが、自分は以下のように選択しました。

![](https://storage.googleapis.com/zenn-user-upload/840db6a6cb7f-20220828.png)

## 試しに動かしてみる

プロジェクトのディレクトリに移動し、サーバーを起動してみます。

```shell
$ pnpm dev
```

`http://localhost:3000/`をブラウザで開くと、このようなページが表示されます。（「Just the basics」を選択した場合）

![](https://storage.googleapis.com/zenn-user-upload/03b9bd8786ea-20220828.png)

## Markdownページを追加

Astroが優れているのは、`pages`ディレクトリ下にそのままMarkdownファイル（MDXも可）を置くとページとして扱ってくれるという点です。

現在は`index.astro`しかないので`index.html`しか生成されませんが、ここにMarkdownファイルを置いていくことでブログなどを簡単に作れます。

```
pages
└── index.astro
```

:::details `.astro`ファイルについて

ちなみに、`.astro`ファイルとはAstro独自の構文で書かれたコンポーネントで、Vue.jsのようにHTML + JS + CSSが一体になっており、JSX/TSXライクな記法が使えるので違和感はあまり感じませんでした。

```jsx:index.astro（一部抜粋）
---
import Layout from '../layouts/Layout.astro';
import Card from '../components/Card.astro';
---

<Layout title="Welcome to Astro.">
  <main>
    <h1>Welcome to <span class="text-gradient">Astro</span></h1>
    <p class="instructions">
      Check out the <code>src/pages</code> directory to get started.<br />
      <strong>Code Challenge:</strong> Tweak the "Welcome to Astro" message above.
    </p>
    <ul role="list" class="link-card-grid">
      <Card
        href="https://docs.astro.build/"
        title="Documentation"
        body="Learn how Astro works and explore the official API docs."
      />
      ...
    </ul>
  </main>
</Layout>

<style>
  :root {
    --astro-gradient: linear-gradient(0deg, #4f39fa, #da62c4);
  }

  h1 {
    margin: 2rem 0;
  }

  main {
    margin: auto;
    padding: 1em;
    max-width: 60ch;
  }
  ...
</style>

```

:::

AstroでのMarkdownの扱いについてはドキュメントに書いてあります。（日本語版は情報が古いので英語版を確認することをオススメします）

https://docs.astro.build/en/guides/markdown-content/

早速、`posts`ディレクトリと`sample.md`を作成します。

`sample.md`の代わりに`sample/index.md`を作成しても大丈夫です。

```
pages
├── index.astro
└── posts
    └── sample.md
```

`sample.md`は以下のようにしておきます。

内容はダミーなので何でも構わないですが、フロントマターに`layout`を指定する必要があるのでご注意ください。

```markdown:sample.md
---
layout: ../../layouts/PostLayout.astro
title: "Astro v1 Launch!"
author: "Matthew Phillips"
date: "09 Aug 2022"
---

# Hello World!

This is a sample sentence.

```

次に、先ほど指定したレイアウトファイルを用意します。

元からある`layouts`ディレクトリ下に、`PostLayout.astro`ファイルを作成します。

```jsx:PostLayout.astro
---
const { frontmatter } = Astro.props;
---
<html>
  <head>
    <title>{frontmatter.title}</title>
  </head>
  <body>
    <h1>{frontmatter.title} by {frontmatter.author}</h1>
    <slot />
    <p>Written on: {frontmatter.date}</p>
  </body>
</html>
```

propsからフロントマターを参照できるようになっています。

また、`<slot />`にMarkdownからレンダリングされたHTMLがセットされます。内部的にはRemarkを使っているそうです。


ここまで行うと、以下のように記事ページが生成されます。

CSSを設定していないので見辛いですが、フロントマターに設定したタイトルや、記事の内容が問題なく表示されていることがわかります。

![](https://storage.googleapis.com/zenn-user-upload/c1222bbe8610-20220828.png)

## 記事一覧ページを作るには

以下のドキュメントに詳しく書いてある通り、`Astro.glob()`を使うと、指定のディレクトリ内にあるMarkdownファイルの情報を一気に取得できます。

この記事では解説しませんが、これを使うことで記事一覧を作成することも可能です。

https://docs.astro.build/en/reference/api-reference/#astroglob

```jsx
---
const posts = await Astro.glob('../pages/post/*.md'); // returns an array of posts that live at ./src/pages/post/*.md
---

<div>
{posts.slice(0, 3).map((post) => (
  <article>
    <h1>{post.frontmatter.title}</h1>
    <p>{post.frontmatter.description}</p>
    <a href={post.frontmatter.url}>Read more</a>
  </article>
))}
</div>
```

## 参考文献

https://zenn.dev/saldra/articles/47cf666d350029#%E5%AE%9F%E9%9A%9B%E3%81%AB%E4%BD%BF%E3%81%A3%E3%81%A6%E3%81%BF%E3%82%88%E3%81%86

