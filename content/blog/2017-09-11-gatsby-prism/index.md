---
date: "2017-09-11T11:20:31.812Z"
title: "GatsbyにコードハイライトのPrismを導入する"
tags: ["gatsbyjs"]
description: ""
---

プラグイン `gatsby-remark-prismjs` を入れます。

<!-- more -->

## インストール

```sh
yarn add gatsby-remark-prismjs
```

## 設定

`gatsby-config.js` を編集。`gatsby-transformer-remark` のオプションに追記します。

```javascript{25-30}
module.exports = {
  siteMetadata: {
    author: 'tmnm',
    title: 'tmnm.tech',
  },
  plugins: [
    'gatsby-plugin-catch-links',
    {
      resolve: 'gatsby-source-filesystem',
      options: {
        path: `${__dirname}/src/pages`,
        name: 'pages',
      },
    },
    {
      resolve: 'gatsby-transformer-remark',
      options: {
        plugins: [
          {
            resolve: 'gatsby-remark-images',
            options: {
              linkImagesToOriginal: false
            }
          },
          {
            resolve: 'gatsby-remark-prismjs',
            options: {
              classPrefix: 'language-',
            },
          },
        ]
      }
    },
    'gatsby-plugin-react-helmet',
    'gatsby-plugin-sharp',
    {
      resolve: 'gatsby-plugin-postcss-sass',
      options: {
        precision: 8,
      },
    },
  ],
}
```

オプションの `classPrefix` はコードブロックの言語指定時のクラス名の頭につく文字列です。
例えば \`\`\`js だったら `class="language-js"` になります。

## css 追記

[公式にも書いていますが](https://www.gatsbyjs.org/packages/gatsby-remark-prismjs/)
Gatsbyでは行単位のハイライトにPrismの機能を使っていないので、スタイルの追記が必要に
なります。以下のような（色などは自分で変える）スタイルを追加します。

行番号の出し方がわかりませんでした。

```css
.gatsby-highlight-code-line {
  background-color: lighten(#141414, 10%);
  display: block;
  margin-right: -1em;
  margin-left: -1em;
  padding-right: 1em;
  padding-left: 0.75em;
  border-left: 0.25em solid $green;
}
/**
 * Add back the container background-color, border-radius, padding, margin
 * and overflow that we removed from <pre>.
 */
.gatsby-highlight {
  border-radius: 0.3em;
  margin: .5em 0;
  padding: 1em;
  overflow: auto;
}

/**
 * Remove the default PrismJS theme background-color, border-radius, margin,
 * padding and overflow.
 * 1. Make the element just wide enough to fit its content.
 * 2. Always fill the visible space in .gatsby-highlight.
 */
.gatsby-highlight pre[class*="language-"] {
  margin: 0;
  padding: 0;
  overflow: initial;
  float: left; /* 1 */
  min-width: 100%; /* 2 */
}
```



