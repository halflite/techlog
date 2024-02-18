---
title: "pom.xmlでEclipseの設定を定義する"
date: 2024-01-31T00:00:00+09:00
draft: false
tags:
  - java
  - eclipse
---

* eclipseの設定を `pom.xml `に記述します。
* Java17 / Maven 3.9.6 を想定しています。
* 文字コードはUTF-8、改行コードはLFになるように設定しています。
* 外部のライブラリのソースとJavadocも、ローカルにダウンロードするように設定しています。

```xml
  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <build>
    <plugins>
       <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-eclipse-plugin</artifactId>
        <version>2.10</version>
        <configuration>
          <downloadSources>true</downloadSources>
          <downloadJavadocs>true</downloadJavadocs>
          <additionalConfig>
            <file>
              <name>.settings/org.eclipse.core.resources.prefs</name>
              <content>
<![CDATA[eclipse.preferences.version=1${line.separator}encoding/<project>=${project.build.sourceEncoding}${line.separator}]]>
              </content>
            </file>
            <file>
              <name>.settings/org.eclipse.core.runtime.prefs</name>
              <content>
<![CDATA[eclipse.preferences.version=1${line.separator}line.separator=\n${line.separator}]]>
              </content>
            </file>
          </additionalConfig>
        </configuration>
      </plugin>
    </plugins>
  </build>
```

上記を `pom.xml` を書き足して、以下のようにコマンドでEclipse設定ファイルを作り直します。

```sh
mvn eclipse:clean eclipse:eclipse
```

______

大抵のEclipseの設定には[m2e](https://projects.eclipse.org/projects/technology.m2e "Eclipse Maven Integration - m2eclipse | projects.eclipse.org")が入っているでしょうし、m2eでEclipse設定ファイルを更新するのもありでしょう。

[![m2e setting](https://i.ibb.co/QP4wZ0s/image.png)](https://ibb.co/QP4wZ0s)

______

まだまだ現役なEclipseとMaven、ついつい設定忘れがちなので、書き残してみました。[^1]

[^1]: 2020/05/04 [MavenでEclipseプロジェクトを更新した際に、ソースコードの文字コードをUTF-8にする](/techlog/posts/20200504_eclipse_setting_utf8/) に、加筆修正して、再度公開しました。