---
title: "サーバーレスWebAPI開発に入門できました"
date: 2022-01-03T01:20:24+09:00
draft: false
author: ["lockedgirl"]
tags: ["AWS", "Golang"]
categories: ["Tutorial"]
archives: ["2022", "2022-01"]
description: "前回の記事の続きです。 go-swaggerを使うのをやめて、Ginというフレームワークを使う事で無事入門できました。 "
url: "/entry/2022/01/03/012024"
---

[前回の記事の続きです。](/entry/2021/12/30/142509)  
go-swagger を使うのをやめて、Gin というフレームワークを使う事で無事入門できました。

Gin フレームワークの使い方はこちらの記事を参考にさせて頂きました。

{{< blog-card "https://selfnote.work/20211120/programming/golang-gin-api-2/" >}}

ソースコードを公開してます。参考になれば幸いです。

[GitHub - nns7/serverless-api-go-tutorial](https://github.com/nns7/serverless-api-go-tutorial)

### 開発環境

- Windows 10 Home(20H2)
- WSL2
- Docker Desktop
- Visual Studio Code
  - Remote - Containers 拡張

### DynamoDB Local をローカルにデプロイする

前回の記事ではローカルでの DynamoDB のテストに LocalStack を使っていましたが、調べたところ AWS が DynamoDB Local というものを用意してくれているので、そちらを使うようにします。
[コンピュータ上で DynamoDB をローカルでデプロイする - Amazon DynamoDB](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/DynamoDBLocal.DownloadingAndRunning.html)

公式に docker-compose が解説されているため、そちらを参考に以下のように記述しました。  
エンドポイントを`http://dynamodb-local:8000`とすればアクセスできます。

#### docker-compose.yml

```yaml
version: "3"
services:
  aws-sdk-go-containter:
    build:
      context: .
      dockerfile: ./build/Dockerfile
    container_name: aws-sdk-go-containter
    depends_on:
      - dynamodb-local
    volumes:
      - "~/.aws:/root/.aws"

  # DynamoDB Local
  dynamodb-local:
    command: "-jar DynamoDBLocal.jar -sharedDb -optimizeDbBeforeStartup  -dbPath ./data"
    image: "amazon/dynamodb-local:latest"
    container_name: dynamodb-local
    ports:
      - "8000:8000"
    volumes:
      - "./docker/dynamodb:/home/dynamodblocal/data"
    working_dir: /home/dynamodblocal
```

### API サーバーをローカルに立てる

公開しているソースコードは Lambda にデプロイするためのものですが、`main.go`と`db.go`を以下のように書き換えることで、API サーバーとしてローカルで実行できるようになります。

#### main.go

```go
package main

import (
    "github.com/nns7/userapp"

    "github.com/gin-gonic/gin"
)

func main() {
    router := gin.Default()
    // ユーザー作成
    router.POST("/users", userapp.PostUsers)
    // ユーザー一覧の取得
    router.GET("/users", userapp.GetUsers)
    // ユーザーの更新
    router.PUT("/users/:user_id", userapp.PutUser)
    // ユーザーの削除
    router.DELETE("/users/:user_id", userapp.DeleteUser)
    // ユーザーの検索
    router.GET("/users/search", userapp.SearchUser)
    router.Run()
}
```

#### db.go

```go
(省略)

func init() {
    usersTable = "local_users"
    if usersTable == "" {
        log.Fatal("missing env variable: DYNAMO_TABLE_USERS")
    }

    dbEndpoint := "http://dynamodb-local:8000"
    sess := session.Must(session.NewSessionWithOptions(session.Options{
        Profile:           "local",
        SharedConfigState: session.SharedConfigEnable,
        Config: aws.Config{
            Endpoint:   aws.String(dbEndpoint),
            DisableSSL: aws.Bool(true),
        },
    }))
    gdb = dynamo.New(sess)
}
```

書き換えたら Terminal 上で以下のコマンドを実行してください。DynamoDB Local にはテーブルが存在しないため、合わせてテーブルを作成します。

```bash
$ aws dynamodb create-table --table-name 'local_users' --endpoint-url "http://dynamodb-local:8000" \
--attribute-definitions '[{"AttributeName":"user_id","AttributeType": "S"}]' \
--key-schema '[{"AttributeName":"user_id","KeyType": "HASH"}]' \
--provisioned-throughput '{"ReadCapacityUnits": 5,"WriteCapacityUnits": 5}'
$ go run cmd/lambda/main.go
```

### API サーバーにリクエストを投げてみる

Chrome 拡張の『[Talend](https://chrome.google.com/webstore/detail/talend-api-tester-free-ed/aejoelaoggembcahagimdiliamlcdmfm?hl=ja)』を使って HTTP リクエストを投げていきます。

#### ユーザーの登録

- URL:<http://localhost:8080/users>
- METHOD:POST
- BODY:↓

```json
{
  "user_id": "001",
  "name": "gopher"
}
```

Response に`200 OK`と表示されたら成功です。

#### ユーザーの一覧

- URL:<http://localhost:8080/users>
- METHOD:GET
- Response↓

```json
[
  {
    "user_id": "001",
    "name": "gopher"
  }
]
```

さきほど登録したユーザーが取得できましたね。  
別のユーザーをもう一つ登録してみて、GET メソッドで取得できるユーザーが増える事を確認してみてください。

#### ユーザーの更新

- URL:<http://localhost:8080/users/001>
- METHOD:PUT
- BODY:↓

```json
{
  "name": "gopher2"
}
```

GET メソッドでユーザーの一覧を取得すると、`name`が変わっている事が確認できます。

#### ユーザーの削除

- URL:<http://localhost:8080/001>
- METHOD:DELETE
- Response↓

```json
{
  "message": "user has been deleted"
}
```

GET メソッドでユーザーの一覧を取得すると削除された事が確認できます。

### 参考

- [[Golang/Go 言語]Gin フレームワークで API 開発 2~CRUD の実装~ | セルフノート](https://selfnote.work/20211120/programming/golang-gin-api-2/)

- [DynamoDB×Go 連載#1 Go で DynamoDB でおなじみの guregu/dynamo を利用する | フューチャー技術ブログ](https://future-architect.github.io/articles/20200225/)

- [gin を MVC で設計している時にコントローラのテストを書く](https://zenn.dev/diwamoto/articles/aba45dc2da36b8)

- [AWS Lambda で Golang の Web フレームワーク Gin を利用してみた | デロイト トーマツ ウェブサービス株式会社（DWS）公式ブログ](https://blog.mmmcorp.co.jp/blog/2020/06/07/aws-lambda-with-golang-gin/)

- [DynamoDB をローカルで使う](https://zenn.dev/ysmtegsr/articles/bb4c41aa28a9786de61d)

- [aws cli で DynamoDB を使う - Qiita](https://qiita.com/ekzemplaro/items/93c0aef433a2b633ab4a)
