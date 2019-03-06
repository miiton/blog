---
date: "2017-09-10T11:33:06.343Z"
title: "ブログをGatsbyに移行しました - とりあえず動かすところまで"
tags: ["gatsbyjs", "blog"]
description: ""
---

ブログをGatsbyに移行しました。

## やったことまとめ

- starterを使いました
- とりあえず starter の見た目のままです

```sh
git clone https://github.com/DSchau/gatsby-blog-starter-kit.git blog
cd blog
```

`npm-check-updates`を確認すると結構バージョンが上がっているのでアップデートしちゃいます（要注意）
この記事を書いている時点で以下のようにバージョンが上がりました。

```sh
$ ncu

Using /Users/tmnm/dev/src/gitlab.com/tmnm/blog/package.json
⸨░░░░░░░░░░░░░░░░░░⸩ ⠏ :
 gatsby                      ~1.0.5  →  ~1.9.26
 gatsby-link                 ~1.0.1  →  ~1.6.16
 gatsby-plugin-sharp         ~1.0.1  →   ~1.6.7
 gatsby-remark-images        ~1.2.0  →  ~1.5.11
 gatsby-source-filesystem    ~1.0.1  →  ~1.4.12
 gatsby-transformer-remark   ~1.1.0  →   ~1.7.7
 gh-pages                   ~0.12.0  →   ~1.0.0
 prettier                    ~1.5.3  →   ~1.6.1

The following dependencies are satisfied by their declared version range, but the installed versions are behind. You can install the latest versions without modifying your package file by using npm update. If you want to update the dependencies in your package file anyway, run ncu -a.

 gatsby-plugin-catch-links   ~1.0.1  →  ~1.0.8
 gatsby-plugin-react-helmet  ~1.0.1  →  ~1.0.6

Run ncu with -u to upgrade package.json
```

```sh
ncu -au
rm -rf node_modules
yarn install
```

記事を追加するためのコマンドも入れます。これは Gatsby 作者のリポジトリに
あるものですが、 `src/pages/` 以下にテンプレートから `.md` ファイルを
生成するだけなので、他にもありそう（探していない）ですし、自分で作っても
いいと思います。

```sh
yarn add @dschau/create-gatsby-blog-post --dev
```


`package.json` にコマンドを追加します

```
{
  "name": "@dschau/gatsby-blog-starter-kit",
  "description": "Gatsby blog starter example",
  "version": "1.0.0",
  "author": "Dustin Schau <dustinschau@gmail.com>",
  "dependencies": {
    "gatsby": "~1.0.5",
    "gatsby-link": "~1.0.1",
    "gatsby-plugin-catch-links": "~1.0.1",
    "gatsby-plugin-react-helmet": "~1.0.1",
    "gatsby-plugin-sharp": "~1.0.1",
    "gatsby-remark-images": "~1.2.0",
    "gatsby-source-filesystem": "~1.0.1",
    "gatsby-transformer-remark": "~1.1.0",
    "react-icons": "~2.2.5"
  },
  "devDependencies": {
    "@dschau/create-gatsby-blog-post": "^1.1.0",
    "gh-pages": "~0.12.0",
    "prettier": "~1.5.3"
  },
  "keywords": [
    "gatsby"
  ],
  "license": "MIT",
  "main": "n/a",
  "scripts": {
    "build": "gatsby build",
    "deploy": "gatsby build --prefix-paths && gh-pages -d public",
    "develop": "gatsby develop",
    "preserve": "npm run build",
    "serve": "gatsby serve",
    "start": "npm run develop",
    "format": "prettier --trailing-comma es5 --single-quote --write \"src/**/*.js\"",
    "test": "echo \"Should probably write some tests!\"",
    "new": "create-post -- --date-format 'YYYY-MM-DD'"
  }
}
``` 

これで

```sh
yarn new -- newpost
```

というコマンドで `src/pages/YYYY-MM-DD-newpost/index.md` というファイルができるので、
適宜ディレクトリ名や `index.md` の中身を書き換えていきます。  
ここで `yarn start` で起動します。 http://localhost:8000/ にアクセスして、
新しく記事ができているのを確認します。

### `gatsby-config.js` を書き換える。

