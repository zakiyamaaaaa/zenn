---
title: "【Flutter】OneSignalを使ってPush通知を簡単に実装してみる（iOS）"
emoji: "⛳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, onesignal, ios]
published: false
---

# 記事概要
OneSignalを利用してPush通知を簡単に実装してみる。アプリ側はFlutterを使い、OSはiOS iPhoneでやっています。（今後Androidのほうも書き足す予定）

# 前提知識：OneSignalとは
https://onesignal.com/

OneSignalは、次のようなさまざまなチャネルにメッセージを送信して、ユーザーの関心を維持できるツールです。
- プッシュ通知
- SMS
- メール
- アプリ内通知

今回はプッシュ通知を扱います。

もともと[Supabase](https://supabase.com/)というオープンソースサービスの評判がよく、こちらと連携がしやすいと言われているメッセージングサービスが今回扱うOneSignalとなります。

https://supabase.com/partners/integrations/onesignal

モバイルからWebに対応しており、今回の記事ではFlutterを採用しています。
料金体系は下記のようになっており、無料プランでも１万通/月のメール送信、モバイルプッシュ通知は制限なしとなります。そのため最初に試すハードルが低いのも魅力の一つです。

![](/images/onesignal-push-notification/image1.png)

# 本編
## OneSignalの設定
[OneSignalのページ](https://onesignal.com/)から、アカウント登録をします。

アカウントを登録してログインすると、サービスに使うアプリはどのようなタイプなのか、どのような機能を使うのかなどの項目を入力します。

入力が完了すると、早速最初の配信設定を行います。
OneSignal上のアプリ名や所属組織を入力し、最初の配信チャネルを選択します。
ここではまずはiOSの配信をしたいので、Apple iOS(APNs)を選択します。

![](/images/onesignal-push-notification/image2.png)

次のページに進むと、証明書の設定を行います。

### p8証明書の設定
`.p8 Auth Key Recommend`の設定を行います。

1. ブラウザから https://developer.apple.com/ にアクセスし「Account」へ（うまくいかない場合はSafariを使ってください）
2. iOS dev center にログイン
3. Certificates, IDs & Profiles > Keys ページ(https://developer.apple.com/account/resources/authkeys/list)に移動し、Keyを追加するため、プラスボタンを押し、Keyを新規作成します。

![](/images/onesignal-push-notification/image3.png)

4. Keyの名前をつけ、Apple Push Notification servicesのチェックボックスを選択肢、Continueボタンで次の画面へ。
5. 完了画面が表示されここで作成したKeyのダウンロードができるので、.p8ファイルをダウンロードします。このときKEY IDはあとで使うのでどこかにメモしてください

### Apple iOS (APNs) Configuration

![](/images/onesignal-push-notification/image4.png)

それではさきほどのOneSignalのページに戻りiOSの設定を行います。
`APNs Authentication Type`は.p8 Auth Keyとし、次の項目で先ほどダウンロードした.p8ファイルをアップロードします。

Key IDには先ほど.p8ファイルをダウンロードしたページに記載されたものを使用します。
Team IDはApple Developerアカウントのページで確認できます。
https://developer.apple.com/account のページを下にスクロールするとTeam IDの記載があります。

![](/images/onesignal-push-notification/image5.png)

App Bundle IDはApple Develop Center、Xcodeで確認できます。

**Apple Develop Center**
https://developer.apple.com/account/resources/identifiers/list

ここから該当のアプリ→Bundle IDの項目があります。

**Xcode**
![](/images/onesignal-push-notification/image5-1.png)

オプションの設定としてProvisional Notificationを選択することができます。

https://documentation.onesignal.com/docs/ios-customizations

次のページへ進むと使用するプラットフォームを選択するページとなります。

![](/images/onesignal-push-notification/image6.png)

今回はFlutterで行うため、Flutterを選択して次に進みます。

これでOneSignal側のPush Notification事前設定が終わりました。
次のページにすすむと、OneSignal向けのIDやFlutterのドキュメントがあります。
**Flutter SDKセットアップドキュメント**

https://documentation.onesignal.com/docs/flutter-sdk-setup

ここからの内容は、主にこちらのドキュメントを日本語に翻訳したものとなります。

## iOS setup

プロジェクトの ios フォルダーにある Runner.xcworkspace ファイルを Xcode で開きます。 ルートプロジェクト > メイン アプリ ターゲット > 署名と機能を選択します。 プッシュ通知が有効になっていない場合は、[+ 機能] をクリックしてプッシュ通知を追加します

![](/images/onesignal-push-notification/82b4cec-1.png)

もう一度 [+ 機能] をクリックし、バックグラウンド モードを追加します。次に、リモート通知を確認します。

![](/images/onesignal-push-notification/2269f7e-2.png)

### 通知サービス拡張機能の追加
OneSignalNotificationServiceExtensionを使用すると、iOSアプリケーションで画像、ボタン、バッジを使ったリッチな通知を受け取ることができます。また、OneSignalのConfirmed Delivery分析機能にも必要です。

![](/images/onesignal-push-notification/6eb36bd-73ca2a9-1.png)

Xcode で File > New > Target... を選択します。

![](/images/onesignal-push-notification/6eb36bd-73ca2a9-2.png)

Notification Service Extensionを選択し、Nextをクリックします。
製品名に「OneSignalNotificationServiceExtension」と入力し、「完了」を押します。
「完了」を選択した後に表示されるダイアログではスキームをアクティブにしないでください。 「スキームのアクティブ化」プロンプトで「キャンセル」を押します。

OneSignalNotificationServiceExtension ターゲットと全般設定を選択します。 [最小デプロイメント] をメイン アプリケーション ターゲットと同じ値に設定します。これは iOS 11 以降である必要があります。

### App Groupsに追加
アプリ グループを使用すると、アプリがアクティブでない場合でも、通知を受信したときにアプリと OneSignalNotificationServiceExtension が通信できるようになります。これはバッジと確認済み配達に必要です。

Select your Main App Target > Signing & Capabilities > + Capability > App Groups.

![](/images/onesignal-push-notification/ca64b5e-flutter-appgroups1.png)

アプリ グループ内で、[+] ボタンをクリックします。

アプリ グループ コンテナを group.YOUR_BUNDLE_IDENTIFIER.onesignal に設定します。YOUR_BUNDLE_IDENTIFIER はメイン アプリケーションの「バンドル識別子」と同じです。

![](/images/onesignal-push-notification/a51bc81-flutter-appgroups2.png)

[OK] を押して、OneSignalNotificationServiceExtension ターゲットに対して繰り返します。 [OneSignalNotificationServiceExtension ターゲット] > [署名と機能] > [+ 機能] > [アプリ グループ] を選択します。

![](/images/onesignal-push-notification/98c0fa6-flutter-appgroups3.png)

App Groups内で+ボタンをクリックします。

App Groupsのコンテナをgroup.YOUR_BUNDLE_IDENTIFIER.onesignalに設定します。YOUR_BUNDLE_IDENTIFIERはメインアプリケーションの「バンドル識別子」と同じです。

OneSignalNotificationServiceExtensionは含めないでください。

### OneSignal SDKをOneSignalNotificationServiceExtensionに追加

Podfileに次のコードを追加します。

```bash: Podfile
target 'OneSignalNotificationServiceExtension' do
  use_frameworks!
  pod 'OneSignalXCFramework', '>= 5.0.0', '< 6.0'
end
```

Podfile の先頭に platform :ios, '11.0' があることを確認してください。こちらはアプリの環境によって11以降であれば問題ないです。

```bash:Podfile
# Uncomment this line to define a global platform for your project
platform :ios, '11.0'
```

ターミナルを開き、ios ディレクトリに移動し、pod install を実行します。

:::message
以下のエラーが表示された場合は、use_frameworks を追加してください。ポッドファイルの先頭に移動して、再試行してください。

Runner (true) and OneSignalNotificationServiceExtension (false) do not both set use_frameworks!.
:::

Xcode プロジェクト ナビゲーターで、OneSignalNotificationServiceExtension フォルダーを選択し、NotificationService.m または NoticeService.swift ファイルを開きます。
ファイルの内容全体を次のコードに置き換えます。

```swift: NotificationService.swift
import UserNotifications

import OneSignalExtension

class NotificationService: UNNotificationServiceExtension {
    
    var contentHandler: ((UNNotificationContent) -> Void)?
    var receivedRequest: UNNotificationRequest!
    var bestAttemptContent: UNMutableNotificationContent?
    
    override func didReceive(_ request: UNNotificationRequest, withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void) {
        self.receivedRequest = request
        self.contentHandler = contentHandler
        self.bestAttemptContent = (request.content.mutableCopy() as? UNMutableNotificationContent)
        
        if let bestAttemptContent = bestAttemptContent {
            /* DEBUGGING: Uncomment the 2 lines below to check this extension is executing
                          Note, this extension only runs when mutable-content is set
                          Setting an attachment or action buttons automatically adds this */
            // print("Running NotificationServiceExtension")
            // bestAttemptContent.body = "[Modified] " + bestAttemptContent.body
            
            OneSignalExtension.didReceiveNotificationExtensionRequest(self.receivedRequest, with: bestAttemptContent, withContentHandler: self.contentHandler)
        }
    }
    
    override func serviceExtensionTimeWillExpire() {
        // Called just before the extension will be terminated by the system.
        // Use this as an opportunity to deliver your "best attempt" at modified content, otherwise the original push payload will be used.
        if let contentHandler = contentHandler, let bestAttemptContent =  bestAttemptContent {
            OneSignalExtension.serviceExtensionTimeWillExpireRequest(self.receivedRequest, with: self.bestAttemptContent)
            contentHandler(bestAttemptContent)
        }
    }  
}
```
::::details　No such module 'OneSignal'
:::message
ちなみにここで私はハマりました
import OneSignalExtensionのところでNo such module~と出て該当のライブラリが認識されてません。という内容のエラーです。
調べてみると公式のほうでもトラブルシューティングとして対応内容が書いてありました。

https://documentation.onesignal.com/docs/troubleshooting-ios

しかし、Embedded frameworkに追加するところで該当のOneSignalのフレームワークが出てこず、悩んでいたところ、iOS SetupのページにSwift Package Managerを利用して追加する方法が書かれていました。（Flutterにも書いて・・・）

以下Swift Package Managerを使用したやり方です。
OneSignal SDK は Swift パッケージとして追加できます (Objective-C でも動作します)。
パッケージの URL を入力します: https://github.com/OneSignal/OneSignal-XCFramework 
依存関係ルールが次のメジャー バージョンまでに設定されていることを確認します。 「パッケージの追加」をクリックします

これでEmbedded frameworkのほうにも追加できるようになりました。

ここが公式の文書と違うところがあるのですが、下記のようなフレームワークを追加します。
* OneSignalExtension
* OneSignalFramework
* OneSignalInAppMessages
* OneSignalLocation

![](/images/onesignal-push-notification/image7.png)

※Pods_One〜のライブラリはビルドすると自動的に入ってくるので、設定するときに入ってる必要はないです

:::
::::

ここで試しにビルドしてみてエラーが生じなければここまでの過程は完了です。
自分の場合は上のところで書いたように、OneSignalのライブラリを読み込むところに若干ハマり、その次にCoccoaPodsがうまく動かないエラーに遭遇しました。CocoaPodsについては今回のトピックからずれるのでここでは割愛させていただきます。

## Flutter側の実装
iOS 11~、Xcode14が必須となります。
※Android：「Google Play ストア (サービス)」がインストールされている Android 5.0 以降のデバイスまたはエミュレータ

### packageの追加
Onesignalのパッケージを追加します。

```pubspec.yaml
dependencies:
	onesignal_flutter: ^5.1.2
```

`main.dart`にライブラリをインポートします。

```dart:main.dart
import 'package:onesignal_flutter/onesignal_flutter.dart';
```

### 初期化
OneSignalを初期化し、通知を受け取る準備ができます。
次のコードで通知許可ダイアログを表示します。
ここでは例として`main.dart`に書いています。
`YOUR APP ID HERE`のところはOneSignalのページで設定完了したときに表示されるIDを入力します。

![](/images/onesignal-push-notification/image7-5.png)

```dart:main.dart
//Remove this method to stop OneSignal Debugging 
OneSignal.Debug.setLogLevel(OSLogLevel.verbose);

OneSignal.initialize("<YOUR APP ID HERE>");

// The promptForPushNotificationsWithUserResponse function will show the iOS or Android push notification prompt. We recommend removing the following code and instead using an In-App Message to prompt for notification permission
OneSignal.Notifications.requestPermission(true);

```

ここまでできたら、一度ビルドしてみて通知を許可しましょう。

![](/images/onesignal-push-notification/image8.png =300x)

これでOneSignal管理画面でSubscriptionのユーザーとして登録されます。
お疲れ様でした！実装はこんな感じで終わりです！

## 通知を送信する
それでは最後に確認としてテスト通知を配信したいと思います。
OneSignalの管理ページからMessages-Pushを選択し、右上のNew Messageから「New Push」を選択します。
1. Audience
ここではターゲットを設定します。すべての顧客と特定の顧客の選択肢があるので適切なほうを選びましょう

2. Message
ここでは具体的なメッセージ内容を設定していきます。

![](/images/onesignal-push-notification/image9.png)

画像やURLを設定することも可能です。

ここで右上のボタン「Send Test Push」を選択してテスト通知を送ってみます。
まだテストユーザーがいない場合は下の図のようなダイアログが表示されるかと思います。

「Add Test Users」ボタンを押すとサブスクリプション登録レコードが表示されます。
このレコードに登録しているのは、さきほどアプリを起動して許可したユーザー担っているかと思います。（Subscription Statusがチェックマークついてると思います）このレコードはテスト向けとは別のレコードとなっているので、テストユーザーとしても登録します。左側の「・・・」から「Add to Test Subscriptions」を選ぶとTestユーザー名を入力するダイアログが出てくるので適当な名前を入力してみましょう。
完了したら、さきほどの通知設定のページのところで出ているテストプッシュ通知のダイアログのところでRefreshを押すと、更新されてさきほど追加したテストユーザーが出てくるはずです。

![](/images/onesignal-push-notification/image10.png)

ここでチェックボックスにチェックをいれ、右下の「Send Test Push」ボタンを押してみましょう。
ここまで完了していれば、さきほど登録したデバイスにプッシュ通知が届くはずです。

![](/images/onesignal-push-notification/image11.png =300x)

なお、今回はSimulatorで試してみましたが、簡単に実行することができました。実機テストが好ましいですが、シミュレーターでぱぱっと試してみるぶんにはこれでいいと思います。

# まとめ
今回は時間の都合上iOSのみでのプッシュ通知テストをやってみました。思ったよりも簡単にできて、公式のドキュメントも整備されており、とてもスムーズに行うことができました。Androidのほうも別に設定して、Flutterで即座に使うことができると思いますので、有力なメッセージングツールとしてOneSignalが有用だと思います。


# 参考にさせていただいた記事
https://support.fanship.jp/hc/ja/articles/900006538803-APNs%E8%A8%BC%E6%98%8E%E6%9B%B8-p8%E5%BD%A2%E5%BC%8F-%E3%81%AE%E7%99%BA%E8%A1%8C%E6%96%B9%E6%B3%95