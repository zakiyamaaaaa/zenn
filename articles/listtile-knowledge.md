---
title: "【Flutter】様々なListTileを理解する"
emoji: "🟰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, dart]
published: true
---

# ListTileとは
FlutterでのListTileとは、iOSでいうところのTableViewCell、AndroidでいうところのListViewのlist itemに相当します。
アプリでの使われることの多いウィジェットの一つで、設定などの画面で段組みに項目を作成する際などに使われます。

```dart
class MainApp extends StatelessWidget {
  MainApp({super.key});

  final animals = [
    (animal: 'Dog', emoji: '🐶', icon: Icons.pets),
    (animal: 'Cat', emoji: '🐱', icon: Icons.abc),
    (animal: 'Bird', emoji: '🐤', icon: Icons.ac_unit)
  ];

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: ListView(
            children: animals
                .expand((e) => [
                      ListTile(
                        title: Text(e.animal),
                        leading: Icon(e.icon),
                        subtitle: Text(
                            '${e.emoji} This is a ${e.animal.toLowerCase()}'),
                      ),
                      const Divider(),
                    ])
                .toList(),
          ),
        ),
      ),
    );
  }
}
```

![](/images/listtile-knowledge/image1.png)

詳しくはこちらを参照してください
https://youtu.be/l8dj0yPBvgQ

# ListTileの種類
FlutterでのListTileは一般的なListTileから、CheckBoxListTile、RadioListTile、SwitchListTileと、ネイティブよりも、細かくウィジェットの種類が用意されています。それぞれの挙動、仕様の違いについて理解していきます。

## ListTile
ListTileの基本となります。タイルのアイコンは先頭と末尾のパラメータで定義されます。
* `isThreeLine = true`の場合は、subtitleのスペースが２行割当られます。subtitleのテキストがそれ以上であれば、isThreeLineの値に限らず、自動的に割り当てられます。デフォルトでは`false`になっています。

| isThreeLine = true | isThreeLine = false |
| ---- | ---- |
| ![](/images/listtile-knowledge/image2.png) | ![](/images/listtile-knowledge/image1.png) |

* `dense = true`の場合、このタイルの全体の高さと、タイトルとラブタイトルウィジェットをラップするDefaultTextStylesのサイズは縮小します。
* このウィジェットには、それ自体を描画するツリー内のマテリアルウィジェットの祖先が必要になってきます。これは、アプリのScaffoldによって提供されることが一般的です。
* ListTileのColorプロパティ（tileColor, selectedColor, focusColor, hoverColor）はListTile自体ではなく、祖先にあるマテリアルウィジェットによって描画されます。そのため以下のような書き方ができます。

```dart
const ColoredBox(
  color: Colors.green,
  child: Material(
    child: ListTile(
      title: Text('ListTile with red background'),
      tileColor: Colors.red,
    ),
  ),
)
```
しかし、多数のListTileを個別に`Material`でラッピングするのはコストがかかるため、必要なListTilesに対してラッピングするか、共通のMaterialを祖先とすることが望ましいです。なお、他の`CheckBoxListTile`,`RadioListTile`,`SwitchListTile`も同様に必要となります。

### アイコン
* タップ可能な先頭と末尾のウィジェットサイズは少なくとも48x48である必要があります。しかし、Material仕様に準拠するために、１行のListTileの末尾と先頭のウィジェットは視覚的に高さが最大32(dense: true) or 40(dense: false)であるべきで、アクセシビリティの要件とコンフリクトする可能性があります。
* このため、１行のListTileでは、先頭と末尾のウィジェットの高さをListTileの高さに制約することができます。これにより、十分な大きさのタップ可能な先頭と末尾のウィジェットを作成できます。

ListTileは便利ですが、実現したいものが当てはまらないときは、自分でオリジナルのウィジェットを作ることも検討してください。

## CheckBoxListTile
https://youtu.be/RkSqPAn9szs

```dart
class MainApp extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => _MainApp();
}

class _MainApp extends State<MainApp> {
  final animals = [
    Animal(animal: 'Dog', emoji: '🐶', icon: Icons.pets, checked: false),
    Animal(animal: 'Cat', emoji: '🐱', icon: Icons.abc, checked: false),
    Animal(animal: 'Bird', emoji: '🐤', icon: Icons.ac_unit, checked: false)
  ];

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: ListView(
            children: animals
                .expand((e) => [
                      CheckboxListTile(
                        value: e.checked,
                        onChanged: (v) {
                          setState(() {
                            e.checked = v!;
                          });
                        },
                        title: Text(e.animal),
                        subtitle: Text(
                            '${e.emoji} This is a ${e.animal.toLowerCase()}'),
                      ),
                      const Divider(),
                    ])
                .toList(),
          ),
        ),
      ),
    );
  }
}

class Animal {
  String animal;
  String emoji;
  IconData icon;
  bool checked;

  Animal(
      {required this.animal,
      required this.emoji,
      required this.icon,
      required this.checked});
}
```
![](/images/listtile-knowledge/image3.png =300x)

