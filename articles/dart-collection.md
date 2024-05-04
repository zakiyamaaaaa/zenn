---
title: ""
emoji: "📑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

## forEach
void forEach(
void f(
E element
)
)

https://api.dart.dev/stable/3.3.3/dart-collection/ListQueue/forEach.html

リストの各要素に対して操作を行う。返り値はVoidなので、元の要素自体は変更されない

## map

Iterable<T> map<T>(
T toElement(
E element
)
)

https://api.dart.dev/stable/3.3.3/dart-collection/ListQueue/map.html

新しい要素を返す

```dart
// map
  final numbers2 = numbers.map((e) => e * 2).toList();
  print(numbers2);
```

## contains()
bool contains(
Object? element
)

https://api.dart.dev/stable/3.3.3/dart-collection/ListQueue/contains.html

あたら得られた要素があるかをboolで返す

## sort

## reduce

## fold
Generics型で返却値指定できる

ちょっとイメージ難しい

```dart
final numbers = <double>[10, 2, 5, 0.5];
const initialValue = 100.0;
final result = numbers.fold<double>(
    initialValue, (previousValue, element) => previousValue + element);
print(result); 
```