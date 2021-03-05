---
title: '[Docker] CentOS6でGo言語の開発環境を作ってみた'
author: しゃまとん
date: 2016-10-08T03:08:44+00:00
url: /posts/305
featured_image: /images/posts/2016/10/small_v-dark.png
is_comment_form_freeze:
  - on
comment_form_freeze_message:
  - ' '
categories:
  - docker

---
お世話になっております。  
しゃまとんです。

巷で話題のdockerに興味が湧いてしまいまして、試しに何かやってみようということでイメージを作ってみたメモです。

何をやろうかなぁということでちょっとしたGoの開発環境をつくることにしました。  
作成された環境は下記のような感じを想定しました。

・Go言語をインストールされ開発できる  
・go getしたいのでgitがインストールされている  
・いつも使うCentOS6をベースにする

dockerではイメージを作って、イメージからコンテナを生成することで使い捨て可能な環境を実現できるそうです。
イメージはDockerfileを用意することで作成が可能です。

ということでDockerfileを作ってみました。gitの最新版を導入し、goをインストールし、
dockerというユーザーを作成し、GOPATHを設定しています。

イメージを作成には下記のようにコマンド実行します。go_verは指定しない場合Dockerfileに定義されている値を使います。

```shell
docker build . -t <タグ名> --build-arg go_ver=1.7.1
```

ずらずらーと実行されて、イメージが作成されます。

{{< figure src="/images/posts/2016/10/docker_image.png" >}}

イメージを使ってコンテナを生成して起動してみます。

```shell
docker run --name <名前> -it <タグ名> /bin/bash
```

もろもろ確認してみます。

{{< figure src="/images/posts/2016/10/docker_run_check.png" >}}

確認が終わったら削除します。起動時に名前をつけておくと削除しやすいですね。

```shell
docker rm <名前>
```

Dockerfileの記述は知識が必要ですが、なれればサクサクを環境を作っていけそうな感じでおもしろいですね。
作ったイメージはdockerhubで共有できたり、他のユーザーが作ったイメージも使えるので、とても便利そう。

自分のやつも一応アップしておきましたのでご自由にどうぞ。

{{< blogcard url="https://github.com/shamaton/golang-centos6" >}}

dockerhubはこちら。  
 [dockerhub/golang-centos6][4]

色々と有効活用したいですね。  
以上です。

■ 参考

{{< blogcard url="http://qiita.com/muddydixon/items/15e5fe8f04a0c325eb8e" >}}

 [4]: https://hub.docker.com/r/shamaton/golang-centos6/
