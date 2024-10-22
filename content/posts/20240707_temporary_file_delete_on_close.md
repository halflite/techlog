---
title: "続：Java NIO2 で一時ファイルを作成、処理後に削除"
date: 2024-07-07T00:00:00+09:00
draft: false
tags:
  - java
---

* [前回](/techlog/posts/20200222_temporary_file_delete_on_close/ "Java NIO2 で一時ファイルを作成、処理後に削除 | ひとり開発日記。")の修正版です

```java
  public void doSomething() {
    try {
      Path tempFilePath = Files.createTempFile("id", ".tmp");
      try (Writer writer = Files.newBufferedWriter(tempFilePath)) {
        // do something.
        // 終わった後に一時ファイルを削除したい
      }
    } catch (IOException e) {
      // ログとか出したり、または、別の例外再スロー
    }
  }
```

前回書いた `Files.newBufferedWriter(tempFilePath, StandardOpenOption.DELETE_ON_CLOSE)` って、OSや権限で効かない場合もある、って知りました。 って言うか、自分で地雷を踏んでしまったんですけどね。

で、簡易的な解決だと、以下のようにしたら良いと思います。

```java
  public void doSomething() {
    try {
      Path tempFilePath = Files.createTempFile("id", ".tmp");
      try (Closeable closeable = () -> Files.deleteIfExists(tempFilePath);
        Writer writer = Files.newBufferedWriter(tempFilePath)) {
        // do something.
      }
    } catch (IOException e) {
      // ログとか出したり、または、別の例外再スロー
    }
  }
```

`Closeable` の匿名クラスを作って、try-with-resources 構文の中で、一時ファイルを削除する、と言う処理ですね。
`Java 17` で書きましたが、 `Java SE 8` でも動くと思います。

