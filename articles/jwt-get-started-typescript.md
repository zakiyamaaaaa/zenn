---
title: "Typescriptで颯爽にJWT認証を実装する"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Typescript,node,jwt,javascript,express]
published: true
---

# はじめに
普段はモバイルアプリの開発をメインにしていますが、サーバーサイドの認証技術を深く理解するために、TypeScriptを使ってJWT認証を実装しました。この記事では、JWT認証そのものの詳細な説明は省略し、TypeScriptを用いた具体的な実装方法に焦点を当てて説明します。JWTについての基本的な説明は他の参考資料をご覧ください。

## 開発環境
* OS: macOS Sonoma 14.5
* Node.js: v20.9.0
* npm: v10.1.0

## 必要なパッケージをインストール
Express.jsとjwtのライブラリをインストールします。

```bash
npm install express jsonwebtoken
```
## サーバーサイドの実装
手順に沿って説明していきます。

### ライブラリのインストール
先ほどインストールしたexpress,jwt,そして環境変数を扱うために、dotenvライブラリをインストールします。

```ts
import express, { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';
import dotenv from 'dotenv';
```

### Private keyの設定
ここでは、簡易的に環境変数に設定したSecret keyを読み込む処理を実装します。
まず、プロジェクトルートディレクトリに、.envファイルを設定します。

```bash:.env
SECRET_KEY=your-secret-key
```

```ts
dotenv.config();
const SECRET_KEY = process.env.SECRET_KEY;

if (!SECRET_KEY) {
    throw new Error('SECRET_KEY is not defined');
}
```

### 認証ミドルウェアの実装
JWT認証の仕組みを実装するミドルウェアを作成します。このミドルウェアでは以下の手順を踏んで認証を行います。

1.	リクエストヘッダからAuthorizationを読み込み、トークンを抽出。
2.	トークンが存在しない場合は401 Unauthorizedを返却。
3.	トークンをSECRET_KEYを使って検証し、エラーがあれば403 Forbiddenを返却。
4.	認証が成功した場合、次のミドルウェアに処理を渡します。

```ts
const authenticateToken = (req: Request, res: Response, next: NextFunction) => {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1];
    if (token == null) {
        res.sendStatus(401);
        return;
    }

    jwt.verify(token, SECRET_KEY, (err, user) => {
        if (err) {
            res.sendStatus(403);
            return;
        }
        (req as any).user = user;
        // 認証成功後次のミドルウェアへ
        next();
    });
}
```

### エンドポイントの作成
認証が必要なエンドポイントと不要なエンドポイントを作成します。/loginエンドポイントではJWTトークンを発行し、/protectedエンドポイントではトークンの検証を行います。

```ts
// 認証不要なルート
app.get('/login', (req: Request, res: Response)=>{
    const username = req.query.username;
    const user = { name: username};
    const token = jwt.sign(user, SECRET_KEY, { expiresIn: '1h'});
    res.json({ token});
});

// 認証が必要なルート
app.get('/protected', authenticateToken, (req: Request, res: Response) => {
    res.json({ message: 'Authenticated'});
});

```

### サーバーの起動
最後に、アプリケーションがリクエストを受け付けるためのポートを設定します。

```ts
const app = express();
const PORT = 3000;

app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
});
```

最終的には、このようなコードとなります。

```ts:server.ts
import express, { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';
import dotenv from 'dotenv';

dotenv.config();
const SECRET_KEY = process.env.SECRET_KEY;

const app = express();
const PORT = 3000;

if (!SECRET_KEY) {
    throw new Error('SECRET_KEY is not defined');
}

// 認証ミドルウェア
const authenticateToken = (req: Request, res: Response, next: NextFunction) => {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1];
    if (token == null) {
        res.sendStatus(401);
        return;
    }

    jwt.verify(token, SECRET_KEY, (err, user) => {
        if (err) {
            res.sendStatus(403);
            return;
        }
        (req as any).user = user;
        // 認証成功後次のミドルウェアへ
        next();
    });
}

// 認証不要なルート
app.get('/login', (req: Request, res: Response)=>{
    const username = req.query.username;
    const user = { name: username};
    const token = jwt.sign(user, SECRET_KEY, { expiresIn: '1h'});
    res.json({ token});
});

// 認証が必要なルート
app.get('/protected', authenticateToken, (req: Request, res: Response) => {
    res.json({ message: 'Authenticated'});
});

app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
});
```

## 実行準備

それでは、実際に動かしていきます。

### Typescriptのインストール

まずTypescriptをグローバルにインストールします。

```bash
   npm install -g typescript
```

### TypeScript設定ファイルの作成
tsconfig.jsonファイルを作成し、以下のように設定します。

```json
   {
     "compilerOptions": {
       "target": "ES6",
       "module": "commonjs",
       "outDir": "./dist",
       "rootDir": "./",
       "strict": true,
       "esModuleInterop": true
     },
     "include": ["server.ts"]
   }
```

### ビルドの実行
次のコマンドでTypeScriptをコンパイルします。

```bash
tsc
```

tsconfig.jsonの設定に基づいて、server.tsがコンパイルされ、distディレクトリにJavaScriptファイルが生成されます。

### サーバーの起動
```bash
node dist/server.js
```

これで実行することができる準備が整いました。

## 実行テスト

### /loginエンドポイントをテスト
まずログイン認証を試してみます。
usernameには適当な名前を入れてください。

```bash
curl "http://localhost:3000/login?username=your-name"
```

成功していれば、次のようなレスポンスが返ってきます。
{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiemFraSIsImlhdCI6MTcyOTU0MDA3NywiZXhwIjoxNzI5NTQzNjc3fQ.UupfZCCj2ZDP1H9otgg-Rb8QM5Rjry4sGHlsnX5vclE"}%

### /protectedエンドポイントをテスト

ヘッダに先ほど取得したjwtトークンを使って、認証が必要なエンドポイントにアクセスします。

```bash
curl -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiemFraSIsImlhdCI6MTcyOTU0MDA3NywiZXhwIjoxNzI5NTQzNjc3fQ.UupfZCCj2ZDP1H9otgg-Rb8QM5Rjry4sGHlsnX5vclE' http://localhost:3000/protected
```

成功していれば、次のレスポンスが返ります。
```json
{"message":"Authenticated"}
```

# おわりに
以上TypescriptでJWT認証を作っていきました。
あまりフロント側では認証の仕組みをしらなくてもなんとかなることが多かったのですが、最近ではパスキーなどセキュアな認証方法が広がっているので、こうした認証について実際に手を動かすことで理解が深まりました。