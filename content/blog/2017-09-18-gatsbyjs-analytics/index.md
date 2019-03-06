---
date: "2017-09-18T01:30:00.104+0900"
title: "GatsbyでGoogle Analyticsを設定する"
tags: ["gatsbyjs", "analytics"]
description: ""
---

これは簡単でした。

<!-- more -->

もうREADME通りなのですが、
[gatsby\-plugin\-google\-analytics](https://github.com/gatsbyjs/gatsby/tree/master/packages/gatsby-plugin-google-analytics)
をインストールします。

```sh
yarn add gatsby-plugin-google-analytics
```

次に `gatsby-config.js` に設定を追記します。

```js{42-47}
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
    {
      resolve: 'gatsby-plugin-google-analytics',
      options: {
        trackingId: 'UA-11111111-1',
      },
    },
  ],
}
```

これだけで適用されます。簡単。ちなみに、ローカルでの稼働時(developモード)
にはAnalyticsのコードは挿入されず、ビルドすることでAnalyticsが有効なコード
が出力されます。ここがわからなくて最初「ん？」となりましたが、S3にデプロイ
したら有効になりました。


