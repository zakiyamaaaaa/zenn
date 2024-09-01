---
title: "iOSのExtensionでもFlavorsを使いたい！"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, Flavors, iOS, Xcode]
published: false
---

# はじめに
Flutterでのアプリ開発において、異なる環境を設定する場合には`dart-define-from-file`で`Flavors`を使うケースが多いと思います。
（`dart-define-from-file`については私が書いたこちらの記事に詳細があります）
https://zenn.dev/yamazaking/articles/dart_define_from_file_to_build

しかしプッシュ通知などの使う場合には、iOSの場合はExtensionを追加する必要があり、そのExtensionでBundle Identifierを`Flavors`からビルド環境ごとに切り替えたい、ということがあります。
そうしたケースでも、切り替えをする方法を説明していきます。

## やり方
### Extensionフォルダ内で.xcconfigファイルを生成
前提として`ios/Flutter/DartDefines.xcconfig` が生成されている環境とします。
こちらの生成については、こちらの記事がわかりやすいです。
https://zenn.dev/altiveinc/articles/separating-environments-in-flutter

まずは、Extensionのフォルダ内で、File>New>File（もしくは⌘+N）で、`Configuration Settings File`を選択して、`.xcconfig`ファイルを生成してください。
![](/images/flavor-to-xcode/image0.png)

ios/Flutter内の`.xcconfig`（Generated以外のDebug.xcconfigやRelease.xcconfig）と同様の`.xcconfig`ファイルを生成します。
`PushNotification.Debug.xcconfig`のように生成します。
生成した.xcconfigファイルに`#include "../Flutter/DartDefined.xcconfig"`として、ios/Flutterの`DartDefines.xcconfig`を参照します。

### RunnerのinfoでConfigurationsを設定する
Runner Projectを選択して、`Info`>`Configurations`にある、各ビルド設定のExtensionに先ほどの`.xcconfig`を適用します。
![](/images/flavor-to-xcode/image1.png)

### Build identifierの適用
最後に、ExtensionsのTargetを選択して、Build Settings>Packaging>Product Bundle Identifierで使用したFlavor変数を指定して完成です！
![](/images/flavor-to-xcode/image2.png)

## 参考にした記事とか
https://stackoverflow.com/questions/69808448/setting-ios-today-extension-bundle-version-to-flutter-flutter-build-number/69898579#69898579
