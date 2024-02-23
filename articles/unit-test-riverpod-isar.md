---
title: "【Flutter】IsarとRiverpodでのUnit Test"
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, Riverpod, Isar, Unit Test]
published: false
---

# はじめに
FlutterアプリでIsarとRiverpodを組み合わせて使ったアプリを作成したのですが、その際にユニットテストの作成方法を説明します。

なお、動作環境は次のとおりです。

```
Flutter 3.16.9 • channel stable •
https://github.com/flutter/flutter.git
Framework • revision 41456452f2 (4 weeks
ago) • 2024-01-25 10:06:23 -0800
Engine • revision f40e976bed
Tools • Dart 3.2.6 • DevTools 2.28.5

isar: ^3.1.0+1
flutter_riverpod: ^2.4.9

```

## テストの準備
このアプリでは、
1. 起動時にデータが無い場合（初回起動時）はJSONファイルからデータを保存
2. 保存したデータを表示する非同期処理
を実装しています。

それを踏まえ、今回想定しているテストは**初回起動時にアプリにデータが存在するか**です。
データは`dataProvider`で参照し、`dataProvider`はIsarのProviderを参照しています。

Flutterでのテストメソッドは`test()`ですが、全体のテスト実行前に１回だけ実行される`setUpAll`メソッドでまず1.のデータを保存していきます。
`setUpAll`の中身は次のようになります。

```dart: isar_test.dart
void main() {
    late Isar isar;
    late Data testDataProvider;

    setUpAll(() async {
        TestWidgetsFlutterBinding.ensureInitialized();
        await Isar.initializeIsarCore(download: true);
        final dir = await Directory.systemTemp.createTemp();
        isar = await Isar.open([QuestionSchema], directory: dir.path);
        final container = ProviderContainer(
            overrides: [
                isarProvider.overrideWith((FutureProviderRef<Isar> ref) async => isar),
            ],
        );
        testDataProvider = container.read(dataProvider.notifier);
        testDataProvider.save();
    });
}
```
`TestWidgetsFlutterBinding.ensureInitialized()`はWidgetTestでは必須ですが、デバイスのDBを扱うためここでも必要です。
また[Isarの公式ページ]()より、IsarをUnit testする際は、Isarを使う前に`await Isar.initializeIsarCore(download: true)`をしてください、とあります。

そして、今回は一時的なデータベースを作成するだけなので、Directory.systemTemp.createTemp() で一時的なファイルのディレクトリを指定しています。

:::message
ちなみにここでpath_providerの`getApplicationDocumentsDirectory()`などを呼ぼうとすると、MissingPluginExceptionが発生するかと思います。その場合は[こちら](https://qiita.com/teriyaki398_/items/642be2f0ed1e87d8ae38)と[こちら](https://docs.flutter.dev/testing/plugins-in-tests)のページを参照することを推奨します。
:::

最後のところでsaveメソッドを呼んで、データを保存する処理をしているのですが、普通であれば`ref.read(provider.notifier).save()`というような形で、WidgetRefを参照するのですが、どうしたらいいんだ！ってなったときは公式ページを読みましょう。
[Riverpodの公式ページ](https://riverpod.dev/ja/docs/essentials/testing)でも説明されていますが、Riverpodの主なテストの違いはProviderContainer オブジェクトを作成する必要があることです。このオブジェクトにより、テストでプロバイダーと対話できるようになります。
dataProviderはIsarProviderに依存しているため、`ProviderContainer.overrides`でプロバイダを差し替えます。

## テストの後処理
いまのままだとデータが残った状態になってしまい、テスト開始時にデータが無い状態にしたいので、テスト終了時の処理を記述します。
すべてのテストが実行後に１回だけ実行される処理をしたい場合は`tearDownAll`関数に記述します。

```dart: isar_test.dart

void main() {
    late Isar isar;
    late Data testDataProvider;

    setUpAll(() async {
        TestWidgetsFlutterBinding.ensureInitialized();
        .
        .
        .

    });

    tearDownAll(() async {
        await isar.close();
    });
}
```

これでテスト終了後にIsarインスタンスが開放され、データも削除されます。

main関数を実行してみて、エラーが生じず、ルートディレクトリに`libisar.dylib`ファイルが作成されていたら成功です。
こちらのファイルは.gitignoreに記述して追跡不可能にしたほうがいいです。

## テストの記述
最後にデータベースにデータがちゃんと存在するかをテストします。

```dart: isar_test.dart
void main() {
  late Isar isar;
  late Data testDataProvider;
  
  /// すべてのテストが始まる前に一度だけ実行される
  setUpAll(() async {
    TestWidgetsFlutterBinding.ensureInitialized();
    .
    .
    .
    testDataProvider.save();
  });

  tearDownAll(() async {
    await isar.close();
  });

  group("データベースにデータが存在する", () {
    test("Isarインスタンスが存在する", () async {
      expect(isar.isOpen, true);
    });

    test("データベースにデータが存在する", () async {
      expect(testDataProvider.state, isA<AsyncLoading>());
      await testDataProvider.future;
      expect(testDataProvider.state, isA<AsyncData>());
      expect(testDataProvider.state.value!.length, greaterThan(0));
      expect(await isar.questions.count(), greaterThan(0));
    });
  });
}
```

groupで一連のテストをまとめています。
簡単にコードの中身を説明すると、

非同期でデータを取得するため、最初にLoading状態になっているかチェックします。
次に`future()`で処理が終わるのを待ちます。
取得したデータ（=state）が期待している型なのか、またデータは１つ以上あるのかをdataProvider,isarの両方チェックしています。

以上となります。
初見でテストする場合は結構ハマるポイントが多いかなと思ったので、この記事が助けになれば幸いです。
DM等でも質問受け付けています。

マイペースにつぶやいています。
https://twitter.com/yamazaking0

# 参考にさせていただいた記事

https://zenn.dev/riscait/books/flutter-riverpod-practical-introduction/viewer/test

https://zenn.dev/laiso/articles/cfff69685553753ee378

https://qiita.com/teriyaki398_/items/642be2f0ed1e87d8ae38

