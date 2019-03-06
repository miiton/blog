---
date: "2017-09-14T02:13:49.553Z"
title: "Androidエミュレーターが起動しない時の対処のうちのいくつか"
tags: ["vagrant", "android", "emulator", "tips", "docker", "virtualbox"]
description: ""
---

## Vagrantが悪さしているパターン

```sh
emulator -avd Nexus_5_Android_7
```

というコマンドでエミュレーターを起動しようとしたら、

```
emulator: ERROR: Unfortunately, there's an incompatibility between HAXM hypervisor and VirtualBox 4.3.30+ which doesn't allow multiple hypervisors to co-exist.  It is being actively worked on; you can find out more about the issue at http://b.android.com/197915 (Android) and https://www.virtualbox.org/ticket/14294 (VirtualBox)
Internal error: initial hax sync failed
```

というエラーが出たので、Dockerを入れていたので、
[こちらの記事](http://qiita.com/rhoboro/items/1c6a1550f0d24e470460)
に従って、haxm のバージョンをあげてもダメで、Dockerを終了してもダメ。

VirtualBoxを起動してもVMは全部停止している。

色々調べたりして、以下のコマンドを実行して見たところ、
**VirtualBox上では起動していないけど、Vagrant的には起動している**
マシンがいました。

```sh
$ vagrant global-status

id       name    provider   state    directory
-------------------------------------------------------------------------
29f8fe1  default virtualbox poweroff /Users/tmnm/dev/vagrant/EdgeWin10
7921d85  default virtualbox running  /Users/tmnm/dev/vagrant/win10
f5ca0de  default virtualbox running  /Users/tmnm/dev/vagrant/win8
d1fe4a6  default virtualbox running  /Users/tmnm/dev/vagrant/ie9
1244785  default virtualbox running  /Users/tmnm/dev/vagrant/win7
```

それぞれ、

```sh
vagrant halt 7921d85
vagrant halt f5ca0de
vagrant halt d1fe4a6
vagrant halt 1244785
```

としてあげることで、エミュレーター起動しました。

ちなみにこの状態でDocker for Macも起動できました。

### 参考

- [Android Studio Unable to run AVD \- Stack Overflow](https://stackoverflow.com/questions/37397810/android-studio-unable-to-run-avd)

## やっぱりDockerが悪さしているパターン

Android Studio Canary で発生したんですが、 Android 7.1.1 / 8.0 は起動できるのに
6.0 / 5.1 が起動できないというパターンです。

以下のようなログが出ます。

```
Failed to sync HAX vcpu contextInternal error: Initial hax sync failed
```

この場合はDockerを終了させると起動しました。

### 参考

- [Fix for the Android Emulator crashing during launch \| Bram\.us](https://www.bram.us/2017/05/12/fix-for-the-android-emulator-crashing-during-launch/)

## 所感

Androidはハマりどころが多い...