* チェックボックスは日本語、英語のように左から右に読む言語では、デフォルトで右に表示されます。これは`controlAffinity`を使って変更することができます。

## RadioListTile
* ラジオボタンつきのListTileです。

```dart
class MainApp extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => _MainApp();
}

class _MainApp extends State<MainApp> {
  final animals = [
    Animal(animal: 'Dog', emoji: '🐶', icon: Icons.pets, checked: false),
    Animal(animal: 'Cat', emoji: '🐱', icon: Icons.abc, checked: false),
    Animal(animal: 'Bird', emoji: '🐤', icon: Icons.ac_unit, checked: false)
  ];
  int selectedAnimal = 0;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: ListView.builder(
            itemCount: animals.length,
            itemBuilder: (context, index) {
              final e = animals[index];
              return RadioListTile(
                value: index,
                groupValue: selectedAnimal,
                onChanged: (v) {
                  setState(() {
                    selectedAnimal = v!;
                  });
                },
                title: Text(e.animal),
                subtitle:
                    Text('${e.emoji} This is a ${e.animal.toLowerCase()}'),
                secondary: Icon(e.icon),
              );
            },
          ),
        ),
      ),
    );
  }
}

class Animal {
  String animal;
  String emoji;
  IconData icon;
  bool checked;

  Animal(
      {required this.animal,
      required this.emoji,
      required this.icon,
      required this.checked});
}
```

![](/images/listtile-knowledge/image4.png =300x)

* ラジオボタンは日本語、英語のように左から右に読む言語では、デフォルトで左に表示されます。これは`controlAffinity`を使って変更することができます。
* 他のListTileと違うのは、`groupValue`という引数があり、これによって、排他的な選択を実現しています。

## SwitchListTile
https://youtu.be/0igIjvtEWNU

* スイッチボタン（Androidではトグルスイッチ）つきのListTileです。

```dart
class MainApp extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => _MainApp();
}

class _MainApp extends State<MainApp> {
  final animals = [
    Animal(animal: 'Dog', emoji: '🐶', icon: Icons.pets, checked: false),
    Animal(animal: 'Cat', emoji: '🐱', icon: Icons.abc, checked: false),
    Animal(animal: 'Bird', emoji: '🐤', icon: Icons.ac_unit, checked: false)
  ];
  int selectedAnimal = 0;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: ListView.builder(
            itemCount: animals.length,
            itemBuilder: (context, index) {
              final e = animals[index];
              return SwitchListTile(
                value: e.checked,
                onChanged: (v) {
                  setState(() {
                    e.checked = v;
                  });
                },
                title: Text(e.animal),
                subtitle:
                    Text('${e.emoji} This is a ${e.animal.toLowerCase()}'),
                secondary: Icon(e.icon),
              );
            },
          ),
        ),
      ),
    );
  }
}

class Animal {
  String animal;
  String emoji;
  IconData icon;
  bool checked;

  Animal(
      {required this.animal,
      required this.emoji,
      required this.icon,
      required this.checked});
}
```

![](/images/listtile-knowledge/image5.png =300x)

* スイッチボタンは日本語、英語のように左から右に読む言語では、デフォルトで右に表示されます。

##　おわりに
これまで見てきたListTileの種類をまとめると、次のようになります。

| ListTile | CheckBoxListTile | RadioListTile | SwitchListTile |
| ---- | ---- | ---- | ---- |
| ![](/images/listtile-knowledge/image1.png =150x)| ![](/images/listtile-knowledge/image3.png =150x)|![](/images/listtile-knowledge/image4.png =150x)|![](/images/listtile-knowledge/image5.png =150x)|


FlutterではListTileのウィジェットが様々あり、シンプルなリストを作成するのがネイティブより簡単な場合があります。
それぞれの使い方に応じて適切なListTileを選択し、場合によってはオリジナルのウィジェットでリストを表現することができると良さそうです。

##　参考にさせていただいた記事とか
https://api.flutter.dev/flutter/material/ListTile-class.html
