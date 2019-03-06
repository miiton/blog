---
date: "2017-09-10T19:26:20.067+0900"
title: "Gatsbyでアドセンスを追加"
tags: ["gatsby", "adsense"]
description: ""
---

react-adsense を使います。

<!-- more -->

`src/html.js`

ここで、 `adsbygoogle.js` を読み込む記述をしておきます。

```js{3}
  {this.props.headComponents}
  {css}
  <script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
</head>
```

広告を掲載したい箇所に `react-adsense` の記述を追記します。  
この例では

`src/template/blog-post.js` に 追記しました。

```js{4,14-18,24-33}
import Helmet from 'react-helmet';
import BackIcon from 'react-icons/lib/fa/chevron-left';
import ForwardIcon from 'react-icons/lib/fa/chevron-right';
import AdSense from 'react-adsense'

import Link from '../components/Link';
import Tags from '../components/Tags';

...

        <h2 className="date">
          {post.frontmatter.date}
        </h2>
        <AdSense.Google
          client="ca-pub-9616054341361384"
          slot="5153910610"
          format="auto"
        />
        <div
          className="blog-post-content"
          dangerouslySetInnerHTML={{ __html: post.html }}
        />
        <Tags list={post.frontmatter.tags || []} />
        <AdSense.Google
          client="ca-pub-9616054341361384"
          slot="4735108213"
          format="rectangle"
        />
        <AdSense.Google
          client="ca-pub-9616054341361384"
          slot="5431282218"
          format="rectangle"
        />
        <div className="navigation">
          {prev &&
            <Link className="link prev" to={prev.frontmatter.path}>

``` 
