---
title: "サーバーレスWebAPI開発に入門しようとしたけどうまくいかなかった話"
date: 2021-12-30T14:25:09+09:00
draft: false
author: ["lockedgirl"]
tags: ["AWS", "Golang", "Docker"]
categories: ["Tutorial"]
archives: ["2021", "2021-12"]
description: "AWS DAVの勉強をしていたら、Lambdaを使えばとても簡単にWebAPIが作れてしまうのでは？と感じたので、 ググって最初にヒットしたこちらの記事を参考に、サーバーレスWebAPIを作ろうとしたらうまくいかなかった話です。 "
url: "/entry/2021/12/30/142509"
---

AWS DAV の勉強をしていたら、Lambda を使えばとても簡単に WebAPI が作れてしまうのでは？と感じたので、ググって最初にヒットしたこちらの記事を参考に、サーバーレス WebAPI を作ろうとしたらうまくいかなかった話です。

なお、Terraform を使ったインフラ構築は特に詰まることも無かったので本記事では触れません。

{{< blog-card "https://future-architect.github.io/articles/20200927/" >}}

そもそも Windows の開発環境で無理やりやっている事が原因だと考えていますが、
Dockerfile の書き方とか色々調べてて勉強になったので書き残しておきます。  
記事の内容が間違っているぞということが主旨ではないです。

### 開発環境

- Windows 10 Home(20H2)
- WSL2
- Docker Desktop
- Visual Studio Code
  - Remote - Containers 拡張

読みにくいかもしれませんが、以下にやったことを時系列順に書いていきます。

### Docker コンテナ上に Golang 開発環境を作る

最初は Windows 上で作業していたのですが、Makefile 中のコマンドを
Windows のコマンドに置き換える事がうまくできなかったため、
Docker コンテナ上に Golang 開発環境を作ることにしました。

こちらの記事を参考に、`Dockerfile`と`docker-compose.yml`を作成。

{{< blog-card "https://cpp-learning.com/go-docker/" >}}

#### フォルダ構成

```powershell
.
├build
│└─Dockerfile
├cmd
├gen
├testdata
└docker-compose.yml
```

#### Dockerfile

```yaml
# goバージョン
FROM golang:1.17.5-alpine

# 前提パッケージのインストール
RUN apk add --update && apk --no-cache add git curl zip unzip sudo binutils make alpine-sdk build-base

ENV GLIBC_VER=2.34-r0

# install glibc
RUN curl -sL https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub -o /etc/apk/keys/sgerrand.rsa.pub
RUN curl -sLO https://github.com/sgerrand/alpine-pkg-glibc/releases/download/${GLIBC_VER}/glibc-${GLIBC_VER}.apk
RUN curl -sLO https://github.com/sgerrand/alpine-pkg-glibc/releases/download/${GLIBC_VER}/glibc-bin-${GLIBC_VER}.apk
RUN apk add --no-cache glibc-${GLIBC_VER}.apk glibc-bin-${GLIBC_VER}.apk

# aws cli v2 のインストール
# https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/install-cliv2-linux.html
RUN curl -sL "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
RUN unzip -q awscliv2.zip
RUN sudo ./aws/install

RUN go get -u github.com/go-swagger/go-swagger/cmd/swagger
```

コンテナは軽量なほうがいいだろうと、ベースイメージには Alpine Linux を選択。  
後々 Golang をビルドする時に『stdlib.h が足りない』と怒られるため、`alpine-sdk`と`build-base`をインストール。  
`go get`の裏では git が動いているようなので、`git`も前提パッケージとなります。  
また、AWS CLI をインストールしたいが、Alpine Linux ではそのまま
`sudo ./aws/install`してもうまくいかないため`glibc`をインストールした上で AWS CLI をインストールしてます。  
apk コマンドでインストール可能なパッケージはこちらのサイトから検索できます。

