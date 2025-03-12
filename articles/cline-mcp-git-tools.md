---
title: "ClineでMCP入門：Git Tools"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

## はじめに
- Clineとは（簡単な紹介）
  - Clineは、Model Context Protocol (MCP) を利用した開発を支援するツールです。コマンドラインからGit操作などを簡単に行えます。
- MCPの概要（基本的な説明のみ）
  - MCPは、異なるツールやサービス間でコンテキストを共有するためのプロトコルです。Clineはこのプロトコルを利用して、Gitなどのツールを操作します。
- 本記事の目的
  - ClineとMCPを使ってGit操作を効率化する方法を紹介します。具体的な手順と例を交えて解説します。

## 1. Git MCPサーバーの設定
- MCPサーバーの設定ファイルの場所
  - Clineの設定ファイル（通常は`~/.cline/config.json`）にMCPサーバーの情報を記述します。このファイルはJSON形式で、MCPサーバーのアドレスや認証情報などを設定します。
- Git MCPサーバーの有効化方法
  - 設定ファイルにGit MCPサーバーを追加します。例えば、以下のような設定を追加します。
  ```json
  {
    "mcpServers": {
      "git": {
        "address": "http://localhost:8080",
        "type": "git"
      }
    }
  }
  ```
  Clineを再起動して、設定を反映させます。

## 2. 基本的なGit Tools
- git_status：リポジトリの状態確認
  - `git_status`ツールを使って、リポジトリの現在の状態を確認します。例えば、以下のコマンドを実行します。
  ```
  cline git_status
  ```
  これにより、変更されたファイルやステージングの状態が表示されます。
- git_diff_unstaged：ステージングされていない変更の確認
  - `git_diff_unstaged`ツールで、まだステージングされていない変更点を確認します。
  ```
  cline git_diff_unstaged
  ```
  変更された内容が詳細に表示されます。
- git_diff_staged：ステージングされた変更の確認
  - `git_diff_staged`ツールで、ステージング済みの変更点を確認します。
  ```
  cline git_diff_staged
  ```
  ステージングされた変更内容を確認できます。
- git_add：ファイルのステージング
  - `git_add`ツールを使って、ファイルをステージングします。例えば、`file.txt`をステージングするには、以下のコマンドを実行します。
  ```
  cline git_add file.txt
  ```
- git_commit：変更のコミット
  - `git_commit`ツールで、変更をリポジトリにコミットします。コミットメッセージを指定する必要があります。
  ```
  cline git_commit -m "変更内容の説明"
  ```
- git_reset：ステージングのリセット
  - `git_reset`ツールで、ステージングを取り消します。
  ```
  cline git_reset
  ```
  これにより、ステージングされたファイルが元の状態に戻ります。

## 3. ブランチ操作のGit Tools
- git_create_branch：新しいブランチの作成
  - `git_create_branch`ツールで、新しいブランチを作成します。例えば、`new_feature`というブランチを作成するには、以下のコマンドを実行します。
  ```
  cline git_create_branch new_feature
  ```
- git_checkout：ブランチの切り替え
  - `git_checkout`ツールで、ブランチを切り替えます。`new_feature`ブランチに切り替えるには、以下のコマンドを実行します。
  ```
  cline git_checkout new_feature
  ```

## 4. 実践的な使用例
- 新機能開発のシンプルなワークフロー
  - 新機能開発では、まず`git_create_branch`で新しいブランチを作成します。次に、変更を行い、`git_add`でファイルをステージングし、`git_commit`でコミットします。最後に、プルリクエストを作成してレビューを依頼します。
- バグ修正の基本的な流れ
  - バグ修正では、まず修正ブランチを`git_create_branch`で作成します。次に、バグを修正し、`git_add`と`git_commit`で変更をコミットします。最後に、プルリクエストを作成してレビューを依頼します。

## まとめ
- Git Toolsの利点
  - ClineとMCPを使うことで、Git操作を効率化できます。コマンドラインから簡単にGit操作を実行でき、開発効率が向上します。
- 参考リソース
  - Clineのドキュメント: [Clineの公式ドキュメント](https://example.com/cline-docs)
  - MCPの仕様: [Model Context Protocolの仕様](https://example.com/mcp-spec)
