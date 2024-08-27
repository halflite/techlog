---
title: "2023年 Javaベースのプロジェクトを作る"
date: 2023-10-10T00:00:00+09:00
draft: false
tags:
  - java
---

単に、自分の備忘録です。

* [初期プロジェクト作成](#create_init_project)
* [pom.xmlを何かする](#edit_pom_xml)

_____

### 初期プロジェクト作成 {#create_init_project}

```sh
mvn archetype:generate \
  -DarchetypeArtifactId=maven-archetype-quickstart \
  -DinteractiveMode=false \
  -DgroupId=web_app_example \
  -DartifactId=app \
  -Dpackage=app
```

単にバッチや単独ウェブアプリなら、パッケージは浅いほうが良いでしょうね。

### pom.xmlを何かする {#edit_pom_xml}

pom.xml に諸々を追加することになります。  
初期はこんなものが自動生成されます

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>web_app_example</groupId>
  <artifactId>app</artifactId>
  <packaging>jar</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>csvbatch</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

ここからライブラリを足していくと、手組みで作っているなぁ！と言う感じになります。