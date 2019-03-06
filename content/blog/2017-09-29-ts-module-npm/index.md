---
date: "2017-09-29T01:28:55.475Z"
title: "超簡単！TypeScriptでyarn(npm) add(install)できるモジュールを作る方法"
tags: ["typescript", "npm", "yarn"]
description: ""
---

[alexjoverm/typescript\-library\-starter: Starter kit with zero\-config for building a library in TypeScript, featuring RollupJS, Jest, Prettier, TSLint, Semantic Release, and more\!](https://github.com/alexjoverm/typescript-library-starter)

こちらを使うだけなんですが、ものすごく簡単でした。

## 手順

READMEのUsage通りですが、yarnに置き換えて書きます

### クローンとインストール

```sh
git clone https://github.com/alexjoverm/typescript-library-starter.git hogehogejs
cd hogehogejs
yarn install
```

`yarn install` を実行すると、

```bash{8}
Hi! I'm setting things up for you!!

 Removed .git directory


Removed files: tools/init.ts,.all-contributorsrc,.gitattributes

Enter your library name (use kebab-case):
```

というメッセージが出てくるので、ライブラリ名を入力します。

```bash{8-26}
Hi! I'm setting things up for you!!

 Removed .git directory


Removed files: tools/init.ts,.all-contributorsrc,.gitattributes

Enter your library name (use kebab-case):  hogehogejs

src/hogehogejs.ts,test/hogehogejs.test.ts renamed
/Users/tmnm/dev/src/github.com/miiton/hogehogejs/package.json,/Users/tmnm/dev/src/github.com/miiton/hogehogejs/rollup.config.js,/Users/tmnm/dev/src/github.com/miiton/hogehogejs/LICENSE,/Users/tmnm/dev/src/github.com/miiton/hogehogejs/test/library.test.ts,/Users/tmnm/dev/src/github.com/miiton/hogehogejs/tools/gh-pages-publish.ts updated
Initialized empty Git repository in /Users/tmnm/dev/src/github.com/miiton/hogehogejs/.git/

Git initialized


Removed postinstall script


Happy coding!! ;)

husky
setting up Git hooks
done

✨  Done in 71.76s.
```

インストール完了！

### コードを書く

`src/hogehoge.ts` というファイルができているので、これを起点にコードを書いていきます。

### ビルドする

```sh
yarn build
```

これで、 `dist/` に以下のファイルができます

```
docs/
hogehogejs.es5.js
hogehogejs.es5.js.map
hogehogejs.umd.js
hogehogejs.umd.js.map
types/
```

### commitしてGitHubにpushする

の前に、`.gitignore` の `dist` の記述を消してから `add` して `commit` して `push` します。

## 以上！

これで

```
yarn add https://github.com/USERNAME/hogehogejs
```

が出来るモジュールができました。超簡単ですね。すごい！

この手順で作ったプライベートユースなライブラリ : [miiton/jptelformatter](https://github.com/miiton/jptelformatter)

