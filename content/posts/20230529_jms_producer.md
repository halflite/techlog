---
title: "ActiveMQとQuarkus、Apache Camelで、画像アップロード処理をしたい。 その1 Producer 編"
date: 2023-05-29T00:00:00+09:00
draft: false
tags:
  - java
  - quarkus
  - apache-camel
---

### 承前

- `ブラウザ->ウェブアプリ->ActiveMQ->ApacheCamel->AWS S3` と言うルートを考えてみましょう。
    - ウェブアプリでEXIF情報を消去したり、画像縮小を行ってS3にアップロードするには、少々重たい処理ですね。
        - 非同期で、イベント駆動で処理したいですよね。
    - ウェブアプリはDockerで建てているのだから、`ActiveMQ`[^1] も Consumerである `Apache Camel`[^2] もDockerでインスタンス作って、メッセージ送信/受信したらいいよね。
    - …と、思ったら、意外と大変だった、と言う感じです。

### JMS(Java Message Service) と Producer と Consumer

- JMSは歴史的経緯から、とにかく複雑な仕様で、正直に向き合うと、げんなりしてしまうのですけどね…。
- まぁ、シンプルに`Producer->ActiveMQ->Consumer`で、送信元`Producer`が、受信元`Consumer`に向けて、オブジェクトをメッセージとして送る、オブジェクトのエンコード/デコードを簡単に(？)やってくれる、と、考えたら良いと思います。
    - 今回は`ウェブアプリ`が`Producer`の役割ですね。

### ウェブアプリケーションに画像アップロード

- 結局、ファイル名と拡張子を文字列で取れて、画像のデータを`byte[]`か、`InputStream`で取れれば良いと思います。
    - [マルチパートでのRESTクライアントの使用 - Quarkus](https://ja.quarkus.io/guides/rest-client-multipart)
        - 依存関係に [Quarkus RESTEasy Multipart Runtime](https://mvnrepository.com/artifact/io.quarkus/quarkus-resteasy-multipart)を忘れずに。

### 画像データをActiveMQに送信する

- [JMSの使用 - Quarkus](https://ja.quarkus.io/guides/jms)
    - 依存関係に [Quarkus Artemis JMS Runtime](https://mvnrepository.com/artifact/io.quarkiverse.artemis/quarkus-artemis-jms)を忘れずに。
    - `application.properties`に`quarkus.artemis.***=`を記述することを忘れないように。
    - そうすると、`ConnectionFactory`もDI対象になりますよ。

```java
  public void uploadImage(String fileName, InputStream file) {
    try (JMSContext context = this.connectionFactory.createContext(JMSContext.AUTO_ACKNOWLEDGE);
        ByteArrayOutputStream out = new ByteArrayOutputStream()) {
      // ファイル読み込み
      file.transferTo(out);
      out.flush();

      // メッセージ作成
      BytesMessage message = context.createBytesMessage();
      message.setStringProperty("fileName", fileName);
      message.writeBytes(out.toByteArray());
      
      // トピックを送信
      context.createProducer().send(context.createTopic("upload.image"), message);
    } catch (IOException | JMSException e) {
      throw new RuntimeException(e);
    }
  }
```

[^1]: [ActiveMQ Artemis](https://activemq.apache.org/components/artemis/ "ActiveMQ Artemis")
[^2]: [Apache Camel user manual :: Apache Camel](https://camel.apache.org/manual/index.html "Apache Camel user manual :: Apache Camel")

