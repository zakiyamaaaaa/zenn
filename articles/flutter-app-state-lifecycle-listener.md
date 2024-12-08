---
title: "【Flutter】AppLifecycleStateとAppLifecycleListenerの挙動の違いについて"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, iOS, Android]
published: false
---

# はじめに
本投稿は[Flutter Advent Calendar 2024](https://qiita.com/advent-calendar/2024/flutter)の10日目の記事です🎄

Flutterにおいて、アプリの状態変化（フォアグラウンド・バックグラウンド・再開など）を検知する機能は、以前から `WidgetsBindingObserver` を利用し `AppLifecycleState` を確認することで実現していました。しかし、Flutter 3.13 からは、より明確なライフサイクルイベントを取得するための仕組みとして **AppLifecycleListener** が追加されました。

本記事では、Flutter 3.13 以前からある `AppLifecycleState` の基本的な理解を整理した上で、`AppLifecycleListener` とそれに伴う新たなイベントについて解説します。

:::message
なお、本文章についてはChatGPT-o1を用いて、文章の校正を行っておりますが、最終的な文章・動作のチェックは執筆者本人が行っていますことを了承ください。
:::

## AppLifeｃｙｃleStateとは
Flutterでは、アプリがフォアグラウンドで動作しているか、バックグラウンドに移行したかなどを検知するために、`WidgetsBindingObserver` を用いて `AppLifecycleState` を受け取ることができます。

### AppLifeCycleStateの種類
AppLifecycleStateは列挙型となり、次の列挙子（値）を取ります。
* paused
* hidden
* inactive
* resumed
* detached
この中で、`hidden`はFlutter3.13.0から追加された新しい列挙子で、macOSやlinuxのために追加されました。（詳細：https://docs.flutter.dev/release/breaking-changes/add-applifecyclestate-hidden）
それぞれについて、公式のページから説明を抜粋します。

#### paused
* アプリケーションがユーザーに現在表示されておらず、ユーザー入力にも応答しない状態です。
* この状態のとき、エンジンは [PlatformDispatcher.onBeginFrame] や[PlatformDispatcher.onDrawFrame] コールバックを呼び出しません。
* この状態には iOS と Android でのみ遷移します。

#### hidden
* アプリケーションのすべてのビューが非表示になっている状態です。これは、アプリが一時停止されようとしている場合（iOS や Android）、最小化された、または表示されていないデスクトップ（非Webデスクトップ）、もしくは非表示になっているウィンドウまたはタブ（Web）で実行中であることを意味します。
* iOS と Android では、全てのプラットフォームで状態マシンを統一するため、
[inactive] から [paused] に遷移する際に、または [paused] から [inactive] に遷移する際に、この状態への遷移が合成されます。これにより、アプリが概念的に「隠れた」ことを通知するためのハンドラを1つだけ記述すれば、クロスプラットフォームな実装が可能になります。

#### inactive
* 少なくともアプリケーションの1つのビューは表示されていますが、いずれのビューも入力フォーカスを持っていない状態です。アプリケーションはそれ以外は通常どおり実行中です。
* 非Webデスクトッププラットフォームでは、これはアプリケーションがフォアグラウンドにいないが、依然として表示可能なウィンドウを持っている状態に相当します。
* Web上では、この状態はアプリケーションが実行中ではあるが、入力フォーカスを持たないウィンドウまたはタブで動作していることを示します。
* iOS と macOS においては、Flutter ホストビューがフォアグラウンドながら非アクティブな状態に対応します。アプリは電話中、TouchIDリクエスト中、アプリスイッチャーやコントロールセンター表示中、または Flutter アプリをホストしている UIViewController が遷移中などにこの状態になります。
* Android では、この状態は Flutter ホストビューが Android の paused 状態
( [`Activity.onPause`](https://developer.android.com/reference/android/app/Activity#onPause()) が呼ばれた状態 )、もしくは Android の「resumed」状態だがウィンドウフォーカスを持たない状態に対応します。たとえば、アプリが部分的に覆われていたり、別のアクティビティがフォーカスしている場合、スプリットスクリーンで現在選択されていないアプリ、電話の着信、ピクチャーインピクチャー、システムダイアログ、他のビューが重なっている場合などが該当します。また、通知バーが下がっている場合や、アプリケーションスイッチャーが表示されている場合も非アクティブになります。
* Android と iOS において、アプリがこの状態になると、いつ [hidden] や [paused] に遷移してもおかしくないため、
その可能性を考慮する必要があります。

#### resumed
* Android では、この状態は Flutter ホストビューがフォーカスを持ち、( [`Activity.onWindowFocusChanged`](https://developer.android.com/reference/android/app/Activity#onWindowFocusChanged(boolean)) が true で呼ばれた )Android の「resumed」状態にあることに対応します。
* iOS と macOS では、この状態はアプリがフォアグラウンドでアクティブな状態を示します。

#### detached
* アプリが初期化される前にデフォルトで、この状態になります。アプリが起動された場合に、`detached`から基本的に`resumed`に切り替わります。
* すべてのビューがデタッチされたあと（iOS/Android）にこの状態になります。
* この状態はiOS/Androidでのみ遷移しうるものですが、すべてのプラットフォームでアプリケーションが開始する前のデフォルトの状態でもあります。

### 実装例
AppLifecycleStateを用いたアプリの状態を取得するコード例を以下に示します。
こちらは現時点でのFlutterバージョンでも問題なく動きます。

```dart
import 'package:flutter/material.dart';

class MyAppState extends State<MyApp> with WidgetsBindingObserver {
  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance.addObserver(this);
  }

  @override
  void dispose() {
    WidgetsBinding.instance.removeObserver(this);
    super.dispose();
  }

  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    super.didChangeAppLifecycleState(state);
    switch (state) {
      case AppLifecycleState.resumed:
        print("App is in foreground (resumed)");
        break;
      case AppLifecycleState.paused:
        print("App is in background (paused)");
        break;
      case AppLifecycleState.inactive:
        print("App is inactive");
        break;
      case AppLifecycleState.detached:
        print("App is detached");
        break;
      // hiddenはiOSなど特定環境でのみ発火
      case AppLifecycleState.hidden:
        print("App is hidden (iOS)");
        break;
    }
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: const Text('AppLifecycleState App'),
        ),
        body: const Center(
          child: Text('Hello, World!'),
        ),
      ),
    );
  }
}
```

### 挙動
上記コードを動かした場合に、アプリをバックグラウンドにした場合、次のようなstateの遷移となります。

inactive→hidden→paused

またアプリをスワイプでkillした場合は、次のようになります。
inactive→hidden→paused→detached

次にアプリをバックグラウンドからフォアグラウンドにした場合の状態の遷移を見てみます。
hidden→inactive→resume

## Flutter 3.13から追加されたAppLifecycleListenerとは
Flutter 3.13 では、AppLifecycleListener という新たなクラスが導入され、アプリのライフサイクルイベントをより明確かつシンプルに扱えるようになりました。

AppLifecycleListener は WidgetsBindingObserver を代替・補完するもので、これによって各ライフサイクルイベントの明確なハンドリングが可能になります。

### AppLifecycleListenerのイベント
AppLifecycleListenerでは、コールバック関数から状態を受け取ることができます。
関数は次の７つがあります。
* onShow
* onResume
* onHide
* onInactive
* onPause
* onDetach
* onRestart

また、状態が変更された場合は、`onStateChange`が呼び出されます。このとき、AppLifecycleStateが渡されるので、前述のAppLifecycleStateでのハンドリングが可能となります。
ここでは、AppLifecycleListenerから追加された2つの状態（関数）について、公式のドキュメントを載せます。

#### onShow
* アプリケーションが表示された時に呼び出されるコールバックです。
* モバイルプラットフォームでは、これは通常、他のアプリケーションからこのアプリケーションが前面に来る直前に呼び出されます。
* デスクトッププラットフォームでは、最小化された状態から復帰したり、何らかの形でアプリケーションのビューが表示される直前に呼び出されます。
* Webでは、ウィンドウ（またはタブ）が表示される直前に呼び出されます。

#### onRestart
* アプリケーションが一時停止（paused）状態から再開（resumed）する際に呼び出されるコールバックです。
* モバイルプラットフォームでは、このアプリケーションが再びアクティブアプリケーションとして前面に出る直前に呼び出されます。
* デスクトッププラットフォームおよびWebでは、この関数は呼び出されません。

### 実装例
AppLifecycleListener は WidgetsBindingObserver と似た流れで使用できます。WidgetsBinding.instance に対して addObserver する代わりに、AppLifecycleListener を作成して listen メソッドを呼び出します。

```dart
import 'package:flutter/material.dart';

class AppLifecyclePage extends StatefulWidget {
  const AppLifecyclePage({super.key});

  @override
  State<AppLifecyclePage> createState() => _AppLifecyclePageState();
}

class _AppLifecyclePageState extends State<AppLifecyclePage> {
  late final AppLifecycleListener _listener;

  @override
  void initState() {
    super.initState();

    _listener = AppLifecycleListener(
      onShow: () => print('onShow'),
      onResume: () => print('onResume'),
      onHide: () => print('onHide'),
      onInactive: () => print('onInactive'),
      onPause: () => print('onPause'),
      onDetach: () => print('onDetach'),
      onRestart: () => print('onRestart'),
      onStateChange: _onStateChanged,
    );
  }

  @override
  void dispose() {
    _listener.dispose();

    super.dispose();
  }

  void _onStateChanged(AppLifecycleState state) {
    switch (state) {
      case AppLifecycleState.detached:
        _onDetached();
      case AppLifecycleState.resumed:
        _onResumed();
      case AppLifecycleState.inactive:
        _onInactive();
      case AppLifecycleState.hidden:
        _onHidden();
      case AppLifecycleState.paused:
        _onPaused();
    }
  }

  void _onDetached() => print('detached');

  void _onResumed() => print('resumed');

  void _onInactive() => print('inactive');

  void _onHidden() => print('hidden');

  void _onPaused() => print('paused');

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: const Text('AppLifecycleListener App'),
        ),
        body: const Center(
          child: Text('Hello, World!'),
        ),
      ),
    );
  }
}
```

### 挙動
上記コードでは、アプリの状態の変更があった場合は、状態に対する関数と、`onStateChange`の両方が呼び出されます。これによって、発火する状態イベントがどのように異なるのかが、確認しやすくなります。
一旦、ここでは状態に対する関数について、どのように発火するのかについて説明します。
AppLifecycleStateの場合と同様に、まずはフォアグラウンドからバックグランドにアプリが遷移した場合について見ていきます。

(foreground):onInactive→onHide→onPause

AppLifecycleStateの場合と同様です。アプリをkillした場合も、同じ状態遷移が発火します。

(foreground):onInactive→onHide→onPause→onDetach

次にアプリをバックグラウンドからフォアグラウンドへ起動した場合を見てみます。

(background):onRestart→onShow→onResume

今回追加されたonRestart,onShowという関数が発火しています。
AppLifecycleStateとの対応を見てみると、次のようになっていることがわかります。

`AppLifecycleState`
(background):hidden→inactive→resumed

`AppLifecycleListener`
(background):onRestart→onShow→onResume

AppLifecycleListenerのほうが、明確に再起動していることをキャッチしているように見えます。

また、挙動の図として一番わかりやすいのが公式に載っていたので貼っときます。
![](/images/app_state_lifecycle_flutter_3.13/image1.png)
（参考：[url](https://flutter.github.io/assets-for-api-docs/assets/dart-ui/app_lifecycle.png)）


## 終わりに
AppLifecycleStateとAppLifecycleListenerの違いを見てきました。
AppLifecycleListenerのほうが、WidgetBindingを使用せずに使えたり、検知できる状態の種類が多いため、こちらのほうが使い勝手が良さそうです。
ただ、いくつかの注意点があります。
* 各イベントはプラットフォーム（iOS/Android/Web）で挙動が異なる場合があります。開発中の環境で必ず確認してください。
* AppLifecycleState と AppLifecycleListener を併用する場合、似たようなイベントが複数回発火することがあります。必要に応じて整理するか、AppLifecycleListener 側に一本化することを検討してください。
そして、今回記事の作成中に気づいたのですが、Androidのemulatorでは、再起動での状態イベントがonResume(resumed)のみになってしまいました。慌てて実機で確認してみると、ちゃんとiOSと同じ状態イベントを取得できていたので、前述のように、プラットフォーム・環境ごとの違いを認識する必要があります。

アプリの状態通知はよく使われると思うのですが、Flutter3.13以前のAppLifecycleStateを用いた方法の記事が多かったため、こちらの記事が参考になれば幸いです。アプリの細かな状態イベントの発火については、AAkiraさんの[こちらの記事](https://speakerdeck.com/aakira/flutter-kaigi-2023)が参考になります。


## 参考資料
* https://api.flutter.dev/flutter/widgets/AppLifecycleListener-class.html

* https://api.flutter.dev/flutter/dart-ui/AppLifecycleState.html

* https://docs.flutter.dev/release/breaking-changes/add-applifecyclestate-hidden

* https://speakerdeck.com/aakira/flutter-kaigi-2023

* https://kazlauskas.dev/blog/flutter-app-lifecycle-listener-overview/

