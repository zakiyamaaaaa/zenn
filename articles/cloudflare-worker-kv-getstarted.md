---
title: "ローカル環境でCloudflareのWorker/KVの使用例（Qiita APIを題材に）"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Cloudflare, TypeScript, Qiita]
published: true
---

# はじめに
この記事では、ローカル環境でCloudflareのWorkerを扱う方法について説明していきます。
Cloudflareは最近、というか少し前から有名なサービスでホスティングやドメインの提供などクラウドインフラの様々な機能を提供しています。
今回はWorkerを利用して、APIサーバーを作成する方法について説明しています。
普段、あまりインフラ・サーバー側に慣れてない方でもわかりやすい内容となっています。
結構ローカルの開発環境で実行するところに苦戦して、日本語の参考記事が少なかった印象だったため、こちらの記事が参考になれば幸いです。

## Worker
CloudflareでWorkerを作成する方法としては、ダッシュボード上で作成する方法とCLIで作成する方法があります。

### ダッシュボードでWorkerを作成する
Cloudflareに登録して、ダッシュボードのトップを開きます
![](/images/cloudflare-worker-kv-getstarted/image1.png)

左のメニューからWorkers & Pages->概要を選択します。
ここで、現在作成済みのWorkerが表示されます。
作成ボタン→ワーカーの作成ボタンを押して、今回のWorkerを作成していきます。

次のページでは、”Hello World” Workerとなっており、リクエストされたらHello Worldを返すシンプルなWorkerのテンプレートとなっています。
Workerの名前は適当なものを入力してください。
ここでは、`qiita-api-sandbox`としました。
![](/images/cloudflare-worker-kv-getstarted/image2.png)

下にある展開ボタンを押すと作成完了となります。
最初に見たWorkers＆Pagesの概要を見てみると、先程作成したWorkerができているのがわかります。
こちらを選択すると、そのWorkerの内容やログが確認できます。
![](/images/cloudflare-worker-kv-getstarted/image3.png)
![](/images/cloudflare-worker-kv-getstarted/image4.png)

次にCLIでWorkerを作成する方法を説明します。

### CLIでWorkerを作成する

#### Wrangler
WranglerはCloudflareの開発、デプロイ、管理を簡単に行うためのCLIです。Node.js（16.17.0以上）とnpmが必要なため、適宜インストールしてください。

##### Wranglerのインストール
Wranglerは各プロジェクトにローカルにインストールされます。これにより、チームで扱う場合は、同じバージョンを使用してください。
Wranglerのインストールは次のコマンドです。

**プロジェクト内でインストールする場合**
```bash
npm install wrangler --save-dev
```

**グローバルにインストールする場合**
```bash
npm install -g wrangler
```

:::message
もしWranglerがインストールできなかった場合は、`npx wrangler`を使用して、最新のバージョンをインストールしてください。
:::

インストールの確認は次のコマンドです。

```bash
npx wrangler --version
// or
npx wrangler version
// or
npx wrangler -v
```

##### Wranglerの更新
npmを用いて、Wranglerを更新する例です。

```bash
npm install wrangler@latest
```

#### CLIでWorkerを作成する
それでは、Wranglerがインストールされた状態でWorkerを作成していきます。
`<WORKER_NAME>`にWorkerの名前をいれて、下記のコマンドを実行します。

```bash
npm create cloudflare@latest -- <WORKER_NAME>
```

実行すると、次にどのようなテンプレートを使うかを聞いてきます。
ここでは、`Hello World example`を選択します。

![](/images/cloudflare-worker-kv-getstarted/image5.png)

次にHello Worldテンプレートの使用する種類を聞かれます。
ここでは、`Hello World Worker`を選択します。

![](/images/cloudflare-worker-kv-getstarted/image6.png)
使用する言語を選びます。TypeScript,JavaScript,Python(beta)から選択できます。
ここでは、`TypeScript`を選択します。

![](/images/cloudflare-worker-kv-getstarted/image7.png)

最後にデプロイするかどうかを聞かれます。ここではNoと答えます。
これで、Workerがローカル環境に作成されます。
ダッシュボードに反映はされないので、したい方は`wrangler deploy`とすれば反映されます。

#### Workerを起動する
CLIでWorkerを立ち上げる場合は、次のコマンドを実行します。

