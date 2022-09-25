---
title: "AWS Lambda FunctionでJavaを採用したい"
date: 2022-09-25T00:00:00+09:00
draft: false
tags:
  - java
  - aws
---

### 承前

* `AWS Lambda Function` を `Java` で作るというお仕事をしています
    * 基本的に `Spring-Boot` がメインの現場です
    * `Spring-Framawaork` はやめておけ、と公式でも言われてますね。[^1]
* `Java` で何かしらする時は、背骨(フレームワーク)欲しいですよね
    * `DI(DependencyInjection)` 無いと、もう無理ですよね
* `Guice`/`Daggaer` を使えと言われていますが、どっち使ったらいいの？
    * 色々なケースがありますが

### 自分でやったこと

* 自分でやったこと
    * [google/guice](https://github.com/google/guice/wiki)
    * [Apache Geronimo Config](https://geronimo.apache.org/microprofile/index.html)
    * [google/guava](https://github.com/google/guava/wiki)

で、どうやってビジネスロジック注入する？を考えて、やっぱり[google/guice](https://github.com/google/guice/wiki)かなぁって。

実行時コンパイルでも、かなり速い。 まぁ、そもそも `AWS Lambda Function` って、小さいですしね [^2]

設定注入は、[Apache Geronimo Config](https://geronimo.apache.org/microprofile/index.html)かな。

### ケース

まぁまぁ、よくあるケースですが、 `API Gateway` で、何かを送ってもらって、何かをやるみたいなことをやってみましょうか。

```java
public class App implements RequestHandler<APIGatewayProxyRequestEvent, APIGatewayProxyResponseEvent> {

  private final AnyService anyService;

  @Override
  public APIGatewayProxyResponseEvent handleRequest(APIGatewayProxyRequestEvent input, Context context) {
    this.anyService.execute(input);
    APIGatewayProxyResponseEvent res = ... // do something
    return res;
  }

  public App() {
    Injector injector = Guice.createInjector(new AppModule());
    this.anyService = injector.getInstance(AnyService.class);
  }
}
```

```java
public class AppModule extends AbstractModule {

  @Override
  protected void configure() {
    bind(AnyService.class);

    // bind microprofile-config.properties
    final Map<String, String> props = new HashMap<>();
    Sets.newLinkedHashSet(ConfigProvider.getConfig().getConfigSources())
        .stream()
        .map(ConfigSource::getProperties)
        .forEach(props::putAll);
    Names.bindProperties(this.binder(), props);
  }
}
```

まぁー、こんな感じですかね？

[^1]: [AWS Lambda 関数を使用するためのベストプラクティス - AWS Lambda](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/best-practices.html "AWS Lambda 関数を使用するためのベストプラクティス - AWS Lambda")
[^2]: 小さく作るべきですよね