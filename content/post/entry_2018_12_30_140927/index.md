---
title: "PlayFrameworkのチュートリアル（挫折版）"
date: 2018-12-30T14:09:27+09:00
draft: false
author: ["lockedgirl"]
tags: ["Java", "PlayFramework"]
categories: ["Tutorial"]
archives: ["2018", "2018-12"]
description: "前回の記事の続きです。"
url: "/entry/2018/12/30/140927"
---

[前回の記事の続きです。](/entry/2018/12/27/231122)

HelloWorld を表示したので、次はチュートリアルに沿ってサンプルアプリケーションを組んでみたいところですが、[公式の Play 2.4 のドキュメントには英語のチュートリアルしか無い](https://www.playframework.com/documentation/ja/2.4.x/Tutorials)ようなので、[Play 2.2 の時の日本語チュートリアルを参考に](https://www.playframework.com/documentation/ja/2.2.x/ScalaTodoList)TODO 管理アプリケーションを組んでいきたいと思います。  
…って思ってたけど、Task モデルの準備のところで Java の書き方がわからず挫折しました！  
誰かの参考になるかもなので途中まで書いたものは残しておきます。。。

### 概要

`conf/rotes`ファイルを開きますが、書き方がチュートリアルとちょっと変わっています。

```text
GET     /                           controllers.HomeController.index
```

が、意味は一緒です。`app/contorollers/HomeController.java`を開いてみます。

```java
package controllers;

import play.mvc.*;

public class HomeController extends Controller {

    public Result index() {
        return ok(views.html.index.render());
    }

}
```

どうやら`index()`メソッドが動いて、HTML コンテンツを含む 200 OK レスポンスを返しているようです。  
HTML コンテンツとのマッピングは`views.html.(HTMLのファイル名).render()`という書き方になるようです。  
次は`app/views/index.scala.html`を開いてみます。

```html
@() @main("Welcome to Play") {
<h1>Welcome to Play!</h1>
}
```

ここはチュートリアルと違ってべた書きしてあります。

### 開発フロー

チュートリアルのように controllers から views に値を渡したい場合、どのように書けばよいでしょうか。  
まず`app/views/index.scala.html`を開いて次のように書き換えます。

```html
@(message: String) @main("Welcome to Play") {
<h1>@message</h1>
}
```

次に`app/contorollers/HomeController.java`を開いて次のように書き換えます。

```java
package controllers;

import play.mvc.*;

public class HomeController extends Controller {

    public Result index() {
        return ok(views.html.index.render("Hello World"));
    }

}
```

書き換えたらブラウザから[localhost:9000](http://localhost:9000/)を開いてみます。  
Hello World と表示されるはずです。  
views の一行目に変数を宣言すれば、controllers から引数を渡せることがわかりました。

### アプリケーションの準備

`conf/rotes`ファイルを編集します。以下の 3 行を追加します。

```text
# Tasks
GET     /tasks                      controllers.HomeController.tasks
POST    /tasks                      controllers.HomeController.newTask
POST    /tasks/:id/delete           controllers.HomeController.deleteTask(id: Long)
```

ブラウザをリロードするとチュートリアルのようにコンパイルエラーが発生する事が確認できます。
次に`app/contorollers/HomeController.java`に以下の行を追加します。

```java
public Result tasks() {
    return TODO;
}

public Result newTask() {
    return TODO;
}

public Result deleteTask(Long id) {
    return TODO;
}
```

ブラウザから[localhost:9090/tasks](http://localhost:9000/tasks)を開くと TODO が表示されます。  
チュートリアルと同じように`index`アクションが呼び出された時に、`tasks`アクションにリダイレクトするようにします。

```java
public Result index() {
    return redirect(routes.HomeController.tasks());
}
```

### Task モデルの準備（ここで挫折）

Java で書く場合は Model クラスを継承した Task クラスを`models/Task.java`として実装すれば良さそうでしたが、なんのパッケージの Model クラスを import すればよいかわからず手詰まりになりました。

### 感想

ググると日本語の情報はわりとすぐ出てくるのですが、Play のバージョンが 2.2 の時と 2.4 の時と 2.6 の時で若干やり方が違うようで、いい感じに読み替えたり英語のドキュメントにあたるスキルが必要そうだと感じました。
作ってみたい Web アプリケーションはあるので、日本語のチュートリアルが充実している（と思われる）Ruby on Rails に挑戦してみたいと思います。
