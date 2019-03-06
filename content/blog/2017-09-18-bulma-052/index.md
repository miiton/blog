---
date: "2017-09-18T04:40:07.767Z"
title: "[要注意]Bulma 0.5.2で突如cssnextのvariablesが入った"
tags: ["bulma", "gatsbyjs", "postcss", "cssnext", "sass"]
description: ""
---

このサイトでは CSSフレームワークの [Bulma: a modern CSS framework based on Flexbox](http://bulma.io/)
を使用しているのですが、先日リリースされた
[0.5.2](https://github.com/jgthms/bulma/blob/master/CHANGELOG.md#052)
にアップデートしたところ、

```
 WARNING  Compiled with 14 warnings                         13:12:45

 warning  in ./src/css/styles.scss

postcss-custom-properties: /Users/tmnm/dev/src/gitlab.com/tmnm/blog/src/css/styles.scss:3152:3: Custom property ignored: not scoped to the top-level :root element (.columns.is-variable { ... --columnGap:... })
```

というWARNINGが大量に出るようになりました。gatsbyの仕様でブラウザ側にも赤い文字で
派手に表示されるのでとてもウルサイ。

## 問題のコード

`node_modules/bulma/sass/grid/columns.sass`

```sass
.columns.is-variable
  --columnGap: 0.75rem
  margin-left: calc(-1 * var(--columnGap))
  margin-right: calc(-1 * var(--columnGap))
  .column
    padding-left: var(--columnGap)
    padding-right: var(--columnGap)
  @for $i from 0 through 8
    &.is-#{$i}
      --columnGap: $i * 0.25rem
```
cssnextの変数(variables)である `--columnGap` がありますね。  
cssnextの現在の仕様は `:root` に variables を定義しないといけないので、 `.columns.is-variable`
で定義してしまっている点がWARNING対象です。(というかsassにcssnext紛れ込ませるってどういう了見だ。)

今までsass-loaderだけでbulmaを使っていた場合などで、
[BulmaのVariable gap](http://bulma.io/documentation/columns/gap/#variable-gap)
を使う場合にpostcssが必要になるなどといった影響が考えられます。

## 対処

今時点での作者のご意見は

> There's your problem. It's just a warning though. You can ignore it.
>
> https://github.com/jgthms/bulma/issues/1190#issuecomment-329714602

 とのことですので、あまりギャーギャー騒がずに以下のように直すパッチをひっそりと当てることで対応しています。

 ```sass
.columns.is-variable
   $columnGap: 0.75rem
   margin-left: calc(-1 * $columnGap)
   margin-right: calc(-1 * $columnGap)
   .column
     padding-left: $columnGap
     padding-right: $columnGap
   @for $i from 0 through 8
     &.is-#{$i}
       $columnGap: $i * 0.25rem
 ```

