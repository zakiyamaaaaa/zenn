---
title: "Riverpodを使ってJsonを非同期表示"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Riverpod, Flutter]
published: false
---

# はじめに
Riverpodを理解するための記事になります。
今回はRiverpodのFutureProviderを使用して、内部のjsonデータを取得し、表示するアプリを実装していきます。

FutureProviderはRiverpodの提供するProviderの一つで、データの非同期処理などで使用されます。

## packageの設定
**pubspec.yaml**のpackage設定は次のとおりです。（バージョン設定は基本的に最新のバージョンをまずは使ってみてください。エラーが出るようであれば、今回記述したバージョンを使用してみてください。）

```pubspec.yaml
dependencies:
  flutter_riverpod: ^2.4.9
  json_serializable: ^6.7.1
  riverpod_annotation: ^2.3.3

dev_dependencies:
  build_runner: ^2.4.7
  riverpod_generator: ^2.3.9

```

## jsonファイルの設定
これまでFlutterで内部ファイルを扱ったことがない人は、少々戸惑うかもしれません。
まずプロジェクトの直下にassetsフォルダーを作成し、その中にconfig.jsonを作成します。
project_folder
|
|-lib
|-assets
    |- config.json

config.jsonは次のように

```config.json
{
    "appName": "Sample App",
    "isDebug": true,
    "age": 18,
    "category" : ["education", "utility"]
}
```

次にこのassetsファイルのパスをpubspec.yamlに記述します。

```pubspec.yaml
flutter:
  assets:
    - assets/
```

これで内部ファイルの設定は完了です。

## Provider
config.jsonの情報をデコードし、Providerを作成します。

```config_provider.dart
import 'package:flutter/services.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';
import 'dart:convert';

part 'config_provider.g.dart';

@Riverpod(keepAlive: true)
Future<Map<String, Object?>> config(ConfigRef ref) async {
  final jsonString = await rootBundle.loadString("assets/config.json");
  final content = json.decode(jsonString) as Map<String, Object?>;
  return content;
}

```

この後に**dart run build_runner build --delete-conflicting-outputs**コマンドをターミナルから実行してください。
また、riverpodを使わない場合は次のような書き方になります。

```config_provider.dart
final configProvider = FutureProvider<Map<String, Object?>>((ref) async {
  final jsonString = await rootBundle.loadString("assets/config.json");
  final content = json.decode(jsonString) as Map<String, Object?>;
  return content;
});
```

## 表示部分
今回はただjsonの表示を試すだけのシンプルなアプリなので、main.dartの中にページ部分を作っています。

```main.dart

import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:riverpod_practice_1/provider/config_provider.dart';

void main() {
  runApp(const ProviderScope(child: MainApp()));
}

class MainApp extends StatelessWidget {
  const MainApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: Scaffold(
        body: ConfigPage(),
      ),
    );
  }
}

class ConfigPage extends ConsumerWidget {
  const ConfigPage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final config = ref.watch(configProvider);
    return Scaffold(
      appBar: AppBar(
        title: const Text('Config'),
      ),
      body: config.when(
        data: (data) => ListView(
          children: data.entries.map((e) {
            return ListTile(
              title: Text(e.key),
              subtitle: Text(e.value.toString()),
            );
          }).toList(),
        ),
        loading: () => const Center(
          child: CircularProgressIndicator(),
        ),
        error: (error, stackTrace) => Center(
          child: Text(error.toString()),
        ),
      ),
    );
  }
}

```

config.whenのところで、loading状態、エラーが生じた場合、データ取得の場合とスッキリとかけることがわかります。

-----

# 参考
[riverpod_generatorを使ってプロバイダを簡潔な記法で生成する](https://zenn.dev/riscait/books/flutter-riverpod-practical-introduction/viewer/riverpod-generator)
