---
title: "gemini code assistでPRをレビューしてみた"
emoji: "📘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter,gemini,ai,github]
published: false
---

## はじめに
本日GoogleのAIツールであるGemini code assistが個人向けに無料で開放されました。
https://codeassist.google/

この記事では、そのうちのコードレビュー機能を試したものとなります。
なお、出力結果は2025年2月26日のものであり、差異が生じる可能性があることに留意ください。

## gemini-code-assist導入
こちらのページからgemini-code-assistをGitHubに統合します。
https://github.com/apps/gemini-code-assist

![](/images/gemini-code-assistant-tutorial/image1.png)

完了したらこのようなページが最後に表示されます。
![](/images/gemini-code-assistant-tutorial/image2.png)

GitHubに統合したかどうかはGitHubの`Settings>integrations Applications`の、`Installed GitHub Apps`で確認できます。
![](/images/gemini-code-assistant-tutorial/image3.png)

それでは、実際にCode Reviewを試してみます。

## Code Review
自分はFlutterエンジニアであり、Flutterの場合は最初に簡単なカウンターテンプレートを作ることができます。
プロジェクト作成した場合、Flutter開発者にはよく見慣れたカウンターアプリの画面がでてきます。タイトルを変更して、このような感じになります。
右下のボタンを押すと、数字が１ずつ足される簡単なアプリケーションです。

![](/images/gemini-code-assistant-tutorial/image10.png =300x)

### Pull Request
それでは、簡単にPull Requestを出していきます。
数字が３の倍数のときに赤くなるようにします。

```dart
Text(
    '$_counter',
    style: Theme.of(context).textTheme.headlineMedium!.copyWith(
        color: _counter % 3 == 0 ? Colors.red : Colors.black),
),
```
commitメッセージは次のようにしました。（英語としては完全ではないです。）
`number color changes red when _counter is 3 times`

この内容で、Pull Requestを出します。タイトルと内容も、このcommitメッセージと同じものとしました。

### gemini code assistantのレビュー
こんな感じのレビューをもらいました。
![](/images/gemini-code-assistant-tutorial/image4.png)

まずは、プルリクの全体的な総評がなされてるようです。
```md
Summary of Changes
This pull request, titled "number color changes red when _counter is 3 times", modifies the lib/main.dart file to change the color of the counter text to red when the _counter variable is a multiple of 3. The change involves using the copyWith method on the headlineMedium text style to conditionally set the color to red or black based on the _counter value.
```

トリビアもありました。クイズ？（他のレビューではなかったりするので、確率的に発生？）

```
Trivia time!

What is the purpose of the copyWith method in Flutter's TextStyle class?
Click here for the answer

The `copyWith` method in Flutter's TextStyle class is used to create a new TextStyle object with specified properties changed, while keeping the other properties the same as the original TextStyle.
```

プルリクに豆知識を入れ込むことは稀だと思うのですが、なかなかいいかもしれません。

次のコメントには、このようにしっかりとレビューしてくれてるようです。

![](/images/gemini-code-assistant-tutorial/image7.jpeg)

提案内容は、カラーロジックを分けたほうが読みやすいとのこと。的を得たレビューだと思います。優先度のラベルもあり、このレビュー内容がどれくらい推奨されるものかを示していそうです。


ここにあるようにコメントのところで、geminiコマンドを使えるそうです。
https://github.com/marketplace/gemini-code-assist


対話的に聞くこともできます。
心配なところがあるか、聞いてみました。

![](/images/gemini-code-assistant-tutorial/image6.png)

ちゃんと答えてくれてすごい。コミニケーション苦手なエンジニア淘汰されるかも。

日本語でレビューをお願いしましたが、今現在はできないようです。

![](/images/gemini-code-assistant-tutorial/image8.png)

### Code Review (wrong pull request)
それでは、次に間違った内容のPull Requestをやってみたいと思います。

次のようにコードを修正します。

```dart
Text(
    '$_counter',
    style: Theme.of(context).textTheme.headlineMedium!.copyWith(
        color: _counter % 4 == 0 ? Colors.blue : Colors.black),
    ),
```

しかし、コミットメッセージとプルリクの内容は「数字が５の倍数で黄色になる」とします。

レビュー内容はこのようになりました。

![](/images/gemini-code-assistant-tutorial/image9.png)

パット見わかりずらいかもしれませんが、SummaryとHighlightの部分で、「PRタイトルと内容では5の倍数と書いてるけど、4の剰余としている」とあります。
ただ、コードレビューでは4をマジックナンバーとしていることを指摘しています。
ここらへんはまだ発展途上かもしれません。それでも、この指摘も的を得ています。

## おわりに
本日発表されたgemini code assistantのレビュー機能をざっと使ってみました。
驚いたのは、これが無料で使えるということです。
今回は簡単な修正だったのもあるかもしれませんが、指摘内容が的確で2,3秒で結果が出てきたのが驚きました。
また、コメントのところで@geminiとすることで、対話的に聞くこともできて気になる箇所とかを集中的に聞けたりするため、とても効果的だなと思います。
人間が相手だとどうしても時間を取ったりすることだったり、質問内容を気遣いますが、AIであれば簡単に聞くことも容易です。
日々進化していくAI技術に対して、使いこなして効率性を高めていくことが求められそうです。

## 参考記事
https://codeassist.google/

https://github.com/apps/gemini-code-assist