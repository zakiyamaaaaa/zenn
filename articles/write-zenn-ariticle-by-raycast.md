---
title: "Raycastを使用してZennの記事を”爆速”で生成する"
emoji: "🌙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Zenn, Bash, Raycast]
published: false
---

# Raycastとは
最近聞くようになったランチャーツールのRaycast
https://www.raycast.com/

Hotkeyを割り当てたり、独自のコマンドラインを作成でき、MacでいうSpotlightの上位互換です。
しかも無料で使うことができます（２０２４年１月時点）。有料はChatGPTと組み合わせてなんかできるらしいです。

# Zennの記事生成コマンドを都度調べるのを止めたい！
ここで本題です。
Zennで記事を作成するときにCLIから作成するときに、コマンドを思い出すのが面倒臭いと感じたことはありませんか？
```
$ npx zenn new:article --slug 記事のスラッグ --title タイトル --type idea --emoji ✨
```

ここで登場するのが冒頭にだした**Raycast**です！
まずRaycastの設定画面（⌘+,）を表示します
![](/images/write-zenn-article-by-raycast/image.png)

右側の＋ボタンから、Create Script Commandを選択します

![](/images/write-zenn-article-by-raycast/image2.png)

それぞれの項目はこのようにします
Template: Bash
Mode: Compact
Title: (好きなやつ)
Description: (オプション)
Package name: (オプション)

:::message
Titleはコマンドを実行するときの実行名になります
つまり、zennというTitleにすると、zennで実行できるようになります。
ちなみにこちらの設定は後ほど生成されるshファイルから変更可能です。
:::

良ければ「⌘+↵」を押します。そうすると、タイトル名のshファイルが生成されます。
このshファイルは最初にテンプレートとなっていますので、zennの記事が自動的に生成するように修正します。

コード全体は次の通りです。

```bash
#!/bin/bash -l

# Required parameters:
# @raycast.schemaVersion 1
# @raycast.title zenn
# @raycast.mode compact

# Optional parameters:
# @raycast.icon 📝
# @raycast.packageName zenn

# Documentation:
# @raycast.argument1 { "type": "text", "placeholder": "slug", "optional": false}

cd /Users/yourName/Documents/Zenn

npx zenn new:article --slug $1
code -g "./articles/$1.md"
```

:::message
Zennの記事を管理しているフォルダを適宜変更してください
:::

このコードでは、指定のフォルダを参照して、Zennの記事生成するコマンドを実行し、その後に記事をVsCodeで開くようにしています。
なお、実行時にZennのslugを引数設定しています。
このslug設定は12字〜50字、英数字、_-のみとなっており、それ以外を指定するとただのmarkdownファイルとなります。
成功すると、冒頭に↓のような設定がmarkdownファイルにでてきます。

---
title: ""
emoji: "🦊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: true
---

これでZennの記事執筆が爆速ではかどりますね！！