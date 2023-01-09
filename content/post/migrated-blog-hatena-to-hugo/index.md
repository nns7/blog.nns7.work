---
title: "はてなブログからHugoに移行した"
description: "移行方法や移行に際してカスタマイズした部分のメモ。"
slug: Migrated Blog Hatena to Hugo
date: 2023-01-08T21:07:55+09:00
categories: ["Technology"]
tags: ["Hugo"]
image:
math: false
license:
hidden: false
comments: false
draft: false
---

## はじめに

ブログをはてなブログから Hugo に移行してみた。

折角 AWS 認定資格を取得したので、インフラを AWS 上に構築して Web サイトか Web アプリを公開したいな、というのが始まり。  
しかし作りたいアプリは無いので Web サイトの公開かなーとか、そしたらはてなに不満は無いけどブログを移行しようかなーとか、書きかけの記事を GitHub で管理できたらブログの更新頻度が上がるかなー、などなど考えてこうなった。

Web サイト公開までの手順や Terraform のコードなどは別の記事で解説する予定。  
Terraform のコードは GitHub で公開しているので、同じ構成でブログを公開したい方の参考になれば。

{{< blog-card "https://github.com/nns7/blog.nns7.work-terraform" >}}

## 構成

- AWS 上のリソースは Terraform で構築
- 静的サイトジェネレーター Hugo を使用してコンテンツを生成
- 静的サイトジェネレーターのソースコードの管理（ブログ記事の管理）は GitHub
- GitHub Actions を構成して、記事をコミットすると GitHub 側でビルドして S3 に自動で配置
- S3 のコンテンツを CloudFront でキャッシュ＆デリバリ

### 静的サイトジェネレーターとは

静的サイトジェネレーター（Static Site Generator）。  
WordPress やはてなブログのような一般的なブログサービスは、ブラウザからのリクエストを受けた Web サーバーが、都度プログラムを実行し動的に生成した Web ページをレスポンスで返却している。  
そのため、以下のような課題がある

- サーバー側にプログラムを実行するランタイムを用意する必要がある
- 生成する Web ページのもととなるデータを管理するためのデータベースが必要だったりする
- リクエストを受けた後に Web ページを生成するためレスポンスに時間がかかる
- Web ページが動的に変化するため、キャッシュが効かない

このような課題に対応するため、あらかじめ Web ページを生成しておき、静的なサイトとしてサーバーに置いておけばいいじゃん、というのが静的サイトジェネレーターの考え方。

### Hugo

[Hugo](https://gohugo.io/)  
Go 言語で書かれた静的サイトジェネレーターの一種。  
後述するブログ用のテーマが気に入ったので Hugo にした。

## Hugo のカスタマイズ

### はてなブログの過去記事の移行

{{< blog-card "https://michimani.net/post/development-transition-hatena-blog-to-hugo/">}}

Hugo に移行するにあたり、はてなブログ時代の記事を引き継ぎたいなと考えた。  
色々調べた結果、いくつかのツールが見つかったものの、最終的にこちらのツールを利用させて頂いた。  
はてな ID とブログ ID が異なる場合、`main.go`の以下の部分を書き換える必要があるかも。

```go {name="main.go"}
initialAtomLink = fmt.Sprintf("https://blog.hatena.ne.jp/%s/%s.hateblo.jp/atom/entry", hatenaId, <ブログIDに書き換え>)
```

### Hugo テーマ「Stack」

Hugo のカスタムテーマには[Stack](https://stack.jimmycai.com/)を使用した。

{{< blog-card "https://nakureis-notes.netlify.app/post/customize-stack-theme/">}}

こちらの記事を参考に、いくつかのカスタマイズを行った。  
カスタマイズのやり方はリンク先に掲載されているため割愛。

- コメント機能の無効化
- フォントの変更
- フッターの変更
- コードブロックの設定
- コードブロックのカスタマイズ
- default.md のアレンジ

### ショートコードの追加

ブログカードや Amazon の商品リンクなど、はてなでは URL を貼り付けると見た目をいい感じにしてくれる機能がある。  
Hugo では記事を Markdown で記述するため、複雑な HTML を記事に埋め込むショートコードという仕組みがある。  
はてなブログからの移行にあたり、いくつかのショートコードを追加した。

#### ブログカード

{{< blog-card "https://hugo-theme-salt.okdyy75.com/article/salt/blog-card/">}}

Hugo テーマ「salt」で実装済みだそうで、MIT ライセンスだったので利用させて頂いた。
`config.yaml`に以下のパラメーターを追加する必要がある。

```yaml {name="config.yaml"}
params:
  dafaultNoimage: image/default_noimage.png
  imageQuality: 75
```

#### Amazon の商品リンク

{{< blog-card "https://github.com/ikemo3/hugo-amazon-jp">}}

こちらのショートコードを追加した。

## おわりに

Hugo 自体のカスタマイズやショートコードの追加に結構な時間がかかってしまった。  
手間をかけずリッチなブログを作れるはてなブログは凄いなと改めて思った。

## 参考

### Hugo カスタマイズ

- [The world’s fastest framework for building websites | Hugo](https://gohugo.io/)
- [Stack | Card-style theme designed for bloggers](https://stack.jimmycai.com/)
- [Stack テーマのカスタマイズ](https://nakureis-notes.netlify.app/post/customize-stack-theme/)

### はてなブログからの移行

- [はてなブログの記事を Hugo に移行するツールを作ってみた](https://michimani.net/post/development-transition-hatena-blog-to-hugo/)
- [DARK MATTER をはてなブログから Hugo に移行した話](https://io.cyberdefense.jp/entry/darkmatter_transformation/)

### ショートコード

- [Hugo でついに外部 URL のブログカードを作れるようになった【自作ショートコード】](https://hugo-theme-salt.okdyy75.com/article/salt/blog-card/)
- [hugo-amazon-jp](https://github.com/ikemo3/hugo-amazon-jp)
- [hugo-amazon-jp 向けのショートコード等を出力するブックマークレット作った](https://yoshikiito.net/blog/archives/hugo-amazon-bookmarklet/)
