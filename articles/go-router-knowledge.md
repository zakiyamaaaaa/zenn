---
title: "go_routerの各遷移方法について学ぶ"
emoji: "✏️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, go_router, dart]
published: true
---

# はじめに
モバイル開発において、重要な要素である画面遷移ですが、Flutterにおける遷移の方法は大きく２つに分かれます。
それぞれをNavigator1.0,Navigator2.0とすると
* **Navigator1.0**：命令的な方法。Navigator.push()やNavigator.pop()のように、画面をスタックして扱う
* **Navigator2.0**：宣言的な方法。アプリの状態を元に画面遷移を定義し、ナビゲーションの制御を行う。

Navigator2.0の主な特徴はこちらになります。
（本記事の本題ではないので、知っている方は読み飛ばしてください）
:::details Navigator2.0の主な特徴
1. 宣言的ナビゲーション:

Navigator 2.0は、アプリの状態に基づいてナビゲーションを定義できるので、状態管理の一部としてナビゲーションを取り扱えます。
画面のスタックや遷移履歴を明示的に管理でき、ユーザーの操作やアプリの状態に応じたナビゲーションを実現できます。

2. RouterとRouteInformationParser:

Routerは、アプリのナビゲーションのロジックを処理します。これによって、画面遷移やルートの管理を行います。
RouteInformationParserは、URL（または他の形式の入力）から適切な画面を決定するためのものです。これにより、ウェブアプリにおけるURLベースのナビゲーションを簡単に実現できます。

3. 状態管理とURL同期:

URL同期機能を利用して、アプリの状態（どの画面にいるか）をURLに反映させることができます。これにより、ブラウザの「戻る」「進む」ボタンを使ったナビゲーションや、ページを再読み込みしても正しい状態を保つことができます。
この機能は特にPWA（Progressive Web App）やウェブ対応アプリケーションにおいて強力です。

4. PageとRouteの概念:

Navigator 1.0ではRouteが単一の画面を表していましたが、Navigator 2.0では、Pageという新しい概念を導入しています。Pageは、UIの状態を表すものとして、より柔軟に遷移管理を行うことができます。
各ページ（Page）はRouteの集合体として機能し、UIの状態（例えば、データや遷移のコンテキスト）を簡単に扱えるようになっています。
:::

Navigator2.0によって、画面遷移の幅は広がったのですが、Navigator2.0をラップしたライブラリを用いることが多いです。
2025年1月現在では、主に`go_router`, `auto_route`という２つのライブラリが広く使われている印象です。
私が仕事で携わるプロジェクトでは`go_router`を使っていることから、本記事では、`go_router`の遷移API(go,push,replaceなど)の違いについて説明します。

# go_router
https://pub.dev/packages/go_router

## go_routerの概要
公式パッケージのページから、概要を引用します。
> RouterAPIを使って、異なる画面間をナビゲートするための便利なURLベースのAPIを提供するFlutter用の宣言型ルーティングパッケージ。 URLパターンを定義したり、URLを使ってナビゲートしたり、ディープリンクを扱ったり、その他多くのナビゲーション関連のシナリオを扱うことができます。

機能としては次の通りです。（公式から引用）
> ・テンプレート構文を使用したパスとクエリパラメータの解析（例えば、"user/:id')
> ・1つの目的地に対して複数の画面を表示（サブルート）
> ・リダイレクトのサポート - アプリケーションの状態に応じて、ユーザーを別のURLにリルーティングすることができます。
> ・ShellRoute による複数のナビゲーターのサポート - 一致したルートに基づいて独自のページを表示する内部ナビゲーターを表示できます。たとえば、画面の下部にBottomNavigationBar
> ・MaterialとCupertinoアプリの両方をサポート
> ・Navigator API との下位互換性

go_routerのセットアップについては、こちらの記事などを参考にしてください。
https://zenn.dev/channel/articles/af4ffd813b1424

## go_routerの遷移API
次の遷移APIが主にあります。
* go
* push
* replace
* pushReplacement
* pop（今回の記事では説明を省略）

今回はpopについての説明は省きます。popはスタックの一番上の画面を消して、次の画面を取り出します。つまり、前の画面に戻るような動きになります。
ちなみに、goNameというように、Nameがついている遷移APIもあり、これはrouterに指定された名前を使用する方法であり、ディレクトリを指定する方法とほぼ同じになります。
各APIについて説明する前に、画面のスタック状態についての確認方法について説明します。

**Navigator1.0**
Navigator1.0の場合はA→Bへpushで遷移した場合、Aがスタックされているか見ることができます。

```dart
Navigator.of(context).widget.pages
```

**Navigator2.0**
Navigator2.0の場合はA→Bへpushで遷移した場合、A・Bがスタックされているか見ることができます。また、調べたところでは前述の方法とこちらのgo_routerのrouterDelegateで確認できました。

```dart
goRouter.routerDelegate.currentConfiguration.matches
```

また、画面の構成ですが、FirstPage、SecondPage、（ThirdPage）という画面を作り、FirstPage→SecondPage（→ThirdPage）の遷移となり、SecondPageやThirdPageにはpopするボタンがあるという想定です。
実際動かしたい方はGitHubのほうを御覧ください。

