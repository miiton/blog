---
date: "2017-09-20T13:18:46.539Z"
title: "PowerShell v6がβになっていたので触ってみた"
tags: ["powershell", "mac"]
description: ""
---

久々にMacでも使えるオープンソースのPowerShellのリポジトリをのぞいてみたら
betaになっていたので触ってみました。

## beta.7 になっている。

```sh-
$ brew cask info powershell

powershell: 6.0.0-beta.7
https://github.com/PowerShell/PowerShell
Not installed
From: https://github.com/caskroom/homebrew-cask/blob/master/Casks/powershell.rb
==> Name
PowerShell
==> Artifacts
powershell-6.0.0-beta.7-osx.10.12-x64.pkg (Pkg)
==> Caveats
A OpenSSL-backed libcurl is required for custom handling of certificates.
This is rarely needed, but you can install it with
  brew install curl --with-openssl
See https://github.com/PowerShell/PowerShell/issues/2211
```

## インストール！簡単！

```sh
brew cask install powershell
```

## 触ってみる

とりあえずいつも使っているfishから起動。

```powershell
$ powershell
PowerShell v6.0.0-beta.7
Copyright (C) Microsoft Corporation. All rights reserved.

PS /Users/tmnm/dev/src/gitlab.com/tmnm/blog>
```

whoooeh windowsっぽいシェルになりました（macです）。PowerShell on Tmux on Alacritty :p


```powershell
PS /Users/tmnm/dev/src/gitlab.com/tmnm/blog> get-host


Name             : ConsoleHost
Version          : 6.0.0
InstanceId       : 835b627c-bf66-4d35-b3fc-5b876f78d6ef
UI               : System.Management.Automation.Internal.Host.InternalHostUserInterface
CurrentCulture   : ja-JP
CurrentUICulture : ja-JP
PrivateData      : Microsoft.PowerShell.ConsoleHost+ConsoleColorProxy
DebuggerEnabled  : True
IsRunspacePushed : False
Runspace         : System.Management.Automation.Runspaces.LocalRunspace



PS /Users/tmnm/dev/src/gitlab.com/tmnm/blog> Get-Host


Name             : ConsoleHost
Version          : 6.0.0
InstanceId       : 835b627c-bf66-4d35-b3fc-5b876f78d6ef
UI               : System.Management.Automation.Internal.Host.InternalHostUserInterface
CurrentCulture   : ja-JP
CurrentUICulture : ja-JP
PrivateData      : Microsoft.PowerShell.ConsoleHost+ConsoleColorProxy
DebuggerEnabled  : True
IsRunspacePushed : False
Runspace         : System.Management.Automation.Runspaces.LocalRunspace
```

Windowsと一緒で大文字小文字関係ないんですよね。

現在のディレクトリ以下のファイル数を拡張子でグルーピングしてトップ10を表示

```powershell
PS /Users/tmnm/dev/src/gitlab.com/tmnm/blog> Get-ChildItem -Recurse | Group Extension | Sort-Object Count -Desc | Select -First 10

Count Name                      Group
----- ----                      -----
23869 .js                       {gatsby-config.js, gatsby-node.js, postcss.config.js, asn1.js...}
 6474                           {node_modules, public, src, LICENSE...}
 2953 .json                     {package.json, package.json, package.json, package.json...}
 2390 .md                       {README.md, README.md, readme.md, readme.md...}
  888 .test                     {comment_feature.test, eol-comment_feature.test, keyword_feature.test, number_feature.test...}
  723 .h                        {nan_callbacks_12_inl.h, nan_callbacks_pre_12_inl.h, nan_callbacks.h, nan_converters_43_inl.h...}
  351 .flow                     {index.js.flow, index.js.flow, index.js.flow, index.js.flow...}
  302 .map                      {lodash.map, ajv.min.js.map, index.js.map, bulma.css.map...}
  196 .dat                      {byte0000.dat, byte0001.dat, byte0002.dat, byte0003.dat...}
  144 .html                     {index.html, index.html, index.html, index.js.html...}
```

こういう処理をさせるならPowerShell最強でしょう。あとパフォーマンスも結構きびきび動きますね。いいですね。

## コマンドの数は？

alphaバージョンの時は、

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">PowerShell for MacのCmdletの数 345 <a href="https://t.co/emLHqjqcoA">pic.twitter.com/emLHqjqcoA</a></p>&mdash; miiton (@MIITON) <a href="https://twitter.com/MIITON/status/766600451680718849">2016年8月19日</a></blockquote>

で、今見ると、

```powershell
PS /Users/tmnm/dev/src/gitlab.com/tmnm/blog> Get-Command | Measure-Object


Count    : 352
Average  :
Sum      :
Maximum  :
Minimum  :
Property :

```

Cmdletは前に見た時から7つ増えて352個に。これFunctionも含んでいるので正確じゃないですね。

```powershell
PS /Users/tmnm/dev/src/gitlab.com/tmnm/blog> Get-Command | Group CommandType

Count Name                      Group
----- ----                      -----
  129 Function                  {Add-NodeKeys, AddDscResourceProperty, AddDscResourcePropertyFromMetadata, AfterAll...}
  223 Cmdlet                    {Add-Content, Add-History, Add-Member, Add-Type...}
```

こういう結果でした。

ちなみに私の最近の環境は Alacritty + tmux + fish ですが、もともとはWindowsサーバーの
お守りをしていた時にPowerShellをバリバリ使っていた時に

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">LinuxにPowerShellはよ</p>&mdash; miiton (@MIITON) <a href="https://twitter.com/MIITON/status/734908931235975168">2016年5月24日</a></blockquote>

とのたまっていて、Windowsサーバーのお守りから退いた時にこのPowerShellの発表があって
やたらテンションが上がっていたのを覚えています。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">すげーｗｗｗ MacでPowerShellうごいたーーーｗｗｗｗ</p>&mdash; miiton (@MIITON) <a href="https://twitter.com/MIITON/status/766555197690617856">2016年8月19日</a></blockquote>

## デフォルトシェルにする？

さすがにしない...少数派すぎるので色々弊害が出そうです。  
bash/zsh前提で書かれているナレッジばかりですもんね。  
ただ、ちょっとした集計処理とかには使うかもしれません。


## ちなみに現在のWindows 10のPowerShellは5.1でした

```powershell
> Get-Host

Name             : ConsoleHost
Version          : 5.1.15063.608
InstanceId       : 5eb33054-b47a-4cce-971e-ac35201569dc
UI               : System.Management.Automation.Internal.Host.InternalHostUserInterface
CurrentCulture   : ja-JP
CurrentUICulture : ja-JP
PrivateData      : Microsoft.PowerShell.ConsoleHost+ConsoleColorProxy
DebuggerEnabled  : True
IsRunspacePushed : False
Runspace         : System.Management.Automation.Runspaces.LocalRunspace
```
