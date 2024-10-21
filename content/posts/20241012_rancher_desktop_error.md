---
title: "RancherDesktopが \"Error: wsl.exe exited with code 429496729\" で起動できなくて困った話"
date: 2024-10-12T00:00:00+09:00
draft: false
tags:
  - docker
---

### TR;TD {#td_tr}

* WSLのディストリビューションを消すだけで解決するよ
* 先達には感謝ですね

### 何が起きたのか？ {#whats_happening}

ある日、PCを再起動したら、RancherDesktopがエラーで起動しなくて。

> Error: wsl.exe exited with code 4294967295

こう言うのは、PCを再起動すれば、概ね解決…、しない！

### 解決策 {#solution}

でも、先に踏んで解決して下さった方がいたんです！

* [【Docker】Windows10でRancher Desktopを起動するとエラーになる #kubernetes - Qiita](https://qiita.com/takanori-azegami-jp/items/0ba7d456f40eb963c994 "【Docker】Windows10でRancher Desktopを起動するとエラーになる #kubernetes - Qiita")
  * [Error Starting Rancher-desktop on Win10 with wsl error · Issue #2256 · rancher-sandbox/rancher-desktop](https://github.com/rancher-sandbox/rancher-desktop/issues/2256 "Error Starting Rancher-desktop on Win10 with wsl error · Issue #2256 · rancher-sandbox/rancher-desktop")

で、自分で試して分かったんですけど、結局、WSLのディストリビューションを消すだけで解決しましたね。

1. RancherDesktopを終了させる
2. WSLのディストリビューションを確認後、それを消し、WSLをシャットダウン

```ch
> wsl -l -v
* rancher-desktop         Stopped         2
* rancher-desktop-data    Stopped         2

> wsl --unregister rancher-desktop
> wsl --unregister rancher-desktop-data

> wsl --shutdown
```

3. その後、RancherDesktop再起動

これで解決しました。 
ただ、Dockerイメージをpullする時のレイヤーのキャッシュとか全部消えちゃうんで、再度コンテナを立ち上げる時は時間がかかりますね。

### 結局のところ {#at_last}

「少しのことにも、先達はあらまほしき事なり」[^1] とは昔からの伝えでしたよね。

[^1]: [徒然草第52段](https://www2.yamanashi-ken.ac.jp/~itoyo/tsuredure/turedure050_099/turedure052.htm "徒然草第52段")