```bash
wrangler dev
```

起動が成功すると、つぎのようなプロンプトがでてきます

![](/images/cloudflare-worker-kv-getstarted/image8.png)

`[b]open browser`を行うと、`Hello World!`が表示されるかと思います。
これでWorkerの作成は終わりです。

#### Workerの削除
削除したい場合はCLI上ではできずダッシュボードで削除を行ってください。
Workerの詳細ページの設定のところに、削除できる箇所があります。

![](/images/cloudflare-worker-kv-getstarted/image9.png)

### Qiita API
それではここからQiita APIをリクエストしてレスポンスするWorkerを作成していきます。

#### Qiita APIの動作確認
Qiita APIについては、参考資料に記載していますので、こちらを参考ください。
記事一覧を読み込みたいだけなので、スコープではread_qiitaにチェックをいれ、アクセストークンを発行します。

記事一覧を表示するエンドポイントはこちらです

```txt
[GET] https://qiita.com/api/v2/items
```

これをcurlでリクエストします。
AuthorizationにはBearerを指定します。
`<ACCESS_TOKEN>`には先程発行されたアクセストークンをいれてください。

```bash
curl -H 'Authorization: Bearer <ACCESS_TOKEN>' 'https://qiita.com/api/v2/items'
```

レスポンスに記事一覧ぽいのが入ってたら成功です。

#### WorkerでAPIリクエストする
それでは、前項でやってきたことをWorkerでやっていきます。
さきほど作成したWorkerの/src/index.tsを編集していきます。

初期状態だとこのような感じになってるかと思います。

```typescript
/**
 * Welcome to Cloudflare Workers! This is your first worker.
 *
 * - Run `npm run dev` in your terminal to start a development server
 * - Open a browser tab at http://localhost:8787/ to see your worker in action
 * - Run `npm run deploy` to publish your worker
 *
 * Bind resources to your worker in `wrangler.toml`. After adding bindings, a type definition for the
 * `Env` object can be regenerated with `npm run cf-typegen`.
 *
 * Learn more at https://developers.cloudflare.com/workers/
 */

export default {
	async fetch(request, env, ctx): Promise<Response> {
		return new Response('Hello World!');
	},
} satisfies ExportedHandler<Env>;
```

Workerのエントリーポイントとしてfetchが最初から定義されています。
外にリクエストとレスポンスを返却する関数を定義して、fetch内から呼び出し、その内容を表示するスクリプトに修正します。

```TypeScript
/**
 * Welcome to Cloudflare Workers! This is your first worker.
 *
 * - Run `npm run dev` in your terminal to start a development server
 * - Open a browser tab at http://localhost:8787/ to see your worker in action
 * - Run `npm run deploy` to publish your worker
 *
 * Bind resources to your worker in `wrangler.toml`. After adding bindings, a type definition for the
 * `Env` object can be regenerated with `npm run cf-typegen`.
 *
 * Learn more at https://developers.cloudflare.com/workers/
 */

const ACCESS_TOKEN = <ACCESS_TOKEN>; // 既存のアクセストークンをここに設定

async function fetchQiitaArticles(): Promise<any> {
    const url = 'https://qiita.com/api/v2/items';
    const headers = {
        'Authorization': `Bearer ${ACCESS_TOKEN}`,
        'Content-Type': 'application/json'
    };

    try {
        const response = await fetch(url, { headers });
        if (!response.ok) {
            throw new Error(`Error fetching articles: ${response.statusText}`);
        }
        const articles = await response.json();
        console.log(articles);
        return articles;
    } catch (error) {
        console.error('Failed to fetch Qiita articles:', error);
        return null;
    }
}

export default {
	async fetch(request, env, ctx): Promise<Response> {
		const result = await fetchQiitaArticles();

		if (result === null) {
			return new Response(JSON.stringify({ error: 'No data available' }), {
				status: 404,
				headers: { 'Content-Type': 'application/json' }
			});
		}

		return new Response(JSON.stringify(result), {
            headers: { 'Content-Type': 'application/json' }
        });
	},
} satisfies ExportedHandler<Env>;
```

リクエストに成功すると、こんな感じのレスポンスが表示されると思います。

![](/images/cloudflare-worker-kv-getstarted/image10.png)

