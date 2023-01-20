---
title: "Express+ReactでSSRしようとして静的ファイルの読み込みに四苦八苦した話"
description: "express.staticミドルウェア関数が、Expressアプリケーションを起動するディレクトリからの相対パスとなることに気付かずにハマった。"
slug: Express Static Load
date: 2023-01-19T22:20:28+09:00
categories: ["Technology"]
tags: ["Webpack", "Express", "React", "TypeScript"]
image:
math: false
license:
hidden: false
comments: false
draft: false
---

## はじめに

とある Web サイトで挑戦中の技術課題に「Next.js を使わず React で SSR しよう」というものがあり、 Express アプリケーションで SSR した React アプリケーションから静的ファイルを読み込もうとしたのだけど、なかなかうまくいかず四苦八苦した。

## 結論

静的ファイルを Express アプリケーションでサーブする時に使う`express.static`という関数がある。  
この関数に静的ファイルまでの相対パスを指定するとき、実行する Express アプリケーションからの相対パスではなく、Express アプリケーションを起動するディレクトリからの相対パスとなる。  
Express アプリケーションを`package.json`の script から起動するときは注意しよう。

## 構成

### ライブラリ

| ライブラリ | バージョン |
| ---------- | ---------- |
| Node.js    | 19.4.0     |
| React      | 18.2.0     |
| express    | 4.18.2     |

### ディレクトリ

```powershell
.
├── dist/ # ビルドしたファイルの出力先
│ ├── client.js # Reactアプリケーションをビルドしたファイル
│ └── server.js # Expressアプリケーションをビルドしたファイル
├── node_modules
├── src/
│ ├── client.tsx # Reactアプリケーション本体
│ ├── index.tsx # Reactアプリケーションをhydrateするファイル
│ └── server.tsx # Expressアプリケーションの実装
├── .gitignore
├── package-lock.json
├── package.json
├── tsconfig.json
└── webpack.config.ts
```

## ハマったポイント

Express アプリケーションで SSR した React アプリケーションをクライアントサイドで hydrate した時に、useState hook で state を更新できなかった。
Express アプリケーションの当初の実装は以下の通り。

```ts{name="server.tsx",linenos=table,hl_lines=8}
import express from "express";
import Client from "./client";
import { renderToString } from "react-dom/server";

const app = express();
const port = 3000;

app.use(express.static("./"));

app.listen(port, () => {
  console.log(`listening on port ${port}`);
});

const reactComponent = renderToString(<Client />);

const pageHtml = `
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title>counter</title>
        <meta name="viewport" content="width=device-width, initial-scale=1" />
    </head>
    <body>
        <div id="react-root">${reactComponent}</div>
        <script defer="defer" src="client.js"></script>
    </body>
</html>
`;

app.get("/", (_req, res) => {
  res.send(pageHtml);
});
```

`server.tsx`をビルドして`dist/server.js`を出力していたため、8 行目を`express.static("./")`とすれば、同じディレクトリに出力した`dist/client.js`が読み込める想定だった。  
Webpack を利用するのが初めてだったため、ビルドの設定に不備があるのかと Webpack 周りをひたすらググった後に、`<script defer="defer" src="client.js"></script>`でサーブしている`client.js`が 404 になっていることに気付いた。

```json{name="package.json"}
{
  // 省略
  "scripts": {
    "dev": "webpack && node dist/server.js"
  },
  // 省略
}
```

上記はプロジェクトルートにある`package.json`の抜粋。  
原因はプロジェクトルートにある`package.json`の script から`npm run dev`コマンドで Express アプリケーションを起動していたためだった。  
`server.tsx`の 8 行目を`express.static("dist")`に修正したところ`dist/client.js`が読み込まれ、state の更新ができるようになった。

```ts{name="server.tsx",linenos=table,linenostart=6,hl_lines=3}
const port = 3000;

app.use(express.static("dist"));

app.listen(port, () => {
```

## サンプル

以下の GitHub リポジトリに本記事で使用したサンプルコードを公開している。

{{< blog-card "https://github.com/nns7/express-ssr-example">}}

## 参考

- [Express の静的ファイル提供（express.static）が通らなくて、つまづいた話](https://teno-hira.com/media/?p=1621)
