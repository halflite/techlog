---
title: "AWS Lambda と Log4j2 で困った話"
date: 2024-10-14T00:00:00+09:00
draft: false
tags:
  - java
  - aws
---

### TD;TR {#tdtr}

* `maven-shade-plugin` がハマりどころ
* 結構新しい問題だった
* AIも使いよう

### 何が起きたのか？ {#whats_happening}

久し振りに、 `AWS Lambda` を触ったんです。 `AWS EventBridge` と組み合わせて、簡易なバッチとして動かそうと。

で、実際に Lambda Function を FatJARにしてデプロイしたら、あれ？ `log4j2.xml` のパースに失敗して、起動できない??? ローカルでは動くのに…。

まず、 `pom.xml` に log4j2 の BOM を設定

```xml
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-bom</artifactId>
        <version>2.23.1</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
```

依存関係に `log4j-slf4j2-impl` と `aws-lambda-java-log4j2` を足す。

```xml
    <!-- Logging-->
    <dependency>
      <groupId>com.amazonaws</groupId>
      <artifactId>aws-lambda-java-log4j2</artifactId>
      <version>1.6.0</version>
    </dependency>
    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-slf4j2-impl</artifactId>
    </dependency>
```

`log4j2.xml` は、公式にある、一番シンプルなもの[^1] を置く。

そうすれば `slf4j` → `aws-lambda-java-log4j2` → `log4j2` で行けるよね?? って思ってたら、上記の如く、起動しないのです。

### 結局のところ {#at_last}

これは、自分で検索したりしてもダメっぽいな…、と思ったので、Windows11付属のAIアシスタント、Copilotに質問したのですが、要領を得ない解答で少々困ってしまって。 ハレーションを起こさなかっただけマシかもですが。

ただ、Log4j2 GitHubにあるイシュー[^2] へのリンクを付けてくれたのです。 あれ、これ今年の8月21日に起票されたものなんですね。

なんだ、ほぼ正解書いてあるじゃないですか。 [このレス](https://github.com/apache/logging-log4j2/discussions/2874#discussioncomment-10415428) にある **Click of a complete example of Maven Shade Plugin configuration** 以下で記述している `maven-shade-plugin` 設定そのままで、フツーに動きました。[^3]

AIも、所詮はツールと割り切って冷静に接すれば、それなりに使い道もあるよね、と言うお話でした。

[^1]: [log4j2.xml](https://github.com/awsdocs/aws-lambda-developer-guide/blob/main/sample-apps/s3-java/src/main/resources/log4j2.xml)
[^2]: ["ERROR StatusLogger Unrecognized format specifier [d]" after i update the log4j version from 2.20.0 to 2.21.0 · apache/logging-log4j2 · Discussion #2874](https://github.com/apache/logging-log4j2/discussions/2874)
[^3]: Lambda Function なので `<mainClass>org.example.Main</mainClass>` の記述だけはコメントアウトで。