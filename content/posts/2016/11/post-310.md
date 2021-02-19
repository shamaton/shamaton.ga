---
title: '[Docker] golangとredisのコンテナを繋いでみた'
author: しゃまとん
type: post
date: 2016-11-22T16:22:28+00:00
url: /archives/310
featured_image: /wp-content/uploads/2016/10/small_v-dark.png
categories:
  - docker
  - go
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

先月のDocker触ってみた話に引き続いて、作ったgolang環境からredisに繋いでみるテストをしました。今回は複数のコンテナでの連携的なやつです。

<blockquote class="wp-embedded-content">
  <p>
    <a href="http://shamaton.orz.hm/blog/archives/305">[Docker] CentOS6でGo言語の開発環境を作ってみた</a>
  </p>
</blockquote>



連携にはdocker-composeという仕組み（？）を使うといい感じにやってくれるらしいです。docker-composeを使うにはdocker-compose.ymlを作成する必要があります。

<a href="https://docs.docker.com/compose/" target="_blank">Docker Compose</a>

golangからredisに接続して操作するには外部のパッケージが必要なので、自分で作成したimageを利用して、環境を用意するDockerfileを作成します。確認用のコードは記事にしたものをそのまま使います。

<blockquote class="wp-embedded-content">
  <p>
    <a href="http://shamaton.orz.hm/blog/archives/141">[golang]redisで構造体を扱ってみる</a>
  </p>
</blockquote>



まずはDockerfileです。  
やっているのは、dockerhubからimageを取得し、redigoをgo getし、確認用コードをコピーします。redigoでの接続先はdockerの状況で多少異なるかもしれません。（特に意識しなければ、192.168.99.100:6379）



次にdocker-compose.ymlです。  
redisはイメージから取得（image）、go-redisはdockerfileから作成する設定（build）にします。そしてredisにlinkするように設定します。



あとはtest_redis.goを用意したら、イメージとコンテナを作成していきます。まずはイメージをビルドします。

<pre class="lang:default decode:true ">$ cd golang-redis
$ docker-compose build</pre>

実行すると、go_redisのイメージ生成が行われます。（名前がちょっと変）

[<img src="http://shamaton.orz.hm/blog/wp-content/uploads/2016/10/images.png" alt="images" width="594" height="71" class="aligncenter size-full wp-image-311" />][1]

それではdocker-composeを利用して、コンテナを起動します。redisのイメージがローカルにない場合は、取得しにいきます。今回はrunの後にgo_redisを指定して生成したコンテナに接続するようにしています。

<pre class="lang:default decode:true ">$ docker-compose run go_redis bash</pre>

ターミナル表示が切り替わるので、テストコードを実行してみます。

[<img src="http://shamaton.orz.hm/blog/wp-content/uploads/2016/10/test_redis.png" alt="test_redis" width="727" height="99" class="aligncenter size-full wp-image-312" />][2]

こんな感じで接続できました。docker-composeを使うと簡単に複数のコンテナを組み合わせて使えるし、構成もサクッと変更できて便利ですね。

今回も[リポジトリ][3]にしておいたので、ご自由にお使いくださいませ。  
以上です。

 [1]: http://shamaton.orz.hm/blog/wp-content/uploads/2016/10/images.png
 [2]: http://shamaton.orz.hm/blog/wp-content/uploads/2016/10/test_redis.png
 [3]: https://github.com/shamaton/docker-goredis-centos6