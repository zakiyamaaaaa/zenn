---
title: "Dartにおけるパターンを理解する"
emoji: "📘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['dart', 'flutter']
published: true
---

# はじめに
この記事ではDart 3.0から導入されたパターン（Patterns）について、[公式のページ](https://dart.dev/language/patterns)をベースにパターンの中身について説明します。

# パターンとは

Dartにおけるパターンは、ステートメントや式と同様、Dart 言語の構文カテゴリです。パターンは、実際の値と照合できる一連の値の形状を表します。
わかりやすくいうと、ある型を持ってるか、形があるか、特定の値であるかなどを照合できる方法です。Dart3.0から使えるようになっています。

パターンの主な機能としては次の２つになります。
* データの照合
* データをより小さな変数に分割

１つ目のデータの照合は、先程の概要で説明した内容と重複するため、ここでは２つ目の機能について具体的に説明します。
この機能は、英語ではいわゆるDestructureと言われるもので、パターンが一致すると、オブジェクトのデータにアクセスし、それを部分的に抽出することができます。言い換えると、パターンはオブジェクトを分割し、再構造ができます。

```dart
final numList = [1,2,3];
final [a,b,c] = numList;
```
[a,b,c]がnumListのパターンとマッチしているので、`a = 1,b = 2,c = 3`というふうにそれぞれ割り当てられます。

構造化パターンは任意の種類のパターンをネストできます。

```dart
switch (list) {
  case [‘a’ || ‘b’, var c]:
	print(c);
}
```
この例でのパターンは、最初の要素が 'a' または 'b' である2要素のリストに一致し、構造を分解します。

## パターンの種類

| Pattern Type | Example |
| ---- | ---- |
| Const | Null, true, false, const 1, ‘abc’ |
| List | [‘a’ \|\| ‘b’, var c] |
| Map | {‘key’: subpattern} |
| Object | MyClass(…) |
| Wildcard | _ |
| Records | (subpattern1, subpattern2) |

:::message
RecordsはDart 3.10から導入された型で、他の言語ではタプル型に相当します。Typeの違うpatternを入れることができます。

```dart
final records = (111, ‘aaa’);
```
:::

## パターンの使う場面
Dart言語では、さまざまなところでパターンを使うことができます
* ローカル変数の宣言や割当
* ifやswitchケース
* forループ
* コレクションリテラル

### ローカル変数の宣言・割当
パターン変数宣言は、Dartでローカル変数宣言可能な場所であればどこでも使用できます。
パターンは宣言の右側の値と一致します。一致すると、値が分解され、新しいローカル変数にバインドされます。

RecordsとListでそれぞれの宣言をやってみます

```dart
// Records example
final (a,b,c) = (1,’a’,true);
print(a) // 1
print(b) // a
print(c) // true

// List example
final [a,b,c] = [1,’a’,true];
print(a) // 1
print(b) // a
print(c) // true
```

どちらも同じ結果となりました。
ただ、List ≠ Recordなため、`final (a,b,c) = [1,’a’, true]`はエラーとなります

ここで、変数宣言するときの修飾子ではvar, finalが使用可能で、constは使えません

```dart
const (a,b,c) = (1,’a’,true); // ERROR
```

変数の割当を次の例に示します。まず、マッチしたオブジェクトを分割します。次に、新しい変数をバインドするのではなく、既存の変数に値を代入します。これによって、ダミー変数のように余分な変数を使う必要がありません

```dart
final (a,b) = ('left', 'right');
(b,a) = (a,b) //swap
```

### Switch構文・Switch式
すべてのcase節にはパターンが含まれます。
Switch構文

```dart
switch (obj) {
case “one”:
	return 1;
case “two”:
	return 2
}
```

switch式はこれに似ているが、見た目が少し異なり、集合的に値を生成します

```dart
final value = switch (data) {
	PATTERN01 => // code
	PATTERN02 => // other code,
	_ => // last resort code
}
```

switchやif-case文に使うことができ、caseにはどのようなパターンにも使うことができます。
パターンがマッチした場合に、分割代入された値はローカル変数となります。その範囲はケースの本文内のみになります。

```dart
switch (obj) {
	case PATTERN_1:
// code
	case PATTERN_2:
// other code
}


if (variable case PATTERN) {
	//code
}
```

また、論理和なども使うこともできます。

```dart
var yourColor = switch (color) {
	Color.red || Color.blue || Color.green => true,
	_ => false
}
```

次のコードは同じことを行っており、色の値が原色であるかどうかを判断します。

```dart
// switch expression
bool isPrimary = switch (color) {
	Color.red || Color.yellow || Color.blue => true,
	_ => false
}

// switch statement
switch (color) {
	case Color.red || Color.yellow || Color.blue:
		isPrimary = true;
	default:
		isPrimary = false;
}

// If-case
if (color case Color.red || Color.yellow || Color.blue) {
	isPrimary = true;
		} else {
	isPrimary = false;
	}
```

switchケースでガード句を使う場合は、キーワード`when`を仕様します。。ケースの条件をさらに追加し、評価することができます。

```dart
switch (pair) {
	case (int a, int b):
	if (a > b) print(“a is greater than b”)
	// if false, exists the switch
	case (int a, int b) when a > b:
		print(‘a is greater than b);
}
```

### for ループ
forループとfor-inループでパターンを使用すると、コレクション内の値を反復処理したり再構造化ができます。

```dart
Map<String, String> hist = {
	‘ja’: ‘yen’,
	‘us’: ‘dollar’,
	‘zh’: ‘yuan’
}
for (var MapEntry(key: key, value:count) in hist.entries) {
	// code
}
```

このオブジェクトパターンは、`hist.entries`が`MapEntry`という名前付き型を持っていることをチェックし、名前付きフィールドのサブパターンkeyとvalueに再帰します。各反復で MapEntry の key ゲッターと value ゲッターを呼び出し、結果をそれぞれローカル変数 key と count にバインドします。

ゲッター呼び出しの結果を同じ名前の変数とすることはよくあることで冗長です。
そのため、オブジェクトパターンは変数サブパターんからゲッター名を推測することができます。

```dart
for (var MapEntry(:key, value:count) in hist.entries) {
	// code
}
```

## パターンのユースケース
複数のレコードの値を返す場合は、次のような個別に新しいローカル変数を作るより、複数のレコードのフィールドをまとめて作るほうが良いでしょう。

```dart
final info = userInfo(json);
final name = info.$1;
final age = info.$2;

final (name, age) = userInfo(json);
```

### JSONのバリデーション
パターンを使わない次のようなコードになります

```dart
if (json is Map<String, Object?> &&
    json.length == 1 &&
    json.containsKey('user')) {
  var user = json['user'];
  if (user is List<Object> &&
      user.length == 2 &&
      user[0] is String &&
      user[1] is int) {
    var name = user[0] as String;
    var age = user[1] as int;
    print('User $name is $age years old.');
  }
}
```

結構たいへんです。しかし、if-caseを使用するともっと簡単になります。単一のケースは`if-case文`として最適に機能します。
ここでのパターンの使用はjsonを検証するためのより宣言的で冗長性の低い方法を実現します。

```dart
if (json case {'user': [String name, int age]}) {
  print('User $name is $age years old.');
}
```

この少ないコードで、次のことを立証しています

* case節でMapパターンを照合し、jsonはMapであることが確かめられる。また、nullでもない。
* jsonはキー・ユーザーを含んでいる。
* キー・ユーザーと2つの値のリストが対になっている。
* リストの値の型はStringとintである。
* 値を保持する新しいローカル変数は、nameとageです。

# 終わりに
パターンは開発していると使わないことはないものですが、あまり中身について知らないこともあったので、改めて理解することでDartでの多様な表現が可能になると思います。


# 参考にさせていただいた記事とか
https://www.youtube.com/watch?v=aLvlqD4QS7Y

https://dart.dev/language/patterns