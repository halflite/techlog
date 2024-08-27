---
title: "2024年8月 Javaプログラミングで \"Hello World\" を出したいだけだった"
date: 2024-08-25T00:00:00+09:00
draft: false
tags:
  - java
---

前回、[2021年6月 Javaプログラミングで "Hello World" を出したいだけだった](../20210627_java_dev/) の続き。 と言うか、自分で何かしらの結果を出したいですしね。

開発環境は、 VSCode と Rancher Desktop だけでやってみます。

* [自分で作ってみた](#myapp)
* [結構面倒くさいポイントその1 Guice-Servlet初期化](#guiceservletinit)
* [結構面倒くさいポイントその2 FreemarkerServlet のパッケージ](#jakartapackage)
* [結構面倒くさいポイントその3 FreemarkerServlet の設定値](#freemarkercongig)
* [結構面倒くさいポイントその4 java.util.logging(jul)](#julltoslfj)

_____

#### 自分で作ってみた {#myapp}

* [halflite/guice-freemarker-servlet: Guice / Freemarker / Servlet](https://github.com/halflite/guice-freemarker-servlet)

で、自分で作ってみました。結構面倒くさい！

これに関しては、幾つかの縛りがあります。
例えば「"Hello World"を出せるくらいの小さいアプリを…」と言われた時、大抵下記のような前提が言外にあると思っても良いでしょう。

* 立ち上げるのには、それなりの速さが欲しいよね
* 当然、仮想化が前提にあるよね
* それなりにDI出来るコンテナ欲しいよね
* 設定値をDI出来る仕組み欲しいよね
* HTMLテンプレートも欲しいよね
* APIとして使うことを考えてJSONも出力して欲しいよね
* ログを統一的に出す機構が欲しいよね

…で、そこら辺を踏まえた上での、自分が考えるアーキテクチャ。

* サーバーはJetty Embedded
* DIは、Guice/Guice-Servletに従う
* 設定値は取得は Apache Geronimo(MicroProfile Config)
* HTMLテンプレートは(多分)一番シンプルな FreeMarker
* JSON出力は、一番軽量で早い Gson
* ログは Slf4j/Log4j2 

JavaでWebアプリケーションで避けて通れないのが、Java EE -> Jakarta EE への移行でしょう。 あれ、単にパッケージを変えただけでも大概OKなんですが、それでもソース書き換えて再ビルドは、相当きついかもですよ。

だから、ちょっとしたWebアプリケーションは、小さく作って、いつでも交換できるように疎結合にしておく、が、大切ですかね。

#### 結構面倒くさいポイントその1 Guice-Servlet初期化 {#guiceservletinit}

* [Servlets · google/guice Wiki](https://github.com/google/guice/wiki/Servlets)

上記を読むと、よくあるコンテナ(Tomcat)とかにデプロイすることを前提として、 `GuiceFilter` の設置を書いているのですが、そもそも、アプリコンテナ全体をGuiceのDI対象に含めてしまえば、わざわざその設定も不要ということです。

ここら辺を分からないと「…？」と悩むことになります。

_____

#### 結構面倒くさいポイントその2 FreemarkerServlet のパッケージ {#jakartapackage}

昔のFreeMarkerには、`freemarker.ext.servlet.FreemarkerServlet` ってクラスがあったんですが、今ではそれが依存するServletのパッケージが違うので、Jakarta EE対象のコンテナを使おうとしても、コンパイルが通りません。 `freemarker.ext.jakarta.servlet.FreemarkerServlet` に変える必要があるのです

#### 結構面倒くさいポイントその3 FreemarkerServlet の設定値 {#freemarkercongig}

`FreemarkerServlet` に関しては、Tomcatとかに書く `web.xml` には記述あるんですが、コレ、スタンドアローンの時にどうやって設定したら良いの？みたいなのがあります。

わたくしの個人解としては、まずプロパティファイルに、こう書く。[microprofile-config.properties](https://github.com/halflite/guice-freemarker-servlet/blob/main/app/src/main/resources/META-INF/microprofile-config.properties)

```
# FreeMarker Parameters
freemarker.TemplatePath = classpath:views
freemarker.NoCache = true
freemarker.ResponseCharacterEncoding = fromTemplate
freemarker.ExceptionOnMissingTemplate = true
freemarker.incompatible_improvements = 2.3.28
freemarker.template_exception_handler = rethrow
freemarker.template_update_delay = 0
freemarker.default_encoding = UTF-8
freemarker.output_encoding = UTF-8
freemarker.locale = ja_JP
freemarker.number_format = 0.##########
freemarker.time_zone = JST
freemarker.date_format = yyyy/MM/dd HH:mm:ss
```

そして、DI設定の時に、こう書く [ConfigModule.java](https://github.com/halflite/guice-freemarker-servlet/blob/main/app/src/main/java/app/inject/ConfigModule.java)

```java
  /** FreeMarker設定値を作成する */
  @Provides
  @Singleton
  @Named("freemarker.init.parameters")
  public Map<String, String> prividesFreeMarker(Config config) {
    Map<String, String> params = StreamSupport.stream(config.getConfigSources().spliterator(), false)
        .map(ConfigSource::getProperties)
        .map(Map::entrySet)
        .flatMap(Set::stream)
        .filter(e -> e.getKey().startsWith("freemarker."))
        .map(e -> new AbstractMap.SimpleImmutableEntry<String, String>(
            e.getKey().replaceFirst("^freemarker\\.", ""),
            e.getValue()))
        .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue, (e1, e2) -> e1));
    LOG.debug("freemarker init params: {}", params);
    return params;
  }
```

で、DIで取ってこれた値を、 `FreemarkerServlet` に渡せば良い、と言う寸法です。 
[AppContextListener.java](https://github.com/halflite/guice-freemarker-servlet/blob/main/app/src/main/java/app/inject/AppContextListener.java "guice-freemarker-servlet/app/src/main/java/app/inject/AppContextListener.java at main · halflite/guice-freemarker-servlet")

#### 結構面倒くさいポイントその4 java.util.logging(jul) {#julltoslfj}

これ、Javaプログラムやってる人には、永遠の課題なんですが、ログ問題はとにかく面倒くさいです。
自分は、もう割り切って、メインはlog4j2、ファサードはSlf4j、にしています。

だから、まず依存ライブラリに [jul-to-slf4j](https://mvnrepository.com/artifact/org.slf4j/jul-to-slf4j "Maven Repository: org.slf4j » jul-to-slf4j")を含め、更にアプリ立ち上げ最初の方で、それなりの記述が必要です。 [App.java](https://github.com/halflite/guice-freemarker-servlet/blob/main/app/src/main/java/app/App.java)

```
    SLF4JBridgeHandler.removeHandlersForRootLogger();
    SLF4JBridgeHandler.install();
```

_____

いやはや、"Hello World"だけでも、こんなに難しい2024年でしたね。
