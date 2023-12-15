---
title: "【Flutter】Drift×Riverpod×freezedで非同期にUI更新するサンプルアプリ"
emoji: "🦊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, Drift, Riverpod, freezed]
published: true
---

# はじめに
この記事はデータベースにDrift、非同期処理にRiverpodを使って簡単なサンプルアプリを作ってみた記事となります。
なお、バージョンは次の通り
- Flutter:3.16
- Drift:2.13.2
- flutter_riverpod: 2.4.9

## Driftとは
[公式ページ](https://drift.simonbinder.eu/)
DriftはFlutterとDartのためのリアクティブな永続化ライブラリで、SQLiteの上に構築されています。
特徴としてはSQLのクエリをDrift上のAPIで実行することができます。
マルチプラットフォームに対応しており、モバイルからWebまでのデータベースで構築可能です。[(pub.devより引用)](https://pub.dev/packages/drift)

テーブル構造のデータベースを採用する場合には有力なツールとなります。
今回はfreezed、Riverpodの説明を割愛しますが、それぞれ人気のライブラリであり、一緒に使う機会が多いと思います。

## 今回作成するアプリの概要
今回のサンプルアプリでは、
1. ボタンを押すとデータベースからデータを取得し、それを表示
2. 表示されたリストのお気に入りボタンを押すと、表示が切り替わる
3. データベースのお気に入りフラグが反転する
ということをやっていきます。

おおまかな構成としては、
1. Userデータの集合として、DriftでUserテーブルを定義
2. データをモデルに変換するためにuser_modelをfreezedで定義
3. user_modelを扱うためのrepositoryを作成
4. UIからrepositoryにアクセスして、データを表示

という感じです。
それでは順番にやっていきます。

# アプリの作成
## 環境構築
適当なプロジェクトを作成します。
ライブラリは次のように設定します。

```yaml: pubspec.yaml
dependencies:
  drift: ^2.13.2
  flutter:
    sdk: flutter
  flutter_riverpod: ^2.4.9
  freezed_annotation: ^2.4.1
  path: ^1.8.3
  path_provider: ^2.1.1
  riverpod_annotation: ^2.3.3
  sqlite3_flutter_libs: ^0.5.18

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^2.0.0
  build_runner: ^2.4.7
  drift_dev: ^2.13.2
  freezed: ^2.4.5
  riverpod_generator: ^2.3.9
```
※バージョン番号をこの通りにあわせる必要はありませんが、もし後述のコードでエラー等出た場合には見直してみてください。

## DriftでUserテーブルのデータを作成
Usersというテーブルを定義しています。
id,name,favの３つのフィールドを持ち、idは自動的にインクリメント。nameはString、favはbooleanでデフォルトでfalseを定義していきます。
`part 'user_table.g.dart'; `のところは、ファイル名.g.dartとしてください。エラーが出ると思いますが、この後にbuild_runnerを使うとエラーが解消されるのでこの時点では機にしないでください。

Driftのサポートしている型は次のとおりです

![](/images/riverpod_drift/drift_type.png)

また、Nullableの指定もでき、その場合は
```dart
  IntColumn get category => integer().nullable()();
```
という書き方です。


```dart:user_table.dart
import 'dart:io';
import 'package:drift/drift.dart';
import 'package:path/path.dart';
import 'package:drift/native.dart';
import 'package:path_provider/path_provider.dart';

part 'user_table.g.dart';

class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  BoolColumn get fav => boolean().withDefault(const Constant(false))();
}
```

DriftDatabaseクラスをアノテーションとして使用して、指定された DriftDatabase.tables を使用してデータベース クラスを生成する必要があることをジェネレーターに通知します。
データベースクラスを作成するには、まず空のクラスに DriftDatabase のアノテーションを付け、`dart pub run build_runner build` を使ってビルドランナーを実行します。Driftがデータベースクラスと同じ名前のクラスを生成します。このクラスを拡張し、QueryExecutorを提供することで、driftを使用することができます

schemaVersionはデータベースのバージョンです。既存のデータベースから構造等を変更する場合は、数字を増やしてください。今回は最初なので1で大丈夫です。

DriftにはMigrationStrategyというAPIがあり、これを用いて、データ作成時やアップグレード時などでの処理を書いていきます。ここでは、`gon,killa,kurapika`という架空の人物名をいれました。


```Dart: user_table.dart
@DriftDatabase(tables: [Users])
class AppDatabase extends _$AppDatabase { // _$AppDatabase was generated
  AppDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 1;

  @override
  MigrationStrategy get migration => MigrationStrategy(
    onCreate: (m) async {
      await m.createAll();
      await into(users).insert( const UsersCompanion(name: Value('gon')));
      await into(users).insert( const UsersCompanion(name: Value('killa')));
      await into(users).insert( const UsersCompanion(name: Value('kurapika')));
    },
    onUpgrade: (m, from, to) async {},
  );
}
```

最後に接続先のDBを指定します。

```dart: user_table.dart
LazyDatabase _openConnection() {
  return LazyDatabase(() async {
    final dbFolder = await getApplicationDocumentsDirectory();
    final file = File(join(dbFolder.path, 'db.sqlite'));
    return NativeDatabase(file);
  });
}
```

Tableの設定はここまでで終わりです。お疲れさまでした。


## freezedを用いたモデルの定義
先程のUsersテーブルを参考にユーザーモデルを作成していきます。
partの部分では、ファイル名を記述してください。
渡しの場合、`user_model.dart`というファイルにUserModelを記述しているので、`part 'user_model.freezed.dart'`という書き方になります。

```Dart: user_model.dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'user_model.freezed.dart';

@freezed
class UserModel with _$UserModel {
  const UserModel._();
  const factory UserModel({
    required int id,
    required String name,
    @Default(false) bool fav,
  }) = _UserModel;
}

```

テーブルとモデルができたところで、コード生成を行います。
ターミナルで次のコマンド

```
dart run build_runner build
```

ここまでうまくいけば、エラーは表示されなくなると思いますが、表示される方はライブラリのバージョン番号等も含めて見直してみてください。

## Repositoryの作成

RepositoryはUIからアクセスするためのものとして今回は作っていきます。
機能としては
・データをフェッチする
・Favボタンを押したときにfavのフラグを反転させる
の２つのみです。
RiverpodのStateNotifierクラスを継承したUserNotiferクラスを定義していきます。

```dart:user_ripository.dart
import 'dart:async';

import 'package:drift_freeze_riverpod_practice/model/user_table.dart';
import 'package:drift_freeze_riverpod_practice/model/user_model.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

class UserNotifier extends StateNotifier<List<UserModel>> {
  final AppDatabase database;
  UserNotifier({ required this.database }) : super([]);

  Future<void> fetchData() async {
    final users = await database.select(database.users).get();
    state = users.map((user) => UserModel(
      id: user.id,
      name: user.name,
      fav: user.fav,
    )).toList();
  }

  void toggleFav(int id) {
    state = state.map((e) {
     if (e.id == id) {
        return e.copyWith(fav: !e.fav);
      }
      return e;
    }).toList();
  }
}

```

## UIからrepositoryにアクセスして、データを表示
それでは最後の章です。
Riverpod初心者あるあるの、mainメソッド内にあるrunAppでProviderScopeをいれるのを忘れないでください。

```dart: main.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:drift_freeze_riverpod_practice/model/user_table.dart';
import 'package:drift_freeze_riverpod_practice/model/user_model.dart';
import 'package:drift_freeze_riverpod_practice/model/user_repository.dart';
import 'package:riverpod/riverpod.dart';



void main() {
  final database = AppDatabase();
  runApp(
    ProviderScope(
      child: MaterialApp(
        home: MainApp(db: database),
      ),
    )
  );
}

final databaseProvider = Provider<AppDatabase>((ref) => AppDatabase());

final userProvider = StateNotifierProvider<UserNotifier, List<UserModel>>((ref)=> UserNotifier(database: ref.watch(databaseProvider)));


class MainApp extends ConsumerWidget {
  final AppDatabase db;
  const MainApp({super.key, required this.db});

  @override
Widget build(BuildContext context, WidgetRef ref) {
  final data = ref.watch(userProvider);
  return Scaffold(
      appBar: AppBar(
        title: const Text('Data Display'),
      ),
      body: Column(
        children: [
          ElevatedButton(
            child: const Text('Fetch Data'),
            onPressed: () {
              ref.read(userProvider.notifier).fetchData();
            },
          ),
          Expanded(
            child: ListView.builder(
              itemCount: data.length,
              itemBuilder: (context, index) {
                return ListTile(
                  title: Text('Name: ${data[index].name}, ID: ${data[index].id}'),
                  trailing: IconButton(
                    icon: Icon(data[index].fav ? Icons.favorite : Icons.favorite_border),
                    onPressed: () => ref.read(userProvider.notifier).toggleFav(data[index].id),
                  ),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}

```

ここでは簡略化のため、main.dartファイルにざっと書いていますが、本来であれば別ファイルにして分けるのが適切です。
簡単にコードの説明をすると、Widget buildの中で`final data = ref.watch(userProvider);`として、データを監視します。
そして、ボタンを押した際に`ref.read(userProvider.notifier).fetchData()`が実行され、userProviderの変更が監視していたdataに反映され、`ListView.builder〜`のところが描画されます。
簡単なサンプルでしたが、Drift,freezed,Riverpodのモダンな技術を使って、どのような流れで動くかを説明しました。

## 補足：データベースの中身の確認
今回はじめてDriftを触ってみて、若干扱いづらいところがあったのがデータベースにちゃんと値が入っているかを確認するところでした。
普段はSimulatorをiOSで使っていますが、その際のデータベースの確認方法を補足的に説明します。
VisualStudioCode（VS Code）を使っているので、ここではVS Codeを使ったやり方となります。

iOSの場合は、Library/Developer/CoreSimulator/Devicesから直近つかってるフォルダ（SimulatorのID？）/data/Containers/Data/Applications/直近のフォルダ/Documentsの配下にdb.sqliteがあります。

VS CodeにSQLiteのExtensionをインストールします。エクステンション名は`SQLite`
https://marketplace.visualstudio.com/items?itemName=alexcvzz.vscode-sqlite

Cmd+Shit+Pでプロンプトを開いて、そこからSQlite: Open Databaseを選択します。
さきほどのdb.sqliteを選択します。
選択しても何も画面上起こりません。
VS Code左上のExplorerを選択すると、SQLITE EXPLORERというセクションが表示されているはずです
![](/images/riverpod_drift/sqlite.png)

こちらをクリックすると、dbの中身が表示されると思います。
![](/images/riverpod_drift/sqlite2.png)

さらにここのテーブル（今回はusers）を右クリックしてShow Tablesとすると、テーブルの中身が表示されます。
![](/images/riverpod_drift/sqlite3.png)

一応このようにデータベースの中身は確認できるのですが、いちいちデータベースのファイルを探しに行くのが面倒な感じです。
NoSQLで有名なデータベースライブラリの[Isar](https://isar.dev/ja/)はその点、ビルド時にデータベースのリンクを生成し、ブラウザ上でリアルタイムにデータの確認ができるため、その点はとても使い勝手がよく感じます。

# 参考サイト
- [Drift公式ページ](https://drift.simonbinder.eu/)
- [Isar公式ページ](https://isar.dev/ja/)

