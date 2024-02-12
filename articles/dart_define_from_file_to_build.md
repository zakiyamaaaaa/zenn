---
title: "【Flutter】dart-define-from-fileを使ったビルドの環境分けとAndroidリリースビルドの注意点"
emoji: "🌊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, Dart, Flavor]
published: false
---

# はじめに
Flutterのビルド環境切替のための`--dart-define-from-file`の使い方の記事がメインです。

モバイルアプリで開発する際は、リリースビルドと開発ビルドを分ける必要がありますが、それらを手動で変更する場合、手間が多く、変更する箇所が間違っていたり、開発環境からリリース環境、そして開発環境に戻すときに、直すべきところを直してなかったりすると事故が置き、手戻りが発生する可能性が大きくなります。
そのため、ビルドの設定を自動的に反映するようにしたいことが多々あります。
iOSの場合は、Build configurationとして知られる項目を設定することで、ビルドを切替することが出来ます。

Flutterは特にマルチプラットフォームでのアプリ開発・ビルドが可能であるため、ビルド環境設定の切替を自動化することはネイティブのときより効用が多いと考えます。

調べていくと、FlutterはFlavorsというAndroid由来のビルド環境切替ライブラリを使っているそうで、公式でも詳細な使い方の記述があります。

https://docs.flutter.dev/deployment/flavors

