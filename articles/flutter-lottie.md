---
title: "【Flutter】FigmaでLottie作成→Flutter実装までの一連の流れ"
emoji: "⭐️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, lottie, figma]
published: false
---

# Lottie概要
Flutterを用いてなめらかなアニメーションを取り入れたい場合に有力な選択肢となるのが[Lottie](https://airbnb.design/lottie/)です。LottieはAirbnbが開発しているライブラリで、BodymovinというAdobeの拡張機能を利用したJSONを出力します。この出力されたJSONはモバイル、Webで使用することができます。JSONの拡張子ですが、中身がテキストファイルとなっているため、動画やアニメーション画像と比較すると軽量であることが特徴です。[公式ページ](https://airbnb.io/lottie/#/)によると、GIFの約2分の1サイズになるそうです。レンダリング時にSVGとして描画ができるため、画質の劣化も発生しません。
このようにFlutterはもちろんのこと、Web・モバイルともにアニメーションを取り入れる際にはLottieは様々なメリットを有します。
また、[こちらの記事を読むと](https://ics.media/entry/230913/)「.lottie」というLottieの拡張子の開発が進んでいるそうで、もしかすると今後はこちらのほうが主流になっていくのかもしれません。

今回はFigmaでLottieのファイルを作成、FlutterでLottieの実装について説明していきます

# アニメーションファイルの作成
それではまずアニメーションファイルを作成します。今回は無料でも使えるデザインツールのFigmaを利用していきます。
基本的な操作はデスクトップ用のFigmaアプリなので、事前にアプリをダウンロードしてください。（一応ブラウザでもできます）

https://www.figma.com/ja/downloads/

## LottieFilesプラグインの準備
メニューの「プラグイン」→「プラグインを管理」から「LottieFiles」プラグインを検索してインストールします。
このプラグインを使用するにはLottieFilesのアカウント登録をする必要があります。（無料で出来ます）

https://lottiefiles.com/

## 使用する画像の準備
次にアニメーションで動かす画像を準備します。Lottieはベクター画像を使用したアニメーションなので、使用する画像もベクターとなります。ベクターならなんでもいいのですが、今回は[Iconify](https://www.figma.com/community/plugin/735098390272716381/iconify)というベクターのアイコンが使える便利なFigmaのプラグインを使用します。

https://www.figma.com/community/plugin/735098390272716381/iconify

先ほどと同様にプラグインを管理から、「Iconify」を検索してインストールします。

インストールできたら、メニューから「プラグイン」→「Iconify」を選択します。

![](/images/flutter-lottie/image1.png)

この中から適当な言葉で使いたい画像を検索して選択します。
色や大きさも変更できたりします。ただ、ベクター画像なため大きさは後からでも変更可能なのであまり気にしなくてもいいです。

![](/images/flutter-lottie/image2.png)

## アニメーションの作成
それではアニメーションを作成していきます。
Figmaアプリのメニューにある「井」みたいなアイコンを選択し、フレームを選択します。
このフレームごとにアニメーションが変化するイメージです。
先ほどの画像をこのフレームに追加します。左のレイヤーのところから、フレームの中に含めてください。（フレームが1グループのようなイメージ）

下のような形でで画像を拡大・縮小させたり、別の画像を追加させたフレームを複数枚作成しました。

![](/images/flutter-lottie/image3.png)

一番最初のアニメーションとなるフレームを選択し、右にあるプロトタイプからフローの開始点、インタラクションを設定します。
インタラクションは「アフターディレイ」として、「次に移動」でその次のコマとなるフレームを選択します。
このアフターディレイは時間設定もできるので、ここでアニメーションの時間を変更することができます。一応あとでアニメーションの動きを確認できるので、それを見てから時間を設定しましょう。
同じような感じでフレームをつなぎ合わせます。（矢印が表示されると思います）
最後のフレームまでできたら、一番最初のフレームまでつなぎ、ループする一連のアニメーションを作成できます。

インタラクションを作成したら出てくる矢印を選択し、「即時」となっているところを「スマートアニメート」としましょう。選択すると、さらに下にアニメーションの方法（イーズアウト）が表示されるので、適したアニメーション方法を選択しましょう

![](/images/flutter-lottie/image7.png)

プロトタイプが完成したらLottieFilesプラグインからアニメーションを書き出します。
プラグインでLottieFilesを選択して、Lottieアカウントでログインします。

![](/images/flutter-lottie/image4.png)

Select prototype flowから該当のアニメーションフローを選択します。
するとこんな感じでアニメーション結果が出力されます。

![](/images/flutter-lottie/lottie.gif)

これでOKならアニメーションを保存します。保存先はLottieFilesのクラウドに保存されます。
ここからLottieFilesのdashboardを見てみると、先程アップロードしたアニメーションがあるかと思います。

そうすると、右側にダウンロードの項目があるので、ここからLottieJSONをダウンロードしましょう。

![](/images/flutter-lottie/image5.png)

これで、準備するアニメーションファイルが完成しました。
ここまでできれば9割ほど完成しています。

# Flutterでの実装
## 使用するパッケージ
lottieというパッケージを使用します。

https://pub.dev/packages/lottie

## 実装
`pubspec.yaml`でアセットを使えるようにします

```yaml:pubspec.yaml
    flutter:
        uses-material-design: true
        assets:
            - assets/sample.json
```

Lottieファイルの再生だけならとても簡単に実行できます。

```dart:main.dart
    Lottie.asset(
        'assets/sample.json',
        width: 100,
        height: 100,
    )
```
![](/images/flutter-lottie/simulator.gif)

こんな感じで簡単にLottieで生成したJSONファイルを再生することができました。

## おまけ
あまりアニメーションの知識がなくても無料で簡単に作れるのが良いのですが、それでもイケてるアニメーションを作るのは手間がかかります。
そんな人でも、LottieFiles上に公開されているアニメーションを使えるので、手軽にイケてるアニメーションを自分のアプリに組み込むことができます。

https://lottiefiles.com/featured

すべてのアセットが有料ではないのですが、無料のアセットでも十分なクオリティです。
こちらを使用するには、さきほどと同じようにクラウド上のワークスペースに保存し、JSONファイルをダウンロードして使う流れです。

こんなものや
![](/images/flutter-lottie/sample1.gif)

こんなものまで
![](/images/flutter-lottie/sample2.gif)

# 参考にさせていただいた記事

https://ics.media/entry/230913/

