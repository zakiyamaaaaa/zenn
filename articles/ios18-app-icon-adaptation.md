---
title: "iOS/iPadOSアイコンアップデートに備える"
emoji: "👌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [iOS,iPadOS]
published: false
---

# はじめに
（こちらの記事は[Sansanモバイル勉強会vol.1](https://sansan.connpass.com/event/321996/)の内容となります）

2024年のWWDCで発表されたiOS18では、アプリアイコンに大きな変更のアナウンスがありました。
これまでは開発元が提供するアプリアイコンのみ使用されていましたが、1OS18/iPadOS18からはユーザーの設定によって、アプリアイコンの外観カラーが変わるようになりました。

![](/images/ios18-app-icon-adaptation/image1.png)
WWDC session video

## これまでのアイコンとの違い
![](/images/ios18-app-icon-adaptation/image2.png)
Human Interface Guideline

左から`Light`、`Dark`, `Tinted`のテーマでのアプリアイコンとなります。
`Tinted`というのは、ユーザーが好きなように設定できるカラーになります。
こちらの参考画像のように、アイコン背景は`Dark`と同じ暗色になることがわかります。

アプリアイコンは基本的に濃い色や色合いのアイコンをデザインすることが推奨されています。
さまざまな色合いや背景でも、すぐれた視認性のアプリアイコンを作るように心がけましょう
また、アプリアイコンは一番ユーザーがアプリに触れる導線となるので、どのように見えるか、アクセシビリティテストが重要になってきます。

## 今回のアップデートに対応してみる
それでは実際に今回のアップデートにどのように対応していくのかをWWDCのセッションや公式ガイドラインをもとに説明していきます。

:::message
なお、現時点（2024年7月時点）ではベータ版をインストールする必要があるため、Sonoma 14.5~推奨です。
また、今回の検証ではXcode 16.0.0-Beta3を使用しています。

XcodeのインストールはXcodesがおすすめです。[Xcodes](https://www.xcodes.app/)
:::

### Xcodeでの設定方法
[公式ページ](https://developer.apple.com/documentation/xcode/configuring-your-app-icon#Overview)をもとにXcodeでの設定方法を説明します

`Assets->AppIcon`を選択し、`Attribute Inspector->Appearances->Any, Dark, Tinted`に変更しましょう。
そうすると、このようにアイコン設定する箇所が３種類の画面に切り替わります。

![](/images/ios18-app-icon-adaptation/image3.png)

### アイコンの作り方
ここからはどのようにアイコンを作ってくのかを説明していきます。

### Lightアイコン
![](/images/ios18-app-icon-adaptation/light.png)
これまでのアプリアイコンをここに設定します。基本的な形としては、背景とアプリアイコン、という構造で設定すると簡単です。

### ダークアイコンの作り方
![](/images/ios18-app-icon-adaptation/dark.png)
* LightでのアプリアイコンをDarkでのアイコンのベースとして使用します。
* 過度に明るい補色の利用は避ける。
* システムから背景色を設定するため、バックグランドは除外します

### Tintedアイコンの作り方
![](/images/ios18-app-icon-adaptation/tinted.png)
* グレースケール画像として作成
* ほとんどのアプリアイコンでは、アイコン画像全体に垂直方向のグラデーションを適用すると見栄えがよくなる


AppleのHuman Interface GuidelineでAppIconの項目がアップデートされているので、詳細についてはこちらをご参考ください。

https://developer.apple.com/design/human-interface-guidelines/app-icons

また、具体的なアプリアイコン例を見たいという方はFigma, Sketch, Photoshopでデザインテンプレートが提供されているので、こちらを参考にするとレイヤー構造など詳細がわかって理解が深まるかと思います。

https://developer.apple.com/design/resources/

### iPhoneでの確認方法
意外にわかりづらかったので、ここでは簡単にやり方を紹介します。

1. 画面を長押しした状態で、ホーム画面を編集を選択します。
2. 編集ボタンを押すと、カスタマイズという項目があります。そこでアプリのサイズやカラーを変更することができます。

:::message
iOS18を入れた状態のシミュレーター(or実機)で行ってください
:::

## 今回やってみての気付き
1. シンプルなアイコンであればそこまで対応に時間はかからない
AppleのHuman Interface Guidelineに則ってAppIconを作っているのであれば、シンプルなアイコン構成になっているはずです。
そのため、今回のアップデートにおいてもそこまで大きな工数はかからないかと思います。

2. いくつか注意点

* アイコン本体が暗色の場合は注意
   ブランド名がアプリアイコンとなっていて、黒い文字に白い背景みたいなアプリを見かけたことがあるかと思います。
今回のアップデートでこれをそのまま対応すると、Darkの場合、背景が黒となるので視認性がとても悪くなります。
![](/images/ios18-app-icon-adaptation/sample_light_2.png)
![](/images/ios18-app-icon-adaptation/sample_dark_2.png)
![](/images/ios18-app-icon-adaptation/sample_tinted_2.png)

ただ、OS18 Beta3では、自動的にアイコンカラーがいい感じにしてくれていて、Darkの場合はロゴの色を反転するなどしているため、見やすくなっています。
しかし、なるべくであればアプリ事業側でカラーはコントロールしたいかと思います。

* Widgetもカラー適用対象
今回のカラーテーマはアイコンだけではなく、Widgetもカラー変更対象でした。そのため、Widgetのデザインカラーも見直しが必要となるかもしれません。

3. アイコンのカラーモードは結構使われそう
HOME画面はユーザーが一番触れるところでもあり、アイコンのカラーテーマが変更されるという新機能をオシャレなものとして若い世代を中心に使われると感じました。テーマが一様に適用されるため、アップデート対応されていないと変に目立ってしまい、アプリとして重要性が低い場合は削除される可能性も。そのため、アプリアイコンのアップデートはなるべくやったほうが良さそうです。

## 参考
https://developer.apple.com/design/human-interface-guidelines/app-icons
