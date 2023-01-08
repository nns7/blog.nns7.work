---
title: "PlayFrameworkをIntelliJ IDEA CommunityにインポートしてHelloWorld"
date: 2018-12-27T23:11:22+09:00
draft: false
author: ["lockedgirl"]
tags: ["Java", "PlayFramework"]
categories: ["Tutorial"]
archives: ["2018", "2018-12"]
description: "昔ちょっと触ったPlay Frameworkを使ってアプリを作ってみようと思ったけど、 やり方が昔と変わっててググっても最新の情報がいい感じにまとまってるところが無かったので、 備忘録的にメモしつつ始めてみた。"
url: "/entry/2018/12/27/231122"
---

昔ちょっと触った Play Framework を使ってアプリを作ってみようと思ったけど、やり方が昔と変わっててググっても最新の情報がいい感じにまとまってるところが無かったので、備忘録的にメモしつつ始めてみた。

### 筆者の環境

Windows 10 Home（64bit）

### インストールするもの

- JDK 8（update 192）[^footnote]
- IntelliJ IDEA（version.2018.3.2）
- sbt（1.2.7）

### JDK 8 のインストール

1. Oracle のサイトから[Java SE のダウンロードページ](https://www.oracle.com/technetwork/java/javase/downloads/index.html)へアクセス。
2. Java SE 8 の JDK Download ボタンをクリック。
3. Java SE Development Kit 8u192 と書いてある場所の Accept License Agreement にチェックを入れる。
4. Product / File Description の列に Windows x64 と書いてある行の Download リンクをクリック。
5. ダウンロードが完了したらインストーラーを起動。画面の指示に従ってインストール。
6. 途中、ライセンス条項の変更ウィンドウが表示されたら OK ボタンをクリック。
7. コピー先フォルダウィンドウが表示される。変更ボタンから任意のインストール先を選択し、次ボタンをクリック。
8. 正常にインストールされましたと表示されたら閉じるボタンをクリック。
9. システム環境変数に JAVA_HOME を追加。筆者の場合は`C:\Program Files\Java\jdk1.8.0_192`とした。
10. システム環境変数から Path を選択して設定を追加。`%JAVA_HOME%\bin`を追加。
11. インストールが成功しているか確認するためにコマンドプロンプトを起動。以下のコマンドを入力して実行する。

```powershell
> java -version
```

次のように表示されたら成功です。

```powershell
java version "1.8.0_192"
Java(TM) SE Runtime Environment (build 1.8.0_192-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.192-b12, mixed mode)
```

java コンパイラにもパスが通っているか確認する。

```powershell
> javac -version

javac 1.8.0_192
```

### IntelliJ IDEA のインストール

1. Jetbrains のサイトから[IntelliJ IDEA](https://www.jetbrains.com/idea/)のページへアクセス
2. 中央の Download ボタンをクリックするとダウンロードページへ。
   　 Ultimate 版か Community 版かを選ぶページへ移動するので、今回は Community 版を選択。
3. Download ボタンの横から exe か zip か選べる。特に拘りも無いので exe を選んで Download ボタンをクリック。
4. ダウンロードが完了したらインストーラーを起動。画面の指示に従ってインストール。
   （パスにブランクが入るのがなんか嫌だったので、インストール先を Program Files から変更した）
5. インストールが完了したら早速起動してみる。
6. Complete Installation ウィンドウが表示されるので、「Do not import settings」を選択して OK ボタンをクリック。
   （他の開発環境で使ってた設定をインポートできるっぽい？）
7. Jetbrains Privacy Policy ウィンドウが表示されるので、Agreement にチェックを入れて Continue ボタンをクリック。
8. Data Sharing ウィンドウが表示される。お好みで Send Usage Stattistics ボタンか Don't send ボタンをクリック。
9. Set UI theme ウィンドウが表示されるので好みの UI を選択して Next: Default plugins ボタンをクリック。
10. Default plugins ウィンドウが表示される。必要なプラグインがあればここで選択。自分の場合はよくわからなかったのでそのまま Next: Featured plugins ボタンをクリックした。
11. Featured plugins ウィンドウが表示される。Play Framework の開発には Scala プラグインが必要みたいなので、このウィンドウで Scala の Install ボタンをクリック。
    　プログレスバーの表示が Installed になったら Start using IntelliJ IDEA ボタンをクリック。

### sbt のインストール

1. [ダウンロードページ](https://www.scala-sbt.org/download.html)へアクセス
2. SBT-1.2.7.MSI ボタンをクリック
3. ダウンロードが完了したらインストーラーを起動。画面の指示に従ってインストール。
4. インストールが完了したら Finish ボタンをクリック。
5. インストールが成功しているか確認するためにコマンドプロンプトを起動。以下のコマンドを入力して実行する。

```powershell
> cd <sbtがカレントディレクトリ直下にtargetフォルダを作成してしまうので適当なフォルダを作成しておく>
> mkdir project
> sbt console
```

次のように表示されたら成功です。

```powershell
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=256m; support was removed in 8.0
[info] Updated file d:\develop\sbt\project\build.properties: set sbt.version to 1.2.7
[info] Loading project definition from D:\develop\sbt\project
[info] Updating ProjectRef(uri("file:/D:/develop/sbt/project/"), "sbt-build")...
[info] Done updating.
[info] Set current project to sbt (in build file:/D:/develop/sbt/)
[info] Updating ...
[info] Done updating.
[info] Starting scala interpreter...
Welcome to Scala 2.12.7 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_192).
Type in expressions for evaluation. Or try :help.
```

6.  次のコマンドを入力して sbt コンソールを終了します。

```powershell
scala> :quit
```

### プロジェクトの作成

1. コマンドプロンプトを起動し、プロジェクトを作成する適当なディレクトリに移動。

```powershell
> cd <プロジェクトを配置するディレクトリ>
```

2. 新規プロジェクトを作成。Java で作りたかったので seed は Java を選択。

```powershell
> sbt new playframework/play-java-seed.g8
This template generates a Play Java project
name [play-java-seed]:
```

プロジェクト名を聞かれるので「helloworld」と入力。organization は何も入力せずデフォルトのままとした。

```powershell
name [play-java-seed]: helloworld
organization [com.example]:

Template applied in D:\develop\play\.\helloworld
```

### 作成したプロジェクトを IntelliJ IDEA にインポート

1. IntelliJ IDEA を起動。Import Project を選択。
2. インポートするプロジェクトのルートディレクトリを選択。（先程作成した helloworld プロジェクトを選択する）
3. インポートするプロジェクトの種類を聞かれるので、[Import porject from external model]にチェックを入れ sbt を選択して Next ボタンをクリック。
4. Import Project ウィンドウが表示される。Porject JDK の New...ボタンをクリックして JDK を選択。
5. ディレクトリを選択するウィンドウが表示される。最初にインストールした JDK 8 のインストール先を選択して Project JDK が変更されたら Finish ボタンをクリック。
6. Tip of the Day ウィンドウが表示されたら[Show tips on startup]のチェックを外して Close ボタンをクリック。
7. dump project structure from sbt やら import to intellij project model やら表示されるので終わるまで待つ。（結構長い。15 分くらい）

### Hello world してみる

1. IntelliJ のターミナルを開き、`sbt run`と入力する。
2. ブラウザから「[http://localhost:9000](http://localhost:9000)」にアクセスしてみる。
3. Welcome to Play!と表示されたら成功です。お疲れ様でした。

[^footnote]: 執筆時点での最新の Oracle JDK は 11 だが有償サポートがどうの Open JDK がどうのでよくわからないので、とりあえず Play Framework が必須としている JDK 8 をインストールすることにした。