以上でWorkerにQiitaAPIを使った使用例となるのですが、現在のコードだとアクセストークンがコード内にそのまま含まれていて、セキュリティ的によろしくないです。
そのためにKVを使う方法と環境変数を利用する方法を説明していきます。
次章では、そのKVの利用法を説明していきます。

## KV
KVとはKey-Valueのペアによるストレージサービスです。
ChatGPT（v4o）の説明では、次のようになります。

> Cloudflare Workers KV（Key-Value）は、Cloudflareが提供する分散型データストレージサービスです。主にCloudflare Workersから利用され、高速でスケーラブルなデータの保存・取得を可能にします。

また、KVのデータは非公開となっており、Cloudflare Workersからのみアクセス可能となっています。

### CLIからKVを作成する
それではKVを作成していきます。

#### 名前空間を作成する
KVに名前空間を作成していきます。ここでは名前空間の詳細な説明は省き、グループ名のような概念だと理解してください。詳細について、公式ページを参考ください。

```bash
wrangler kv:namespace create <namespace-name>
```

作成が成功したら次のように名前空間の情報が表示されます。
ただ、ここで注意する点があります。ローカルで使用する場合は、ローカル用のidを発行する必要があります。

その場合は--previewコマンドを追加して実行します。

```bash
wrangler kv:namespace create <namespace-name> --preview
```

これらの内容をwrangler.tomlに転記します。
すでに、wrangler.tomlには、名前空間を書くところがあり、コメントアウトされています。

```js
# Bind a KV Namespace. Use KV as persistent storage for small key-value pairs.
# Docs: https://developers.cloudflare.com/workers/wrangler/configuration/#kv-namespaces
# [[kv_namespaces]]
# binding = "MY_KV_NAMESPACE"
# id = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```
↓

```js
# Bind a KV Namespace. Use KV as persistent storage for small key-value pairs.
# Docs: https://developers.cloudflare.com/workers/wrangler/configuration/#kv-namespaces
[[kv_namespaces]]
binding = "qiita_token"
id = "080c5fa48d2d4fb38680e9c64f8602a8"
preview_id = "da104f7e90fa4b168e69cb2433f148bd"
```

#### KVに書き込む
次に作成したpreview idに書き込みます。

```bash
wrangler kv:key put <key> <value> --namespace-id <preview-id>
```
ここで、アクセストークンを書き込みます。

```bash
wrangler kv:key put accessToken "piyofuga" --namespace-id <preview-id>
```

書き込みが成功したら、成功のプロンプトがでてくるのですが、確認する場合は次のコマンドを実行します。

```bash
wrangler kv:key get <key> --namespace-id <preview-id>
```
今回のアクセストークンを確認する場合

```bash
wrangler kv:key get accessToken --namespace-id <access_token>
```

これで、設定したトークンが表示されればOKです。

これで開発環境でKVの作成ができました。
次に、このKVをコードで利用していきます。

### コードからKVを利用する。
コードからKVを取得する場合は、次のように書きます。

```typescript
env.NAME_SPACE.get('key');
```

ただ、TypeScriptではNameSpaceのインターフェースを定義する必要があります。
ここでのNameSpace名は`wrangler.toml`で設定したときのbindingと同じになる必要があります。

```ts
interface Env {
	qiita_token: KVNamespace;
}
```

以上を踏まえて、KVからアクセストークンを読み込むコードに修正します。

```ts
/**
 * Welcome to Cloudflare Workers! This is your first worker.
 *
 * - Run `npm run dev` in your terminal to start a development server
 * - Open a browser tab at http://localhost:8787/ to see your worker in action
 * - Run `npm run deploy` to publish your worker
 *
 * Bind resources to your worker in `wrangler.toml`. After adding bindings, a type definition for the
 * `Env` object can be regenerated with `npm run cf-typegen`.
 *
 * Learn more at https://developers.cloudflare.com/workers/
 */

interface Env {
	qiita_token: KVNamespace;
}

async function fetchQiitaArticles(token: string): Promise<any> {
    const url = 'https://qiita.com/api/v2/items';
    const headers = {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
    };

    try {
        const response = await fetch(url, { headers });
        if (!response.ok) {
            throw new Error(`Error fetching articles: ${response.statusText}`);
        }
        const articles = await response.json();
        console.log(articles);
        return articles;
    } catch (error) {
        console.error('Failed to fetch Qiita articles:', error);
        return null;
    }
}

export default {
	async fetch(request, env, ctx): Promise<Response> {
		const accessToken = await env.qiita_token.get('accessToken');
		if (accessToken == null) {
			return new Response(JSON.stringify({ error: 'Access token not found' }), {
				status: 401,
				headers: { 'Content-Type': 'application/json' }
			});
		}

		const result = await fetchQiitaArticles(accessToken);

		if (result === null) {
			return new Response(JSON.stringify({ error: 'No data available' }), {
				status: 404,
				headers: { 'Content-Type': 'application/json' }
			});
		}

		return new Response(JSON.stringify(result), {
            headers: { 'Content-Type': 'application/json' }
        });
	},
} satisfies ExportedHandler<Env>;

```

