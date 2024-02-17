---
title: "MavenでEclipseプロジェクトを更新した際に、ソースコードの文字コードをUTF-8にする"
date: 2020-05-04T00:00:00+09:00
draft: false
tags:
  - java
  - eclipse
---
WindowsマシンでEclipse使っていると、いつの間にか、ソースの文字コードがMS932になっていて、イラッとしません？

Eclipseの設定から直しても、プロジェクトのクラスパス修正しようと、mvnコマンドで.settingsファイルを初期化したら、文字コード設定が、元に戻ってしまったりして…。

解決策としては、プロジェクトのpom.xmlを、以下のように記述/変更します。

```xml
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <build>
    <plugins>
      <!-- Eclipse -->
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
          </additionalConfig>
        </configuration>
      </plugin>
    </plugins>
  </build>
```

で、Eclipse用の.settingsファイルを作り直すと、次からは、ソースコードの文字コードは、UTF-8になっています。

```sh
mvn eclipse:clean eclipse:eclipse
```

* 元ネタ [Define Eclipse project encoding as UTF-8 from Maven - Stack Overflow](https://stackoverflow.com/a/7605915 "Define Eclipse project encoding as UTF-8 from Maven - Stack Overflow") 

___

旧blog 2018/10/12 の記事に加筆訂正して再掲しました。