自分も最初Flavorsのみを使ったビルド環境切替をやろうとしましたが、結構設定が複雑でした。
その他にも、ビルド環境を切り替えにFlutter3.7より導入された`--dart-define-from-file`が結構使われており、[Gbolahanさんの記事](https://gboliknow.medium.com/efficiently-configuring-your-flutter-app-with-dart-define-from-file-604d9bd49e7b)によると、--dart-define-from-filesは開発者が設定ファイルから環境変数をロードすることを可能にする代替アプローチを提供し、サードパーティのパッケージや手動更新なしで簡単に管理し、セキュリティを確保することができます。今回はこちらを採用してのビルド環境の切替を行いました。
※似た引数として、`--dart-define`がありますが、`--dart-define`を改良したものが`--dart-define-from-file`です。
[おかやまんさんの記事](https://zenn.dev/blendthink/articles/392607db0a65dd)から引用した`--dart-define`の欠点です。

>`--dart-define`には次のような課題がありました。
>
> - 多くの定義がある場合、起動コマンドが非常に長くなってしまう
> - 切り替えるパッケージが複数ある場合、保守が困難になる
> - これらの定義を Android と iOS で直接利用しようとすると、それぞれで Base64 でデコードしなければならない
> これらの課題を解決するために導入されました。

この記事のタイトルにもあるように`--dart-define-from-file`を用いて、Flutterでのビルド環境を分けていきます。
ちなみにビルドの設定などはVSCodeをもとにしています。

# --dart-define-from-fileを使ってみる
## 環境変数ファイルの設定
今回はdevelopment, staging, productionの３つのビルド環境を想定します。
ビルド時に参照する環境変数のファイルを設定します。
プロジェクトのrootから任意のフォルダ名でフォルダ作成し、ここに.json(または.env)ファイルをそれぞれの環境ごとに作成していきます。
今回はフォルダ名をflavorとしました。
JSONのそれぞれのkeyをビルド時に環境変数として利用できます。なので、任意のkey,valueをセットして、それぞれの環境ごとに反映することができます。そのため、すべてのファイルでkeyは統一しましょう。

```json:flavor/dev.json
{
    "flavor": "development",
    "appIdSuffix": ".dev",
    "flavorPrefix": "dev"
}
```

```json:flavor/stg.json
{
    "flavor": "staging",
    "appIdSuffix": ".stg",
    "flavorPrefix": "stg"
}
```

```json:flavor/prod.json
{
    "flavor": "production",
    "appIdSuffix": "",
    "flavorPrefix": ""
}
```

この状態で、一応`--dart-define-from-file`でのビルドが可能ですが、これだけではまだ反映されません。

```bash
flutter run --dart-define-from-file=flavor/dev.json
```
`--dart-define-from-file`のあとに、設定するファイルのパスを指定します。

# アプリアイコンを環境別に分ける
iOS、Androidで必要なアイコン画像の種類が異なり、さらにそれぞれのOSで解像度の違うアイコン画像が様々必要となり、これをひとつひとつ設定するのはとても手間です。
そのため、Flutterでよく使われているflutter_launcher_iconsパッケージを使っていきます。

## 環境別にアイコン画像をプロジェクト内に配置する
普通のアイコン設定する感じです。
適当な場所にそれぞれの環境ごとのアイコンを配置します。
今回は/images/iconsの配下に設定しました。

- images/icons/icon-development.png
- images/icons/icon-staging.png
- images/icons/icon-production.png

![](/images/dart_define_from_file_to_build/icon-development.png =300x)
![](/images/dart_define_from_file_to_build/icon-staging.png =300x)
![](/images/dart_define_from_file_to_build/icon-production.png =300x)

flutter_launcher_iconsのパッケージを導入します。

```bash
flutter pub add flutter_launcher_icons --dev
```

プロジェクトのルートディレクトリに、Flavorごとの設定ファイルを作成していきます。
ファイル名は以下のように先ほど作成したJSONファイルでflavorのvalueを指定し、flutter_launcher_icons-flavorValueとしてください。
これによって、AppIcon-flavorNameが作成され、アプリアイコンのところで後述するようにビルド時に$flavorでそれぞれのビルド環境のアイコン画像を参照できます。

- flutter_launcher_icons-development.yaml
- flutter_launcher_icons-staging.yaml
- flutter_launcher_icons-production.yaml

```yaml:flutter_launcher_icons-development.yaml
flutter_launcher_icons:
  image_path: "images/icons/icon-development.png"
  android: true
  ios: true
  remove_alpha_ios: true
```

remove_alpha_iosはiOSの場合、アイコンに透過が使われていると審査等で弾かれるので、透過を除くようにtrueにしています。

## アイコン画像の書き出し

次のコマンドでアイコン画像がそれぞれのOSごとに合わせて自動的に生成されます。

```bash
dart run flutter_launcher_icons
```

iOSでは ios/Runner/Assets.xcassets/AppIcon-{flavor}.appiconset/
Androidでは android/app/src/{flavor}/mipmap**/ic_launcher.png
という形式で出力されます。

## iOS側の対応

### info.plistでアプリ名を環境ごとに変更
info.plistを編集します

```plist
<key>CFBundleDisplayName</key>
	<string>AppName $(flavorPrefix)</string>
```

こちらで設定したjsonのキーであるflavorPrefixが参照されます。

### BundleIDを環境ごとに変更
BundleIDを変更することで、別のアプリという認識となり、環境ごとにアプリが分けることが出来ます。
TargetsのBuild Settings→Packaging→Product Bundle Identifierで次のように変更します。

PRODUCT_BUNDLE_IDENTIFIER = com.yourdomain.yourapp$(appIdSuffix);

### アプリiconの変更
Xcodes>Runner>TARGETS Runner>Build SettingsのPrimary App Icon Set Nameのところで
AppIcon-$(flavor)に変更します。
これによって、例えば/flavor/dev.jsonのflavorというkeyに対してdevelopmentというvalueが適応されて、AppIcon-developmentの画像がアイコンに適用される、という感じです。

これでiOS側の設定は完了です🎉
### 起動
次のコマンドからビルドしましょう

```bash
flutter run --dart-define-from-file=flavor/stg.json
flutter run --dart-define-from-file=flavor/prod.json
```

画像のようにアイコンとアプリ名が変わっているのがわかります。

![](/images/dart_define_from_file_to_build/image1.png)

### launch.jsonを設定して、ターミナルからの入力を省略する。
これまでの設定で一応ビルド環境を分けることが可能となったのですが、いちいちコマンドを入力するのが面倒なので、VSCodeのビルド設定ファイルを作成し、簡単にします。

VSCodeの左側のビルドメニューを選択し、`launch.jsonファイルを作成`を選択しましょう。.vscodeの配下にlaunch.jsonが生成されます。

![](/images/dart_define_from_file_to_build/image2.png =300x)


次のようにlaunch.jsonファイルを書き換えます。

```json:.vscode/launch.json
{
    // IntelliSense を使用して利用可能な属性を学べます。
    // 既存の属性の説明をホバーして表示します。
    // 詳細情報は次を確認してください: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "appName (debug)",
            "request": "launch",
            "type": "dart",
            "args": [
                "--dart-define-from-file=flavor/dev.json"
            ]
        },
        {
            "name": "appName (profile)",
            "request": "launch",
            "type": "dart",
            "flutterMode": "profile",
            "args": [
                "--dart-define-from-file=flavor/stg.json"
            ]
        },
        {
            "name": "appName",
            "request": "launch",
            "type": "dart",
            "flutterMode": "release",
            "args": [
                "--dart-define-from-file=flavor/prod.json"
            ]
        }
    ]
}
```
VSCodeでビルドの環境を変更する場合は、左側のビルドメニューを選択し、実行とデバッグのところで切り替えができます。

## Androidリリース用ビルドをする際の注意点
Androidアプリを公開する場合は、flutter build appbundleとしてApp Bundleファイル（拡張子.aab）を生成する必要があります。しかし、この場合flavorを参照せずビルドエラーとなるため、次のような形でflavorを参照するようにビルドしましょう。

```bash
flutter build appbundle --dart-define-from-file=flavor/prod.json
```

ちなみに、これでAndroidアプリ公開できる！と喜んだのもつかの間、前に登録していて更新しないまま放置していたDeveloperアカウントがいつの間にか削除されてました。
新しくDeveloperアカウント登録しようにも、最近の仕様の変更によりかなり個人開発者には厳しいので今のところAndroidのリリースは様子見してます。定期的なアプリの更新とGoogleからのメールをチェックするようにしましょう！

# 参考にした記事

- https://qiita.com/kazakago/items/e1027619d0361848f3b8
- https://zenn.dev/blendthink/articles/392607db0a65dd
- https://medium.com/flutter-jp/icon-935d637d2da0
- https://gboliknow.medium.com/efficiently-configuring-your-flutter-app-with-dart-define-from-file-604d9bd49e7b