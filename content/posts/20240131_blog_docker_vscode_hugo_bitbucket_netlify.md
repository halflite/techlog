---
title: "Docker/VSCode/Hugo/Bitbucket/Netlifyでブログを書く技術"
date: 2024-01-31T00:00:00+09:00
draft: false
tags:
  - etc
---
[音楽ブログ](https://bittersmooth.halflite.net/ "bitter*smooth") を数年書いているのですが、安いWindows11環境で、以下のような技術スタックで書いています。

### 要点

* 秘密鍵/公開鍵は難しいけど、頑張って！
* `netlify.toml` がハマりどころ

### 環境

* [Rancher Desktop](https://rancherdesktop.io/ "Rancher Desktop by SUSE") で Docker/仮想環境を作る
* [Visual Studio Code](https://azure.microsoft.com/ja-jp/products/visual-studio-code "Visual Studio Code – コード エディター | Microsoft Azure") (所謂)"VSCode" を使って Markdown で書く
* [Bitbucket](https://bitbucket.org/product/ "Bitbucket | Git solution for teams using Jira") にコミット/プッシュ
* [Netlify](https://www.netlify.com/ "Scale & Ship Faster with a Composable Web Architecture | Netlify") で公開

これ、一回慣れてしまうと簡単なんですけど、それまでが四苦八苦だったので、備忘録代わりに書いておきます。

### VSCode をインストール

1. [Git for Windows](https://gitforwindows.org/ "Git for Windows") も、事前にインストールしておきましょう
2. 拡張機能 [日本語拡張](https://marketplace.visualstudio.com/items?itemName=MS-CEINTL.vscode-language-pack-ja "Japanese Language Pack for Visual Studio Code - Visual Studio Marketplace") と [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers "Dev Containers - Visual Studio Marketplace") は必須でしょうね。

### Rancher Desktop/WSL2 をインストール

1. Rancher Desktop をインストールする際、最近はWSL2/Ubuntuが自動で入るみたいですね

### Bitbucketで、とりあえずリポジトリを作ってCloneする

1. プライベートリボジトリで作るのが良いでしょう
2. ローカルにクローンします。 これがプロジェクトになります。

### 公開/秘密の鍵ペアを作り、Bitbucketに公開鍵を登録する

1. WSL2の中身はUbuntuなので、ターミナルから、Ubuntuを選び、以下で鍵ペアが作れるのではないでしょうか
    * ユーザー名は`C:Users`配下の、自分のディレクトリを参照して下さい
2. bitbucketに公開鍵を登録するのは、公式手順に従うのが良いでしょう
    * [Windows で個人用 SSH キーをセットアップする | Bitbucket Cloud | アトラシアン サポート](https://support.atlassian.com/ja/bitbucket-cloud/docs/set-up-personal-ssh-keys-on-windows/ "Windows で個人用 SSH キーをセットアップする | Bitbucket Cloud | アトラシアン サポート")

```sh
sudo ssh-keygen
ls -la /root/.ssh/
# 公開鍵を表示
sudo cat /root/.ssh/id_rsa.pub
# 秘密鍵をローカルにコピー
sudo cp /root/.ssh/id_rsa /mnt/c/Users/(ユーザー名)
```

### Rancher Desktop/VSCode でローカル仮想環境を作る

1. プロジェクトのトップに `.devcontainer` と言うディレクトリを作ります
2. 以下に3つのファイル `id_rsa` `devcontainer.json` `Dockerfile` を置きます

#### id_rsa

上記で作った秘密鍵 `id_rsa` を `.devcontainer` 配下にコピーします

#### devcontainer.json

```json
{
  "name": "blog",
  "build": {
    "dockerfile": "Dockerfile"
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-ceintl.vscode-language-pack-ja"
      ],
      "settings": {}
    }
  }
}
```

本当に最小限の拡張して入っていないので `extensions` や `settings` は色々弄ったら楽しいと思います。

#### Dockerfile

```dockerfile
# 拡張機能入りdokcer image
FROM klakegg/hugo:0.111.3-ext-debian
COPY id_rsa /root/.ssh/
RUN apt update && apt -y install nodejs git && \
    chmod 600 /root/.ssh/id_rsa && \
    ssh-keyscan -H bitbucket.org >> /root/.ssh/known_hosts
```

この `Dockerfile` の肝心なところは、以下です。

* 拡張機能を入れたHugoのDockerイメージを使っています
    * ターミナル使うなら、debianが良いですよね
    * 最近のHugoは、拡張機能を入れられるようになったので、テンプレートによっては、Node.jsをインストールしないと、ビルドができなかったりします
* ホスト(手元のWinマシン)から、リモート(Docker/仮想環境)に 秘密鍵 `id_rsa` をコピーし、権限を限定させ、ホストに鍵を認識させる `ssh-keyscan` コマンドを打つ設定を入れています。
    * これがないと、リモートのターミナルから、Gitのプッシュも出来なかったりします

### netlify.tomlを書く

これが一番のハマりどころ

プロジェクトトップに `netlify.toml` と言うファイルを作っておくのです

```toml
[build]
publish = "public"

[context.production.environment]
HUGO_VERSION = "0.111.3"
HUGO_ENV = "production"
HUGO_ENABLEGITINFO = "true"
```

netlifyのデフォルトのHugoのヴァージョン、とにかく古いので、最近のテンプレートを使うと、ビルドが普通にコケます。 
なので。 `HUGO_VERSION = "0.111.3"` 等、指定してあげると、普通にビルドされます。

