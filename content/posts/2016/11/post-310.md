---
title: '[Docker] golangとredisのコンテナを繋いでみた'
author: しゃまとん
date: 2016-11-22T16:22:28+00:00
url: /posts/310
featured_image: /images/posts/2016/10/small_v-dark.png
categories:
  - docker
  - go
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

先月のDocker触ってみた話に引き続いて、作ったgolang環境からredisに繋いでみるテストをしました。
今回は複数のコンテナでの連携的なやつです。

{{< blogcard url="/posts/305">}}

連携にはdocker-composeという仕組み（？）を使うといい感じにやってくれるらしいです。
docker-composeを使うにはdocker-compose.ymlを作成する必要があります。

{{< blogcard url="https://docs.docker.com/compose/">}}

golangからredisに接続して操作するには外部のパッケージが必要なので、自分で作成したimageを利用して、
環境を用意するDockerfileを作成します。確認用のコードは記事にしたものをそのまま使います。

{{< blogcard url="/posts/141">}}

まずはDockerfileです。  
やっているのは、dockerhubからimageを取得し、redigoをgo getし、確認用コードをコピーします。
redigoでの接続先はdockerの状況で多少異なるかもしれません。（特に意識しなければ、192.168.99.100:6379）

{{< gist shamaton 9805b8cc93180798337d42e144ba393b >}}

次にdocker-compose.ymlです。  
redisはイメージから取得（image）、go-redisはdockerfileから作成する設定（build）にします。
そしてredisにlinkするように設定します。

{{< gist shamaton 39278b6ccbb9bbadceabb510a87b393b >}}

あとはtest_redis.goを用意したら、イメージとコンテナを作成していきます。まずはイメージをビルドします。

```shell
$ cd golang-redis
$ docker-compose build<
```

実行すると、go_redisのイメージ生成が行われます。（名前がちょっと変）

{{< figure src="/images/posts/2016/10/images.png" >}}

それではdocker-composeを利用して、コンテナを起動します。
redisのイメージがローカルにない場合は、取得しにいきます。
今回はrunの後にgo_redisを指定して生成したコンテナに接続するようにしています。

```shell
docker-compose run go_redis bash
```

ターミナル表示が切り替わるので、テストコードを実行してみます。

{{< figure src="/images/posts/2016/10/test_redis.png" >}}

こんな感じで接続できました。docker-composeを使うと簡単に複数のコンテナを組み合わせて使えるし、
構成もサクッと変更できて便利ですね。

今回もリポジトリにしておいたので、ご自由にお使いくださいませ。  
以上です。

{{< blogcard url="https://github.com/shamaton/docker-goredis-centos6">}}