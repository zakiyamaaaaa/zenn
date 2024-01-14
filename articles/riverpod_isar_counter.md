---
title: "[Flutter]RiverpodとIsarで作るシンプルカウンターアプリ"
emoji: "⏲️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, Riverpod, Isar]
published: false
---

# はじめに
RiverpodとIsarを組み合わせて簡単なカウンターアプリを作りました。
そもそもこの組み合わせでアプリの機能を実装しようとしたときに、参考となる記事が少ない&情報が古く、一応公式のページにcounterのサンプルアプリはあるのですが、2024年1月時点のメジャーバージョンでは実装できなく、Riverpodとの組み合わせをしたい自分にとっては求めていたコードはありませんでした。
そのせいもあって簡単なアプリを作ろうとしたものの少し手こずってしまったので、同じようにRiverpodとIsarを組み合わせて使いたいけど悩んでいる方の助けになれば幸いです。
今回様々な記事を拝見していって、Riverpodの使い方、Riverpod generatorの使い方が前よりだいぶ理解が深まりました。

完成図

![](/images/riverpod_isar_counter/image1.png)

見た目はよくあるカウンターアプリです。
右下のボタンを押すと、表示されているカウント数が自動的にカウントアップされて、Isarのデータベースが更新されます。
内部データベースに値を保存し、アプリをキルしたあとでも、また起動時に最後に表示された値がリセットされません。

# packageの設定
次のようなpackage構成です。

```dart:pubspec.yaml
environment:
  sdk: '>=3.2.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter
  flutter_riverpod: ^2.4.9
  isar: ^3.1.0+1
  isar_flutter_libs: ^3.1.0+1
  path_provider: ^2.1.2
  riverpod_annotation: ^2.3.3

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^2.0.0
  build_runner: ^2.4.8
  isar_generator: ^3.1.0+1
  riverpod_generator: ^2.3.9

```

## Isarのデータモデル
まずはデータモデルを作っていきます。
といっても、モデルは簡単なものです。
ここではIsarのgeneratorを使うため、3行目のpart ~ のところはファイル名になります。
ここでは、Count.dartというファイル名なので、count.g.dartとなります。

```dart:Count.dart
import 'package:isar/isar.dart';

part 'count.g.dart';

@collection
class Count {
  Id id = Isar.autoIncrement;
  int step;

  Count(this.step);
}

```

エラーが有る状態で構わないので、ターミナルに次のコマンドを入力し、コード生成します。

```bash
dart run build_runner build --delete-conflicting-outputs
```

コード生成が問題なければ、count.g.dartファイルが生成され、エラーが消えるはずです。

## Provider
Providerを生成します。今回のアプリではIsarをグローバルで参照できるIsarProviderとCountのProviderをriverpod generatorを用いて生成します。

### Isar Provider
Isarをグローバルに扱えるようにIsarのProviderを作ります。
Riverpod generatorと一緒に使っていきます。

```dart:isar_provider.dart

import 'package:isar/isar.dart';
import 'package:isar_counter/model/count.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';
import 'package:path_provider/path_provider.dart';

part 'isar_provider.g.dart';

@Riverpod(keepAlive: true)
Future<Isar> isar(IsarRef ref) async {
  final dir = await getApplicationDocumentsDirectory();
  return Isar.open([CountSchema], directory: dir.path, inspector: true);
}

```

ここで若干ハマっていたのですが、Isar.openのところでdirectoryを空文字''と最初やっていました。どこかの記事で読んで荘やっていたと思うのですが、それが原因で上手くIsarを生成することができていませんでした。

inspectorをtrueにすると、↓のようにブラウザ上でデバッグ中のリアルタイムのデータベースが見ることができます。これがIsarの特徴の一つです。

![](/images/riverpod_isar_counter/image2.png)

先ほどと同じようにコード生成します。

```bash
dart run build_runner build --delete-conflicting-outputs
```

今回のように、isarというメソッド名でのNotifier名を作成すると、自動的にisarProviderというプロバイダを生成してくれます。
これのいいところは、自動的に型に合わせてProviderを生成してくれることで、ここではFuture<Isar>という型でのNotifierとなっていて、Provider側はisar_provider.g.dartのファイルを見るとわかるように、FutureProvider<Isar>という型を自動的に生成してくれます。