次にWorkerの起動ですが、KVを扱う場合は次のように`--remote`オプションをつけて実行します。
これによって、Cloudflareのリモート環境上でWorkerを実行し、KVを参照することが可能となります。

```bash
wrangler dev --remote
```

指定されたURLでブラウザを開くと、Qiita記事のレスポンスが帰ってきたらOKです。

## 環境変数の利用
それでは、アクセストークンの管理を環境変数で利用する場合の方法を説明していきます。
環境変数の場合、値が暗号化されるため、機密情報を扱うには適しています。
しかし、環境変数を扱う場合は、Worker内から書き込むことができないことに注意です。

### 環境変数の書き込み
環境変数をCLIで開発環境に書き込む場合は次のようになります。

```bash
wrangler secret put --env <environment> <KEY>
```
ここでは次のように実行します。

```bash
wrangler secret put token
```
次にValueの入力が求められるので、アクセストークンの値をコピペして入力します。

ちなみに、作成した環境変数をCLIで確認する方法はないようです。
ファイルで環境変数を開発環境のときのみに設定する場合は、プロジェクトのルートディレクトリ配下に`.dev.vars`というファイルで、KEY,VALUEで設定することも可能です。

### Workerのコードから環境変数を利用する。
KVの場合とほぼ同じで、Envのインターフェースを定義して、`env.KEY_NAME`で値を取得することができます。

```ts
interface Env {
	token: string;
}
・
・
・
const value = await env.KEY_NAME;
```

修正したコード全体は次のようになります。

```ts
interface Env {
	token: string;
}

async function fetchQiitaArticles(_token: string): Promise<any> {
    const url = 'https://qiita.com/api/v2/items';
    const headers = {
        'Authorization': `Bearer ${_token}`,
        'Content-Type': 'application/json'
    };

    try {
        const response = await fetch(url, { headers });
        if (!response.ok) {
            throw new Error(`Error fetching articles: ${response.statusText}`);
        }
        const articles = await response.json();
        console.log(articles);
        return articles;
    } catch (error) {
        console.error('Failed to fetch Qiita articles:', error);
        return null;
    }
}

export default {
	async fetch(request, env, ctx): Promise<Response> {
		const accessToken = await env.token;
		console.log(accessToken);
		if (accessToken == null) {
			return new Response(JSON.stringify({ error: 'Access token not found' }), {
				status: 401,
				headers: { 'Content-Type': 'application/json' }
			});
		}

		const result = await fetchQiitaArticles(accessToken);

		if (result === null) {
			return new Response(JSON.stringify({ error: 'No data available' }), {
				status: 404,
				headers: { 'Content-Type': 'application/json' }
			});
		}

		return new Response(JSON.stringify(result), {
            headers: { 'Content-Type': 'application/json' }
        });
	},
} satisfies ExportedHandler<Env>;
```

また、環境変数を利用する場合も`wrangler dev --remote`で起動してください。
成功すると、画面にQiita APIのレスポンスが表示されます。

![](/images/cloudflare-worker-kv-getstarted/image11.png)

# おわりに
以上のような形でローカルでのKV/環境変数を用いての簡単なAPIリクエストを返すWorkerの作成ができました。
あまりCloudflareについての日本語記事がなかったので、この記事が開発の助けになれば幸いです。


# 参考資料
* https://developers.cloudflare.com/workers/wrangler/install-and-update/
* https://qiita.com/api/v2/docs
* https://qiita.com/koki_develop/items/57f86a1abc332ed2185d