---
date: "2017-09-11T10:46:14.053Z"
title: "GatsbyでBulmaを使う"
tags: ["gatsbyjs"]
description: ""
---

GatsbyでBulmaを使います。他のsassベースのフレームワークも同じ手順でいけると思います。

<!-- more -->

## gatsby-plugin-postcss-sass を導入する

```sh
yarn add gatsby-plugin-sass
```

gatsby-config.js

```js{30-35}
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
          }
        ]
      }
    },
    'gatsby-plugin-react-helmet',
    'gatsby-plugin-sharp',
    {
      resolve: `gatsby-plugin-postcss-sass`,
      options: {
        precision: 8,
      },
    },
  ],
}
```

## Bulmaをインストールする

現時点で v0.5.1 ですね


```sh
yarn add bulma
```

## scssファイルを作成

色々やり方はあると思いますが、 `src/css/base.scss` というファイルに以下の記述をしました
（拡張子がcssだとsassとして認識されないので注意）

```css
@import '../../node_modules/bulma/sass/utilities/initial-variables.sass';
@import './variables.scss';
@import '../../node_modules/bulma/bulma.sass';
```

`variables.scss` は各種sass/scssの変数定義です。この書き方は
[Bulmaの公式ドキュメント](http://bulma.io/documentation/overview/customize/)
を参考にしています。

## レイアウトファイルで呼び出し

`src/layouts/index.js` 

```js
import '../css/base.scss'
```

これで反映されます。
