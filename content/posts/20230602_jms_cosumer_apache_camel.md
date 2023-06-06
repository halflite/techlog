---
title: "ActiveMQとQuarkus、Apache Camelで、画像アップロード処理をしたい。 その2 Consumer/Apache Camel編"
date: 2023-06-02T00:00:00+09:00
draft: false
tags:
  - java
  - quarkus
  - apache-camel
---

### TL;DR

- [Quarkus - Start coding with code.quarkus.redhat.com](https://code.quarkus.redhat.com/ "Quarkus - Start coding with code.quarkus.redhat.com") で、雛形を作ります。
- [`camel-quarkus-sjms2`](https://mvnrepository.com/artifact/org.apache.camel.quarkus/camel-quarkus-sjms2) / [`quarkus-artemis-jms`](https://mvnrepository.com/artifact/io.quarkiverse.artemis/quarkus-artemis-jms) を依存関係に含めます。
- ルート用のクラスは、DI対象に含めます。
- `ActiveMQ Altemis`用の設定を、`application.properties`に記します。

### 前回

- [ActiveMQとQuarkus、ApacheCamelで、画像アップロード処理をしたい。 その1 Producer 編](/techlog/posts/20230529_jms_producer/)

### Apache CamelとQuarkusの第一歩

- [Camel Extensions for Quarkus と Camel K の入門 - 赤帽エンジニアブログ](https://rheb.hatenablog.com/entry/2022/06/28/120434)
    - はい、この通りに、Maven/Gradeleでプロジェクトを作って、実行してみましょう。[^1]
    - [Quarkus - Start coding with code.quarkus.redhat.com](https://code.quarkus.redhat.com/ "Quarkus - Start coding with code.quarkus.redhat.com")

### Routeの修正

上記のサンプルだと、少々使いづらい。 `Routes`クラスを以下のように修正します。

- 継承する親クラスを `EndpointRouteBuilder` にします。[^2]
    - タイプセーフで、[「流れるようなインターフェース」](https://bliki-ja.github.io/FluentInterface/ "流れるようなインタフェース - Martin Fowler's Bliki (ja)")でルートが書けるようになります。
- クラスに `@ApplicationScoped` アノテーションを付けます。
    - `Routes`クラスがDI対象に含まれます。

### ActiveMQからのメッセージ受信

- [`camel-quarkus-sjms2`](https://mvnrepository.com/artifact/org.apache.camel.quarkus/camel-quarkus-sjms2)を依存関係に加えます。
    - 諸々必要なコンポーネントが自動で入ります。
    - ActiveMQ/JMSを通じて、メッセージを受信するためのコンポーネントには、[`camel-activemq`](https://camel.apache.org/components/3.20.x/activemq-component.html) / [`camel-jms`](https://camel.apache.org/components/3.20.x/jms-component.html)があります。
        - このコンポーネントには、`Spring Framework`のライブラリが依存で入ってしまうのです。
        - ネット上のサンプルプログラムは、`Spring-Boot`ベースのConsumerが多いからでしょうね。
- [`quarkus-artemis-jms`](https://mvnrepository.com/artifact/io.quarkiverse.artemis/quarkus-artemis-jms)を依存関係に加えます
    - `camel-quarkus-sjms2`には、ActiveMQとの通信をする、`ConnectionFactory`の実装クラスが入ってないのです。
    - [Artemis JMS](https://ja.quarkus.io/guides/jms#artemis-jms)の記述のように`application.properties`に設定を記述すると、`ConnectionFactory`もDI対象になります。
    - [Quarkus Artemis BOM](https://mvnrepository.com/artifact/io.quarkiverse.artemis/quarkus-artemis-bom)は、Quarkusと違うので、別途依存関係に追加します。

```java
@ApplicationScoped
public class Routes extends EndpointRouteBuilder {
  private final ConnectionFactory connectionFactory;

  @Override
  public void configure() throws Exception {
    from(sjms2("topic:upload.image").connectionFactory(this.connectionFactory))
        // TODO 何かを処理
        .end();

  @Inject
  public MainRoute(ConnectionFactory connectionFactory,) {
    this.connectionFactory = connectionFactory;
  }
}
```

`Routes`クラスは、このような記述になるでしょうか。

[^1]: コンソールにログが出ますので、それで動作確認でしょうか。
[^2]: [Endpoint DSL :: Apache Camel](https://camel.apache.org/manual/Endpoint-dsl.html "Endpoint DSL :: Apache Camel")

