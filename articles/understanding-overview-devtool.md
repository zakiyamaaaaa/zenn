---
title: ""
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

公式より：DevTools は、Dart および Flutter 用のパフォーマンスおよびデバッグ ツール一式です。 Flutter DevTools と Dart DevTools は、同じツール セットを指します。

DevToolsでは次のことが可能です
* FlutterアプリのUIレイアウトと状態を検査
* FlutterアプリのUIジャンクパフォーマンスの問題を診断する
* FlutterまたはDartアプリのCPUプロファイリング
* Flutterアプリのネットワークプロファイリング
* FlutterまたはDartアプリのソースレベルデバッグ
* FlutterまたはDartコマンドラインアプリのメモリ問題をデバッグする
* 実行中のFlutterまたはDartコマンドラインアプリの一般的なログと診断情報を表示する
* コードとアプリのサイズを分析する
* Androidアプリでディープリンクを検証する。

devToolsはVS Code,Android Studio, command lineで動作します。

## VS CodeでのDevToolsの利用
ここでは、VS CodeでのDevToolsの利用について説明していきます。

VS CodeでDevToolsを利用する場合は、Dartの拡張が必要です。ただ、Flutterアプリケーションをデバッグする場合には、すでにFlutter拡張が入ってるので、インストールする必要はありません。

### デバッグを開始する
まず、アプリケーションをデバッグモードで起動します。

デバッグセッションがActiveになり、アプリケーションが起動したら、Open DevToolsのコマンドが可能になります。ショートカットコマンドは`⌘`+`Shift`+`P`です。
dart.embedDevTools 設定を使用して、DevTools を常にブラウザで開くように選択でき、dart.devToolsLocation 設定を使用して、DevTools をフル ウィンドウとして開くか、現在のエディタの隣の新しい列で開くかを制御できます。

```json
 {"dart.devToolsLocation": "beside"}
```
`beside`: アクティブなエディターの別画面でDevToolsを開きます
`active`: アクティブなエディターの同画面、別タブでDevToolsを開きます
`external`:ブラウザのウィンドウでDevToolsを開きます

DevToolsが動いているかどうかは、ステータスバーの`language status area(the {} Dart)`のところで確認できます。

# 機能別DevToolsの紹介
それでは、DevToolsの中身について説明していきます。
VSCodeに搭載されているDevToolsは非常に豊富で９つに分類されます。

1. Flutter inspector
2. Performance view
3. CPU Profiler view
4. Memory view
5. Debug console view
6. Network view
7. Debugger
8. Logging view
9. App size tool

それでは順に説明していきます

## 1. Flutter inspector
一番使うDevtoolです。ウィジェットツリーの視覚化、UIパーツを探したり、レイアウトに問題があるときに調査するために使います。

こちらはDevToolsのメニュー以外にもメニューバーからも選択が可能です。
ここのメニューバーにDevToolsで唯一選択できるということで、一番使われるDevToolsということが予想されます。

TODO: メニューバーImage

Inspectorを開いてみると、左にウィジェットTree、右側にいろいろあります。そして、左上のほうに"Select Widget Mode"というのがあります。
個人的に一番Inspectorが使われる機会として多いのは、ここの画面のこのUIは、どこのウィジェットなのか、というときです。ファイル数が多くなると、ウィジェットのパーツを追うのが結構たいへんです。

このSelect Widget modeをクリックして有効した状態で、シミュレーターをタップすると、そのタップされたWidgetがWidget Tree状でどこのWidgetかを示してくれます。

TODO: 選択したときのImage

こんな感じでサイズやPaddingなんかを示してくれます。
また、並びやflexなども変更して画面上で確認できるので、デザインチェックにも使えます。このときの変更はコードまで変更されてないので、実際に変更する場合はコードも変更してください。

TODO: 変更項目のImage

右上にタブがあります。それぞれの説明をします。

### slow animations
このオプションを有効にすると、アニメーションが５倍遅くなります。見た目があまり正しくないアニメーションを注意深く観察し、微調整する場合に便利です。

TODO: アニメーションのgif

コードでも同様の環境を設定できます。

```dart
import 'package:flutter/scheduler.dart';

void setSlowAnimations() {
  timeDilation = 5.0;
}
```

### show guidelines
UIの様々なガイドラインをアプリに描画します。

TODO: Image

レイアウトの理解を深めるために使用できます。例えば不要なPaddingを見つけたり、ウィジェットの配置を理解したりできます。

コードでは次のよう

```dart
import 'package:flutter/rendering.dart';

void showLayoutGuidelines() {
  debugPaintSizeEnabled = true;
}
```

### show baselines
すべてのテキストのベースラインを表示します。

TODO: Image

これは、テキストが垂直に正確に並んでいるかどうかをチェックするのに便利です。例えば次のベースラインのテキストは微妙にずれています。
アルファベットのベースラインは緑色、表意文字は黄色で表示されます。

コードでは次のよう

```dart
import 'package:flutter/rendering.dart';

void showBaselines() {
  debugPaintBaselinesEnabled = true;
}
```

### Highlight repaints
このオプションはすべてのレンダーボックスの周囲に境界線を描画し、ボックスが再描画されるたびに色が変わります。

TODO: Image

たとえば１つの小さなアニメーションにより、フレームごとにページ全体が再描画される可能性があります。アニメーションをRepaintBoundaryウィジェットでラップすると、再描画がその内部のみに制限されるため、パフォーマンスの向上が期待されます。

公式から引用しますが、次のようなindicatorがあったとします。

```dart
class EverythingRepaintsPage extends StatelessWidget {
  const EverythingRepaintsPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Repaint Example')),
      body: const Center(
        child: CircularProgressIndicator(),
      ),
    );
  }
}
```

これをツールで見た場合、次のようなUIになります。

TODO: gif

これをRepaintBoundaryで囲んでみます。

```dart
class AreaRepaintsPage extends StatelessWidget {
  const AreaRepaintsPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Repaint Example')),
      body: const Center(
        child: RepaintBoundary(
          child: CircularProgressIndicator(),
        ),
      ),
    );
  }
}
```

TODO: gif

このように、無駄なBuild範囲を確認するのに良さそうです。

こちらもコードでは次のようです。

```dart
import 'package:flutter/rendering.dart';

void highlightRepaints() {
  debugRepaintRainbowEnabled = true;
}
```

### Highlight oversized images
データサイズが大きすぎる画像を色の反転、垂直方向の反転をさせて、強調して表示させます。

TODO: image

異常にデータサイズが大きい画像は、特にローエンドデバイスでパフォーマンスの低下を引き起こす可能性があります。こうした画像をリストビューのように多数の画像を使う場合は、パフォーマンスに大きく影響を与えます。

こうした画像を修正する方法は、まずはもとの画像ファイルを変更してサイズを小さくすることです。
これが難しい場合は、ImageコンストラクターでcacheHeightとcacheWidthパラメーターを使用することです。

```dart
class ResizedImage extends StatelessWidget {
  const ResizedImage({super.key});

  @override
  Widget build(BuildContext context) {
    return Image.asset(
      'dash.png',
      cacheHeight: 213,
      cacheWidth: 392,
    );
  }
}
```

これにより、エンジンはこの画像を指定されたサイズでデコードするようになり、メモリ使用量が削減されます。画像は、これらのパラメータに関係なく、レイアウトまたは幅と高さの制約に合わせてレンダリングされます。

このデバッグオプションをコードで有効にする場合は次の形です。

```dart
void showOversizedImages() {
  debugInvertOversizedImages = true;
}

```


# 参考にした記事とか
