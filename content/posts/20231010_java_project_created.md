---
title: "2023年 Javaベースのプロジェクトを作る"
date: 2023-10-10T00:00:00+09:00
draft: false
tags:
  - java
---

単に、自分の備忘録です。

ローカル開発環境は Windows11 / WSL2 / VSCode / RancherDesktop です。

* [リポジトリ作成](#create_repository)
* [初期プロジェクト作成](#create_init_project)
* [pom.xmlを何かする](#edit_pom_xml)
* [コンパルのJavaバージョン](#java_ver_setting)
* [FatJar/UberJar 作成設定](#fatjar_setting)

_____

### リポジトリ作成 {#create_repository}

1. GitHub や BitBUcket にリポジトリを作る
2. VSCode でチェックアウト
3. devcontainerの設定を作る。[^1]

### 初期プロジェクト作成 {#create_init_project}

```sh
mvn archetype:generate \
  -DarchetypeArtifactId=maven-archetype-quickstart \
  -DinteractiveMode=false \
  -DgroupId=web_app_example \
  -DartifactId=app \
  -Dpackage=app
cd app
mvn clean compile
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
  <name>app</name>
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

ここからライブラリを足していくと、手組みで作っているなぁ！と言う感じになります。[^2]

### コンパルのJavaバージョン {#java_ver_setting}

```xml
  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>
```

この3行だけで、「Java17でコンパイル、ソースコードはUTF-8」が決まります。[^3]

### FatJar/UberJar 作成設定 {#fatjar_setting}

```xml
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-assembly-plugin</artifactId>
        <version>3.7.1</version>
        <configuration>
          <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
          </descriptorRefs>
          <archive>
            <manifest>
              <mainClass>app.App</mainClass>
            </manifest>
          </archive>
          <finalName>${project.name}-${project.version}</finalName>
          <appendAssemblyId>false</appendAssemblyId>
        </configuration>
        <executions>
          <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals>
              <goal>single</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
```

`pom.xml` に上記を足すと、FatJar/UberJar(単独で立ち上がるJar)が作られます

```sh
cd app
mvn clean package
java -jar target/app-1.0.0.jar
```

[^1]: 2024/04/07 追記 [2024年 Javaベースのバッチを作るなら その1 環境と初期設定](/techlog/posts/20240407_java_based_batch/)
[^2]: でも、流石に2023年に、 デフォルトが JUnit3 なのはちょっとね…。
[^3]: [Mavenの真実とウソ | PPT](https://www.slideshare.net/slideshow/maven-196821326/196821326)