---
title: "2024年 Javaベースのバッチを作るなら その2 プロジェクトを作る"
date: 2024-04-14T00:00:00+09:00
draft: false
tags:
  - java
---

前回、[2024年 Javaベースのバッチを作るなら その1 環境と初期設定](../20240407_java_based_batch/) の続き。  

開発環境は、 VSCode と Rancher Desktop だけでやってみます。

_____

### プロジェクトを作る

まずバージョン確認

```sh
> mvn --version
Apache Maven 3.9.6 (bc0240f3c744dd6b6ec2920b3cd08dcc295161ae)
Maven home: /usr/share/maven
Java version: 21.0.3, vendor: Amazon.com Inc., runtime: /usr/lib/jvm/java-21-amazon-corretto
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "5.15.146.1-microsoft-standard-wsl2", arch: "amd64", family: "unix"
```

初期プロジェクト作成 [^1]

```sh
mvn archetype:generate \
  -DarchetypeArtifactId=maven-archetype-quickstart \
  -DinteractiveMode=false \
  -DartifactId=csvbatch \
  -DgroupId=app \
  -Dpackage=csvbatch.app
```

このようなプロジェクトが作られます。

```sh
.
|-- LICENSE
|-- README.md
`-- csvbatch
    |-- pom.xml
    `-- src
        |-- main
        |   `-- java
        |       `-- csvbatch
        |           `-- app
        |               `-- App.java
        `-- test
            `-- java
                `-- csvbatch
                    `-- app
                        `-- AppTest.java
```

### 以降

pom.xml に諸々を追加することになります。

基本スタックは以前書いた [AWS Lambda FunctionでJavaを採用したい](../20220925_aws_lambda_guice/) に準拠したいと思います。

[^1]: 単発バッチだと、ネストを深くしないために、アーティファクトID/グループIDは短くしたほうが良いでしょう