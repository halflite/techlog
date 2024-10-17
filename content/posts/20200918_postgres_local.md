---
title: "2020年9月 ローカル環境に Postgres 12 を立てたいだけだった。"
date: 2020-09-18T00:00:00+09:00
draft: false
tags:
  - linux
  - docker
---

Heroku で、小さいアプリケーションを作りたいと思っていて。

MySQL も出来ない訳じゃないんですけど[^1] まぁ、順当に言って Postgres でしょう。 とは言え、今の時代は、Windowsマシンに Postgres を直接インストールするのはアンチパターンですよね…。 

という事で、 Vagrant / VirtualBox 上に CentOS7 のインスタンスを作り、それに Ansible で Docker / Docker-Compose をインストール。 その上で、 Docker-Compose を使って、 Postgres を立てる、と言うスクリプトを書いてみました。

- https://github.com/halflite/postgresql-win-local

こう言う環境を一つ作っておくと、 `docker-compose.yml` をいじるだけで、 MySQL だったり Redis だったりを簡単に(??)立てられるので、まぁ、ありがたいですねー…。 

[^1]: [ClearDB MySQL - Add-ons - Heroku Elements](https://elements.heroku.com/addons/cleardb "ClearDB MySQL - Add-ons - Heroku Elements")