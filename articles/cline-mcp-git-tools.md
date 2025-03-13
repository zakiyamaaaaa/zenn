---
title: "ClineでMCP入門：Git Tools"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [cline, mcp, git,ai]
published: false
---

## はじめに
- Clineとは（簡単な紹介）
  - Clineは、Model Context Protocol (MCP) を利用した開発を支援するツールです。コマンドラインからGit操作などを簡単に行えます。
- MCPの概要（基本的な説明のみ）
  - MCPは、異なるツールやサービス間でコンテキストを共有するためのプロトコルです。Clineはこのプロトコルを利用して、Gitなどのツールを操作します。
- 本記事の目的
  - ClineとMCPを使ってGit操作を効率化する方法を紹介します。具体的な手順と例を交えて解説します。

## 1. Git Toolsのインストール
MCPサーバーの設定は、JSONファイルを設定したりと少し面倒なのですが、2025年2月からClineで`MCP MarketPlace`が使えるようになり、VSCodeのExtensionと同じかたちで、気軽にMCPサーバーの追加をすることができます。
マーケットプレイスを開くには、VSCodeでClineの拡張を選択した画面で、こちらのアイコンを選択してください。
![](/images/gemini-code-assistant-tutorial/image1.png)

選択すると、次のような画面になり、Marketplaceから検索できます。

![](/images/gemini-code-assistant-tutorial/image2.png)

最初は、最新順でソートされてるので、GitHub Starsを押してみましょう。そうすると、スター数の多い順にMCPサーバーが表示されます。
今回は、現在２番目にある`Git Tools`を使用していきます。

![](/images/gemini-code-assistant-tutorial/image3.png)

右のInstallボタンを押すと、MCPサーバーをインストールするTaskが開始されます。

![](/images/gemini-code-assistant-tutorial/image4.png)

コマンドが求められます。コマンドを確認し、Run Commandで、実行します。
```bash
mkdir -p /Users/shoichiyamazaki/Documents/Cline/MCP
```

なんやかんやで、インストールに成功したらデモンストレーションを促されます。
内容を確認して、タスクを実行します。

![](/images/gemini-code-assistant-tutorial/image5.png)

自分の場合、途中でエラーが出ました。

![](/images/gemini-code-assistant-tutorial/image6.png)

エラーの内容を判断して、推奨コマンドを促しています。
内容を確認して、実行します。

どうやらエラーの根本的解決に至りませんでした。

インストールされたGitToolsのページを見に行きます。
先程のMarketplaceからInstalledを開きましょう。
そうすると、エラーメッセージが出ています。

![](/images/gemini-code-assistant-tutorial/image7.png)

こちら調べたところ、次のIssueから解決しました。

command部分を次のようなフルパスにして設定してください。
```json
{
  "mcpServers": {
    "github.com/modelcontextprotocol/servers/tree/main/src/git": {
      "command": "/Users/shoichiyamazaki/.pyenv/versions/3.8.0/bin/uvx",
      "args": ["mcp-server-git"],
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

この内容で保存すると、先程のエラーが消えていました。

それでは次のタスクを実行し、gitブランチ名を取得してみます。

```prompt
GitToolsから現在のgitブランチ名を取得して
```

こんな感じで、Git Toolsを使用を求められました。
ここで、Auto-approveとすると、これ以降承認なしでMCPサーバーが使われます。

このような形で、現在のブランチ名を取得できました。
![](/images/gemini-code-assistant-tutorial/image8.png)

GitToolsのインストールに成功しているようです。

## 2. 基本的なGit Tools
Git Toolsで使えるツールは、MCP ServerのGit Toolsを展開すると、確認できます。
今現在は、次の11このgit commandが使えるようです。
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
git pushは現時点では使えないので、勝手にpushすることは防げるようです。

## まとめ
- Git Toolsの利点
  - ClineとMCPを使うことで、Git操作を効率化できます。コマンドラインから簡単にGit操作を実行でき、開発効率が向上します。

- 参考リソース
https://github.com/cline/mcp-marketplace

https://github.com/modelcontextprotocol/servers/tree/main/src/git