現時点ではここ書き換えたからといって...というものですが、一応。  
（参考 : [Accessing siteMetadata (gatsby-config.js) #1781](https://github.com/gatsbyjs/gatsby/issues/1781)）

**BEFORE** 

```javascript
  siteMetadata: {
    author: 'You!',
    title: `Gatsby Default (Blog) Starter`,
  },
```

この部分を書き換えます

**AFTER**

```javascript
  siteMetadata: {
    author: 'tmnm',
    title: 'tmnm.tech',
  },
```

### サイトタイトルの設定

各所 `react-helmet` の記載を書き換えます。

`src/templates/blog-post.js`

**BEFORE**

```jsx
<Helmet title={`Gatsby Blog - ${post.frontmatter.title}`} />
```

**AFTER**

```jsx
<Helmet title={`${post.frontmatter.title} - tmnm.tech`} />
```

`src/layout/index.js` 

**BEFORE**

```
<Helmet
  title="Gatsby Default (Blog) Starter"
  meta={[
    { name: 'description', content: 'Sample' },
    { name: 'keywords', content: 'sample, something' },
  ]}
/>
```

**AFTER**

```
<Helmet
  title="tmnm.tech"
  meta={[
    { name: 'description', content: '広く浅くやっているエンジニアの軽いブログです。Go言語、TypeScript、PowerShell、Hugo、Gatsby、Vim、Ansible、その他開発環境についての話題を書いています。' },
  ]}
/>
```

### パーマリンクの設定

URLを `/YYYY/MM/DD/slug` という形式にしたいです。

各記事の `frontmatter` の `path` を `/YYYY/MM/DD/slug` と直接書くしか今の所はなさそうです残念。

ただ新規記事を作成するときのテンプレートを指定できるようにするとか工夫すればいけそうですね。

### Hugoからの移行

Hugoからの移行では以下を気にする必要がありました

- frontmatter
  - frontmatterの書式がtomlになっている場合はyamlに変換する必要がある
  - frontmatterに今の状態で `"path", "date", "title", "tags"` がない場合は足す必要がある
  - `tags` が今の状態で 正規表現 `/^[_a-zA-Z][_a-zA-Z0-9]*$/` にマッチしないものは直す必要がある
  - `date` をクォーテーションで囲む必要がある NG: `2017-09-09T12:45:13+09:00`、 OK: `"2017-09-09T12:45:13+09:00"`
      - `egrep "^date: [^\"]" old/* -l | xargs sed -E -i '' 's/date: (.*)/date: "\1"/g'`
  - draft が今の状態でサポートされていない
- 本文
  - shortcodeは変換できないので書き換える必要がある

frontmatterの書式についてはHugo側で素敵コマンド `hugo convert toYAML` が用意されていました。

```sh
mkdir src/pages/old
cd /path/to/oldblog
hugo convert toYAML --output yaml
cp -a yaml/path/to/oldblog/content/post/* src/pages/old/
```

`--output` を指定しない場合は置換されるみたいです。

他の項目についてはチマチマ手動で直しました。

## デプロイまで

### ビルド

- MacBook Pro(15-inch, 2016) - 2.6 GHz Intel Core i7 / 16GB
- 24記事

で20秒でした

```sh
yarn build
```

### デプロイ

以下のようなスクリプトを `packages.json` に書き込みました(AWS S3です)

```json
"deploy": "gatsby build --prefix-paths && aws s3 --profile PROFILE_NAME sync --delete ./public/ s3://S3_BUCKET_NAME/ --cache-control \"max-age=604800\"",
```

```sh
yarn deploy
```

以上でした。 `react-router` みたいなリダイレクトの設定などは不要で動きました。素敵。


## 動きがおかしいところとか

### 記事ファイルを変更したけど反映されない

記事ファイルを移動したときは、移動したにも関わらず記事が2つ表示されてしまうという
謎の状態に。 `yarn start` をし直したり、別の記事を更新したりすると直りました。  

### ブラウザで表示中の記事とは別の記事を更新した場合、更新した記事が変更されていない。

その記事を開いた状態で改めて更新すると反映されます。うーん？

### `yarn start`状態で新規記事を追加した直後、記事一覧には現れるが、クリックしても404

`yarn start` し直すとOKです。それでもダメなら、 `.cache` の中を全部消してから `yarn start`

### `yarn start`状態で新規記事を追加した直後、記事一覧には現れるが、クリックしても無反応

ページリロードしたらOKでした。 `hot-reload` の問題かな。

## Q&A

### ビルド速度はどんなもん？

Hugo(Golang)と比べるとそりゃお察しくださいだと思います。  
それよりも表示がサクサクなのでユーザー視点（見る側）の事を考えればGatsbyの思想の方が良いかなと思います。

### SEO関連はどう？

後日 Search Consoleなどチェックします。

### ページ内リンク(見出し、目次)どうやるの？

議論はされているっぽい（2016年...） 何かしらやりようはあるはず。
[Anchor Links, part 2 · Issue \#386 · gatsbyjs/gatsby](https://github.com/gatsbyjs/gatsby/issues/386)

## 所感

構築にはReact（+ redux）、GraphQLの知識に加えてSSRの知識などが必要になりますので、Wordpressなどの
一般的なブログからの移行先としてはハードルが高い（エンジニアにお願いしましょう）ですが、
ビルド時にWordpressの記事データを抜いてきて静的サイトに変換するプラグインなど尖った機能が盛りだくさん
ですし、何より表示速度がサクサクなので「ユーザーフレンドリー」を優先する場合の良い選択肢が増えたなーと
思います。

とりあえず公式ブログのチュートリアルだけをみてやってやった状態なので、これから
[ドキュメント](https://www.gatsbyjs.org/docs/) を見たりしていじっていきたいと思っています。