[Alpine Linux packages](https://pkgs.alpinelinux.org/contents)

#### docker-compose.yml

```yaml
version: "3"
services:
  aws-sdk-go-container:
    build: # ビルドに使うDockerファイルのパス
      context: .
      dockerfile: ./build/Dockerfile
    container_name: aws-sdk-go-container
    volumes: # ホストのAWS認証情報を共有する
      - "~/.aws:/root/.aws"

  # LocalStack
  localstack:
    image: localstack/localstack:latest
    environment:
      - SERVICE=dynamodb
      - DEFAULT_REGION=ap-northeast-1 # リージョンを設定
    ports:
      - 4566:4566 # サービスへのアクセスポートは4566
```

`volumes`ではホスト側の`C:\Users\<ユーザ名>\.aws`に置かれている AWS 認証情報を、
コンテナから探しにいけるようにしています。  
また元記事では`docker run`コマンドで LocalStack を起動していますが、
せっかく開発環境をコンテナ上に構築しているので、
あわせて起動するよう`docker-compose.yml`に記述します。  
`user_handler_test.go`の dbEndpoint を`http://localstack:4566`に書き換える事で
開発環境のコンテナから LocalStack のコンテナにアクセスできます。

### curl コマンドを叩くと 502 が返ってきてお手上げに

コンテナ上で`make test`がうまくいったため、`make deploy`して AWS にデプロイしました。  
デプロイも成功したため、元記事の通り以下のコマンドを実行したところ、
502 InternalServerErrorException が返ってくるように。

```bash
curl -i https://${rest-api-id}.execute-api.ap-northeast-1.amazonaws.com/test/v1/users
```

AWS マネジメントコンソールから API Gateway の CloudWatch ログを有効化し、
ログを見てみると`no such file or directory: PathError`とのこと。

[alpine のイメージで AWS Lambda 用の go バイナリをビルドしたら PathError になった話 - Qiita](https://qiita.com/astronoka/items/e8d72cef1ad05faec76c)

ググって出てきたこちらの記事を参考に、Makefile 内の`go build`コマンドのオプションを書き換えて再デプロイ。  
エラーメッセージが`string producer has not yet been implemented: apiError`に変わったけれど、
Swagger がよしなにやってくれている部分でエラーが出ているようで、写経しているレベルの初学者にはお手上げになりました。

### WSL2 がホストのメモリを浪費する現象の対策

502 エラーに四苦八苦している時、PC のファンの音が大きくなり、ビルドに時間がかかるようになりました。  
タスクマネージャーを開いてみると CPU、メモリ、ディスクが 100%近く張り付いています。  
調べてみると`Vmmem`というプロセスがメモリを浪費していました。

[WSL2 によるホストのメモリ枯渇を防ぐための暫定対処 - Qiita](https://qiita.com/yoichiwo7/items/e3e13b6fe2f32c4c6120)

ググってみたところ、WSL2 の不具合のようでした。  
こちらの記事を参考に暫定対処を行いましたが、メモリスワップが
発生すると SSD の I/O が 100%になるのは怖いですね。

### 参考

#### 開発環境系

- [docker で go 開発環境構築](https://zenn.dev/akakuro/articles/2426098256785b)

- [【Go 言語（Golang）】WSL2 と Docker で最高の Go 開発環境をつくる｜はやぶさの技術ノート](https://cpp-learning.com/go-docker/)

- [AWS CLI v2 を docker で使えるようにする](https://zenn.dev/tokku5552/articles/aws-container)

- [Alpine ベースのコンテナイメージで AWS CLI v2 を使う - 理系学生日記](https://kiririmode.hatenablog.jp/entry/20200725/1595621558)

- [【Docker】docker-compose で localstack のコンテナを立てる | IT 土方の奮闘記](https://it-blue-collar-dairy.com/localstack_on_docker-compose/)

#### Alpine Linux 系

- [Alpine Linux の apk コマンドでインストール可能なパッケージを検索する方法 | ゲンゾウ用ポストイット](https://genzouw.com/entry/2019/11/28/173400/1795/)

- [【Golang】Alpine Docker で実行すると error: stdlib.h: No such file or directory. "stdlib.h" が足りないと言われる - Qiita](https://qiita.com/KEINOS/items/fd6a299961e3b8f3864f)

- [alpine のイメージで AWS Lambda 用の go バイナリをビルドしたら PathError になった話 - Qiita](https://qiita.com/astronoka/items/e8d72cef1ad05faec76c)

#### メモリ浪費の暫定対処系

- [WSL2 によるホストのメモリ枯渇を防ぐための暫定対処 - Qiita](https://qiita.com/yoichiwo7/items/e3e13b6fe2f32c4c6120)

- [WSL1/WSL2 を再起動する方法 - 備忘録](https://kagasu.hatenablog.com/entry/2020/01/02/155532)
