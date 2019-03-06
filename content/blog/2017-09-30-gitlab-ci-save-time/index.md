---
date: "2017-09-30T03:04:51.336Z"
title: "Gitlab CI の時間を短縮する方法"
tags: ["gitlab", "gitlab-ci", "ci", "docker"]
description: ""
---

dockerを使うパターンで、思いついたら追記していきます。

## docker pullをせずにローカルのイメージを使う

`gitlab-runner` の `config.toml` の `runners.docker` セクションに
`pull_policy = "if-not-present"` または `pull_policy = "never"` を記述します。
こうすると、ローカルのイメージを使うようになるので、`docker pull` にかかる時間を
短縮することができます。

```toml{15}
concurrent = 3
check_interval = 10

[[runners]]
  name = "tmnm-ci-master"
  url = "https://gitlab.com"
  token = "***"
  executor = "docker+machine"
  limit = 3
  [runners.docker]
    tls_verify = false
    image = "ubuntu:16.04"
    privileged = false
    disable_cache = true
    pull_policy = "if-not-present"
```
