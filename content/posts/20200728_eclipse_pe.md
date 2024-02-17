---
title: "2020年7月 EclipseにPropertiesEditorプラグインをインストールしたいだけだった。"
date: 2020-07-28T00:00:00+09:00
draft: false
tags:
  - java
  - eclipse
---

### TL;DR

> インストール用リポジトリーURLは、以下です
> * https://propedit.osdn.jp/eclipse/updates/

### 諸々など

現在、 Google Guice ベースで Bean Validation を入れようとしていて。 独自メッセージファイル ValidationMessages_ja.properties を作ろうかと思ったら、あれ、 Unicode でないとダメなのか…。 Spring-Boot だと、フツーに UTF-8 を読んでくれるんですけどね。

Validator を DI する時に、設定を注入出来ないかな？って思ったのですが、どこにも情報が無い!! まぁ、仕方ないですわ。 Unicodeのまま実装しますかね。 

なので、Eclipse に PropertiesEditor をインストールしようと思って、 ぐぐって [Eclipseで文字化けするプロパティファイルを編集する方法](https://proengineer.internous.co.jp/content/columnfeature/9158#2 "Eclipseで文字化けするプロパティファイルを編集する方法 | サービス | プロエンジニア") を参考にインストール…、しようと思っても入らない!!

まぁ、そういう事情もありましたよね [^1] [^2] で、ドメイン/ホストを色々打ち替えて、上記のリポジトリーURLが分かった次第。

本質的には、 JakartaEE / Hibernate が、メッセージファイルのフォーマットをUTF-8に公式対応してくれれば、それが良いのでしょうが…。

[^1]: [ニュース: OSDNにおけるアドウェア、不適切な広告についてのポリシーの現状について - OSDN運営・管理 - OSDN](https://ja.osdn.net/projects/sourceforge/news/24957 "ニュース: OSDNにおけるアドウェア、不適切な広告についてのポリシーの現状について - OSDN運営・管理 - OSDN")
[^2]: [OSSサイトの「SourceForge」が改称へ　「OSDN」に - ITmedia エンタープライズ](https://www.itmedia.co.jp/enterprise/articles/1504/08/news105.html "OSSサイトの「SourceForge」が改称へ　「OSDN」に - ITmedia エンタープライズ")