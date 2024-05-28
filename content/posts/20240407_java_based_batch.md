---
title: "2024年 Javaベースのバッチを作るなら その1 環境と初期設定"
date: 2024-04-07T00:00:00+09:00
draft: false
tags:
  - java
---

前回、[2020年 Javaベースのバッチを作るなら](../20200625_java_based_batch/) の続き。

これが今のところ、自分なりに、まぁまぁ良い手順かな、と言うのを書きます。

1. [Rancher Desktop](https://rancherdesktop.io/ "Rancher Desktop by SUSE") をインストール
2. [Visual Studio Code](https://azure.microsoft.com/ja-jp/products/visual-studio-code "Visual Studio Code – コード エディター | Microsoft Azure") と その拡張 `Remote Container` をインストール
3. リモートの Git で新規リポジトリを作成
4. チェックアウト
5. Visual Studio Codeでフォルダーを開く
6. `.devcontainer` と言うフォルダーを作る
7. 配下に3ファイル作る `Dockerfile` `devcontainer.json` `docker-compose.yml` 


```
.
|-- .devcontainer
|   |-- Dockerfile
|   |-- devcontainer.json
|   `-- docker-compose.yml
|-- .gitignore
|-- LICENSE
`-- README.md
```

_____

`.devcontainer/Dockerfile` [^1]

```
# Amazon Corretto with Java 21 / Debian 12
FROM maven:3.9-amazoncorretto-21-debian-bookworm
RUN apt update && apt -y install git && \
    unlink /etc/localtime && \
    ln -s /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
```

`.devcontainer/devcontainer.json` [^2]

```json
{
  "name": "batch",
  "dockerComposeFile": [
    "docker-compose.yml"
  ],
  "workspaceFolder": "/workspace",
  "service": "api",
  "forwardPorts": [
    8080,
    5432,
    61616
  ],
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-ceintl.vscode-language-pack-ja",
        "vscjava.vscode-java-pack",
        "redhat.vscode-xml",
        "redhat.vscode-yaml"
      ],
      "settings": {
        "editor.formatOnSave": true,
        "editor.formatOnPaste": true,
        "editor.insertSpaces": true,
        "editor.tabSize": 2,
        "editor.wordWrapColumn": 240,
        "editor.wordWrap": "wordWrapColumn",
        "files.encoding": "utf8",
        "files.eol": "\n",
        "files.trimTrailingWhitespace": true,
        "[xml]": {
          "editor.formatOnSave": false
        },
        "[java]": {
          "java.compile.nullAnalysis.mode": "disabled"
        }
      }
    }
  },
  "postCreateCommand": "echo 'nameserver 127.0.0.11\nnameserver 1.1.1.1' > /etc/resolv.conf"
}
```

`.devcontainer/docker-compose.yml`

```yaml
version: "3"

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: api
    command: sleep infinity
    ports:
      - 8080:8080
      - 9000:8000
    expose:
      - 8080
    volumes:
      - ../:/workspace:cached
    environment:
      - JPDA_ADDRESS=8000
      - JPDA_TRANSPORT=dt_socket
    networks:
      - localnw

networks:
  localnw:
    driver: bridge
```

____

この後、「コンテナーで再開」を選んで実行、で `Debian`/`Java`/`Maven` の開発環境が入りますよね。  
ここからは `mvn` コマンドで作業していくのが早いのではないでしょうか？

[^1]: お好みで。
[^2]: お好みで。とは言え `Java` `XML` `YAML` の拡張は欲しいですよね。
