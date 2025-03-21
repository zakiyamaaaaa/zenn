---
title: "ClineでMCP入門：Git Tools"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [cline, mcp, git,ai]
published: true
---

## はじめに
- Clineとは

>CLIとエディタを使いこなすAIアシスタント、Clineをご紹介します。
Claude 3.7 Sonnetのエージェントコーディング機能により、Clineは複雑なソフトウェア開発タスクをステップバイステップで処理することができます。ファイルの作成と編集、大規模なプロジェクトの探索、ブラウザの使用、ターミナルコマンドの実行（許可を与えた後）などが可能なツールにより、コード補完や技術サポート以上の方法であなたをサポートします。クラインはモデル・コンテキスト・プロトコル（MCP）を使って新しいツールを作成し、自身の能力を拡張することもできる。自律AIスクリプトは従来、サンドボックス環境で実行されていたが、この拡張機能では、すべてのファイル変更とターミナルコマンドを人間が承認するGUIが提供され、エージェント型AIの可能性を探るための安全でアクセスしやすい方法が提供される。

*[公式](https://github.com/cline/cline)から引用*

- MCPの概要
MCPは、異なるツールやサービス間でコンテキストを共有するためのプロトコルです。Clineはこのプロトコルを利用して、Gitなどのツールを操作します。

- 本記事の目的
ClineとMCPを使ってGit操作を効率化する方法を紹介します。具体的な手順と例を交えて解説します。

:::message
前提として、Clineをインストールしている必要があります。
また、今回特に関係ないですが、使用モデルは`gemini-2.0-flash`になります。現時点では、無料の範囲で使うことができます。
`computer use`が使用不可なので、入門の方にも安心して使用できると思います。
ただし、現時点（2025/03/13）の話であり、今後料金体系が変動する可能性があるため、tokenの料金などを事前にチェックしといたほうがいいです。
:::

## 1. Git Toolsのインストール
MCPサーバーの設定は、本来であればJSONファイルを設定したりと少し面倒なのですが、2025年2月からClineで`MCP MarketPlace`が使えるようになり、VSCodeのExtensionと同じかたちで、気軽にMCPサーバーの追加をすることができます。
マーケットプレイスを開くには、VSCodeでClineの拡張を選択した画面で、右上のアイコンを選択してください。
![](/images/cline-mcp-git-tools/image1.png)

選択すると、次のような画面になり、Marketplaceから検索できます。
最初は、最新順でソートされてるので、GitHub Starsを押してみましょう。そうすると、スター数の多い順にMCPサーバーが表示されます。
今回は、現在２番目にある`Git Tools`を使用していきます。

![](/images/cline-mcp-git-tools/image2.png)


右のInstallボタンを押すと、MCPサーバーをインストールするTaskが開始されます。

![](/images/cline-mcp-git-tools/image4.png)

コマンドが求められます。コマンドを確認し、Run Commandで、実行します。
```bash
mkdir -p /Users/YOUR_NAME/Documents/Cline/MCP
```

なんやかんやで、インストールに成功したらデモンストレーションを促されます。
内容を確認して、タスクを実行します。

![](/images/cline-mcp-git-tools/image5.png)

自分の場合、途中でエラーが出ました。

![](/images/cline-mcp-git-tools/image6.png)

エラーの内容を判断して、推奨コマンドを促しています。
内容を確認して、実行しましたが、どうやらエラーの根本的解決に至りませんでした。

インストールされたGitToolsのページを見に行きます。
先程のMarketplaceから`Installed`を開きましょう。
そうすると、エラーメッセージが出ています。

![](/images/cline-mcp-git-tools/image7.png)

こちら調べたところ、次のIssueから解決しました。
https://github.com/cline/cline/issues/1160

command部分を次のようなフルパスにして設定してください。

```json:cline_mcp_settings.json
{
  "mcpServers": {
    "github.com/modelcontextprotocol/servers/tree/main/src/git": {
      "command": "/Users/YOUR_NAME/.pyenv/versions/3.8.0/bin/uvx",
      "args": ["mcp-server-git"],
      "disabled": false,
      "autoApprove": []
    }
  }
}
```
:::message
こちらのフルパスは、uvxのインストール方法によって異なるので、事前にwhich uxなどで調べてみてください。
:::

この内容で保存すると、先程のエラーが消えていました。

![](/images/cline-mcp-git-tools/image8.png)

それでは次のタスクを実行し、gitブランチ名を取得してみます。

```prompt:prompt
GitToolsから現在のgitブランチ名を取得して
```

![](/images/cline-mcp-git-tools/image9.png)

こんな感じで、Git Toolsを使用を求められました。
ここで、Auto-approveとすると、これ以降承認なしでMCPサーバーが使われます。

このような形で、現在のブランチ名を取得できました。
![](/images/cline-mcp-git-tools/image10.png)

GitToolsのインストールに成功しているようです。

## 2. 基本的なGit Tools
Git Toolsで使えるツールは、MCP ServerのGit Toolsを展開すると、確認できます。
今現在は、次の11個のgit commandが使えるようです。

* git status
* git diff unstage
* git diff staged
* git diff
* git commit
* git add
* git reset
* git log
* git create branch

それぞれのツールで、Auto-approveのチェックボックスがあるので、自動的に承認したい場合は、これをONにしましょう。

## まとめ
この記事では、Clineで、gitのMCP Serverの簡単な導入・使い方を説明しました。
マーケットプレイスの登場によって、簡単にMCPサーバーを導入することが可能となります。
MCPによって、開発の幅も広がるので、色々と触っていきたいです。

## 参考資料

https://github.com/cline/cline

https://github.com/cline/mcp-marketplace

https://github.com/modelcontextprotocol/servers/tree/main/src/git

