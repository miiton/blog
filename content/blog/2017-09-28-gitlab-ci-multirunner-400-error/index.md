---
date: "2017-09-28T11:47:55.867Z"
title: "gitlab-ci-multi-runnerが400エラーを出していた"
tags: ["gitlab", "gitlab-ci"]
description: ""
---

しばらく使っていなかったら、こんなエラーが出ていました。

```log
Sep 28 06:25:08 ** gitlab-runner[1302]: time="2017-09-28T06:25:08+09:00" level=warning msg="Checking for builds... failed" runner=d6f5e4dd status="400 Bad Request" #012<nil>
Sep 28 06:25:08 ** gitlab-ci-multi-runner[1302]: time="2017-09-28T06:25:08+09:00" level=warning msg="Checking for builds... failed" runner=d6f5e4dd status="400 Bad Request" 
```

## とりあえず最新版に

`apt-cache show gitlab-ci-multi-runner` を実行するとめちゃくちゃバージョンが上がっていました。最新版をインストールします。

**BEFORE**

```sh
$ dpkg -l gitlab*
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                                      Version                   Architecture              Description
+++-=========================================-=========================-=========================-========================================================================================
ii  gitlab-ci-multi-runner                    1.11.0                    amd64                     GitLab Runner
un  gitlab-ci-multi-runner-beta               <none>                    <none>                    (no description available)
un  gitlab-runner                             <none>                    <none>                    (no description available)
```

**AFTER**

```sh
$ apt-get -y install gitlab-ci-multi-runner

<省略>

$ dpkg -l gitlab*
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                                      Version                   Architecture              Description
+++-=========================================-=========================-=========================-========================================================================================
ii  gitlab-ci-multi-runner                    9.5.0                     amd64                     GitLab Runner
un  gitlab-ci-multi-runner-beta               <none>                    <none>                    (no description available)
un  gitlab-runner                             <none>                    <none>                    (no description available)
```

## 直った

バージョンあげただけでエラーが解消して動き出しました。