https://github.com/zakiyamaaaaa/go-router-sandbox

この前提をもとに、各遷移APIを実行したときのスタック状態を見ていきます。

## go
```dart
context.go('/second');
```

遷移先の画面で、画面のスタックを確認します。
```bash
goRouter.routerDelegate.currentConfiguration.matches

List (1 item)
```
このListの中身は/secondに指定された`SecondPage`でした。
goの実装を見ていくと、このように書かれていました。

```dart
/// Replace the current route matches with the `location`.
  void go(String location, {Object? extra}) {
    _setValue(
      location,
      RouteInformationState<void>(
        extra: extra,
        type: NavigatingType.go,
      ),
    );
  }
```
つまり、locationにマッチするルートへ現在のルートを置き換える、というもので、スタックに積まれません。そのため、遷移先画面からpopすることができません。

## push
```dart
context.push('/second');
```

```bash
goRouter.routerDelegate.currentConfiguration.matches
List (2 items)
```

ListにはFirstPageとSecondPageが積まれていました。
pushの実装を見ていきます。

```dart
/// Pushes the `location` as a new route on top of `base`.
  Future<T?> push<T>(String location,
      {required RouteMatchList base, Object? extra}) {
    final Completer<T?> completer = Completer<T?>();
    _setValue(
      location,
      RouteInformationState<T>(
        extra: extra,
        baseRouteMatchList: base,
        completer: completer,
        type: NavigatingType.push,
      ),
    );
    return completer.future;
  }
```

既存のルートのトップにlocationをpushする、ということがわかります。
この場合、もちろん`context.pop()`で画面を戻る事もできますし、`Navigator.pop(context)`も使うことができます。

## replace
```dart
context.push('/second');
```

```bash
goRouter.routerDelegate.currentConfiguration.matches

List (1 item)
```
replaceは遷移元のページがスタックされてません。
実装を見ていきます。

```dart
/// Replaces the top-most route match from `base` with the `location`.
  Future<T?> replace<T>(String location,
      {required RouteMatchList base, Object? extra}) {
    final Completer<T?> completer = Completer<T?>();
    _setValue(
      location,
      RouteInformationState<T>(
        extra: extra,
        baseRouteMatchList: base,
        completer: completer,
        type: NavigatingType.replace,
      ),
    );
    return completer.future;
  }
```
コメントを読む限り、スタックの一番上と入れ替える、とあります。
どういうことなのか、もう少し実験していきます。
FirstPage→SecondPage→ThirdPageというふうに遷移するページを想定します。
FirstPageからSecondPageはpush、SecondPageからThirdPageへはreplaceを使ってみます。

ThirdPageで画面スタックを確認すると、FirstPageとThirdPageが積まれていました。
そして、ThirdPageからpopした場合はSecondPageではなく、FirstPageに戻ります。
また、SecondPageからThirdPageへgoで遷移した場合は、それまでの遷移スタックがリセットされ、ThirdPageだけスタックに積まれる状態になります。

## pushReplacement
```dart
context.push('/second');
```

```bash
goRouter.routerDelegate.currentConfiguration.matches

List (1 item)
```
こちらもreplaceと同様に遷移元のページ（FirstPage）がスタックされてません。

実装を見てみます。

```dart
/// Removes the top-most route match from `base` and pushes the `location` as a
  /// new route on top.
  Future<T?> pushReplacement<T>(String location,
      {required RouteMatchList base, Object? extra}) {
    final Completer<T?> completer = Completer<T?>();
    _setValue(
      location,
      RouteInformationState<T>(
        extra: extra,
        baseRouteMatchList: base,
        completer: completer,
        type: NavigatingType.pushReplacement,
      ),
    );
    return completer.future;
  }
```

読む限りはreplaceと似た印象です。
先ほどと同様にFirstPage→SecondPage→ThirdPageというふうに遷移するページを想定していきます。
FirstPageからSecondPageはpush、SecondPageからThirdPageへはpushReplacementを使ってみます。

ThirdPageでスタックを確認すると、replaceと同様となっています。しかし、動きを見ると、SecondPageからThirdPageへ遷移アニメーションがあります。

replaceのコードについて、次のようなコメントがありました。

```dart
/// Replaces the top-most page of the page stack with the given one but treats
  /// it as the same page.
  ///
  /// The page key will be reused. This will preserve the state and not run any page animation.
```
つまり、replaceの場合は状態は変わらず、アニメーションがない形で遷移するということでした。これがreplaceとpushReplacementの大きな違いになります。

# おわりに
いかがでしたでしょうか。
Flutterアプリ開発において、遷移方法特にNavigator2.0への理解はとても重要だと考えます。
go_routerを使うにはある程度慣れも必要だと思いますが、これらの基本的な遷移の理解をしていることで、自信を持って遷移処理を実装できると思います。
個人的には、`go_router_builder`と一緒に使うのがおすすめで、go_routerの導入検討している方は調べてみてください。
お読みいただきありがとうございました。

気軽にフォローしてください。
https://x.com/yamazaking0

# 参考にした記事とか
https://pub.dev/packages/go_router

https://zenn.dev/channel/articles/af4ffd813b1424
