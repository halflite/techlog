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

* [依存ライブラリ](#dependncy_library)
* [Maven設定](#comile_setting)
* [DaggerとJakarta EE](#jakarta_ee)
* [結局のところ](#why_batch)

_____

### 依存ライブラリ {#dependncy_library}

```ch
mvn dependency:tree
[INFO] Scanning for projects...
[INFO] 
[INFO] ---------------------< batch.daggar.example:batch >---------------------
[INFO] Building batch 1.0.0
[INFO]   from pom.xml
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- dependency:3.6.1:tree (default-cli) @ batch ---
[INFO] batch.daggar.example:batch:jar:1.0.0
[INFO] +- com.google.dagger:dagger:jar:2.52:compile
[INFO] |  +- jakarta.inject:jakarta.inject-api:jar:2.0.1:compile
[INFO] |  \- javax.inject:javax.inject:jar:1:compile
[INFO] +- org.apache.geronimo.config:geronimo-config-impl:jar:1.2.3:compile
[INFO] |  \- org.eclipse.microprofile.config:microprofile-config-api:jar:1.4:compile
[INFO] +- org.apache.logging.log4j:log4j-slf4j2-impl:jar:2.23.1:compile
[INFO] |  +- org.apache.logging.log4j:log4j-api:jar:2.23.1:compile
[INFO] |  +- org.slf4j:slf4j-api:jar:2.0.9:compile
[INFO] |  \- org.apache.logging.log4j:log4j-core:jar:2.23.1:runtime
[INFO] +- org.slf4j:jul-to-slf4j:jar:2.0.16:compile
[INFO] \- org.junit.jupiter:junit-jupiter:jar:5.11.0:test
[INFO]    +- org.junit.jupiter:junit-jupiter-api:jar:5.11.0:test
[INFO]    |  +- org.opentest4j:opentest4j:jar:1.3.0:test
[INFO]    |  +- org.junit.platform:junit-platform-commons:jar:1.11.0:test
[INFO]    |  \- org.apiguardian:apiguardian-api:jar:1.1.2:test
[INFO]    +- org.junit.jupiter:junit-jupiter-params:jar:5.11.0:test
[INFO]    \- org.junit.jupiter:junit-jupiter-engine:jar:5.11.0:test
[INFO]       \- org.junit.platform:junit-platform-engine:jar:1.11.0:test
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  3.952 s
[INFO] Finished at: 2024-09-07T00:55:58Z
[INFO] ------------------------------------------------------------------------
```

* DIの中心は Dagger
* 設定値取得は MicroProfile Confing/Apache Geronimo Config
* ログ出力は Slf4j/Log4j2 

定番ですね。 まぁ、`Java 17` 位になると `commons-lang` も `guava` も要らなくなるし `jul-to-slf4j` もケースによっては要らんかもね、と言うお話。

### Maven設定 {#comile_setting}

* [pom.xml](https://github.com/halflite/batch-daggar-example/blob/main/batch/pom.xml)

上記参照。

* APTで生成クラスを作るには、 `maven-compiler-plugin` に設定を加える
* 自動生成クラスをクラスパスに通すには `build-helper-maven-plugin` で解決する

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

### DaggerとJakarta EE {#jakarta_ee}

Daggerも最新のものになると `javax.*` のパッケージが含まれるとコンパイルが通らないようです。 
`jakarta.*` に書き換えれば大丈夫ですが、昔のソースを持ってくる時は、それなりに注意が必要です。

### 結局のところ {#why_batch}

バッチ処理は、本当に小さくするべきだと思います。 Eclipse EE や Spring Batch がメジャーだと思って、それに従って作ると痛い目に会うでしょうね。

バッチのランチャーはOSSの[Rundeck](https://www.rundeck.com/ "Rundeck Runbook Automation")を筆頭に、色々あるので、リランとかリトライはそっちに任せた方が良いでしょうね。 
