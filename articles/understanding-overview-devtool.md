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

ここでも様々な機能が提供されているので、それを紹介していきます。

* select widget mode
これを有効にすることで、選択したウィジェットを参照することができます。



# 参考にした記事とか
