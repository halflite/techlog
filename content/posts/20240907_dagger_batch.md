---
title: "2024年9月 Daggerでバッチを作りたいだけだった"
date: 2024-09-07T00:00:00+09:00
draft: false
tags:
  - java
---

* https://github.com/halflite/batch-daggar-example

作りました。 単に標準出力にログを出すだけですが…。

実は「Daggerでバッチプログラム」って全然サンプルないんですよ。 Daggerって、Android用のフレームワークだと思われているのかもしれませんが…。

* [Maven設定](#comile_setting)

_____

### Maven設定 {#comile_setting}

* [pom.xml](https://github.com/halflite/batch-daggar-example/blob/main/batch/pom.xml)

上記参照。

* APTで生成クラスを作るには、 `maven-compiler-plugin` に設定を加える
* 自動正クラスをクラスパスに通すには `build-helper-maven-plugin` で解決する

コレかな。

```xml
  <build>
    <plugins>
      <!-- Dagger Compile -->
      <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.8.1</version>
        <configuration>
          <annotationProcessorPaths>
            <path>
              <groupId>com.google.dagger</groupId>
              <artifactId>dagger-compiler</artifactId>
              <version>${dagger.version}</version>
            </path>
          </annotationProcessorPaths>
        </configuration>
      </plugin>
      <!-- APT -->
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>build-helper-maven-plugin</artifactId>
        <version>3.0.0</version>
        <executions>
          <execution>
            <id>add-source</id>
            <phase>generate-sources</phase>
            <goals>
              <goal>add-source</goal>
            </goals>
            <configuration>
              <sources>
                <source>${project.build.directory}/generated-sources/annotations</source>
              </sources>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
```