### Counter Provider
カウント数を監視＋Isarへの情報を更新するProviderを作成します。
Riverpod generatorを使用したクラスについては[Riverpod公式ページ](https://riverpod.dev/ja/docs/concepts/about_code_generation)に詳しい記述があります。

```dart: count_provider.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:isar_counter/provider/isar_provider.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';
import 'package:isar_counter/model/count.dart';
import 'package:isar/isar.dart';

part 'count_provider.g.dart';

// CountのProviderクラスを作成する
@Riverpod(keepAlive: true)
class Counter extends _$Counter {

  @override
  int build() {
    initialize();
    
    return 0;
  }

  void increment() {
    state++;
    update();
  }

  Future<void> initialize() async {
    final isar = await ref.read(isarProvider.future);
    state = await isar.counts.where().findFirst().then((value) => value?.step ?? 0);
  }

  Future<void> update() async {
    final isar = await ref.read(isarProvider.future);
    await isar.writeTxn(() async {
      final count = await isar.counts.where().findFirst() ?? Count(0);
      count.step = state;
      await isar.counts.put(count);
    });
  }
}

```

コードの概要を説明すると

まず、int型を状態としてもちたいので、build関数ではintを返すようにしています。
そして詰まったポイントなんですが、本当はここでstateをIsarデータベースの数値に設定し、return stateとしようとしました。
しかし、このbuild内でIsarを使おうとすると、非同期関数としなければいけなくなり、そうなるとstateがintではなくFuture<int>となってしまい、UI側で状態変更を監視して、ウィジェットを更新するということができませんでした。そのため、今回はinitializeという非同期関数を用意して、そこでisarとstateの処理を行っています。

```dart: count_provider.dart

  @override
  int build() {
    initialize();
    return 0;
  }

  Future<void> initialize() async {
    final isar = await ref.read(isarProvider.future);
    state = await isar.counts.where().findFirst().then((value) => value?.step ?? 0);
  }

```

update関数では、Isarのデータベースでのカウント(=step)を更新しています。

## 画面の作成
Providerができたので、あとは画面を作成するのみです。

```dart:main.dart


import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:isar_counter/provider/count_provider.dart';
import 'package:isar_counter/provider/counter_service_provider.dart';

void main() async {
  runApp(const ProviderScope(child: MainApp()));
}

class MainApp extends StatelessWidget {
  const MainApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Isar Counter',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.cyan),
        useMaterial3: true,
      ),
      home: HomePage(),
    );
  }
}

class HomePage extends ConsumerWidget {
  const HomePage({super.key});

   @override
   Widget build(BuildContext context, WidgetRef ref) {
   final count = ref.watch(counterProvider);

   return Scaffold(
     appBar: AppBar(
       title: Text("Isar Counter"),
     ),
     body: Center(
       child: Column(
         mainAxisAlignment: MainAxisAlignment.center,
         children: <Widget>[
           const Text(
             'You have pushed the button this many times:',
           ),
           Text(
             '${count}',
             style: Theme.of(context).textTheme.headline4,
           ),
         ],
       ),
     ),
     floatingActionButton: FloatingActionButton(
       onPressed: () {
        ref.read(counterProvider.notifier).increment();
       },
       tooltip: 'Increment',
       child: const Icon(Icons.add),
     ), // This trailing comma makes auto-formatting nicer for build methods.
   );
 }
}

```

![](/images/riverpod_isar_counter/image1.png)

こんな感じです。わからないところがあったら、X（Twitter）で聞いてください！

## 参考
以下参考にさせていただいた記事
- [Notifierの使い方:by Jboyさん](https://zenn.dev/joo_hashi/books/2c6c47e3d79b63/viewer/5dd26b)
- [Isarで初期データ投入を試す:by slowhandさん](https://zenn.dev/slowhand/articles/e89bfe33e604bb)
- [riverpod_generatorを使ってプロバイダを簡潔な記法で生成する:by 村松龍之介さん](https://zenn.dev/riscait/books/flutter-riverpod-practical-introduction/viewer/riverpod-generator)