---
title: '[CI] concourseでUnityのビルドをCIしてみる – 導入編'
author: しゃまとん
date: 2019-07-02T13:49:22+00:00
url: /posts/664
featured_image: /images/posts/2019/06/concourse_icon.png
is_comment_form_freeze:
  - on
comment_form_freeze_message:
  - ' '
categories:
  - CI/CD
  - Concourse

---
 お世話になっております。  
しゃまとんです。  
  
Unityで開発をしているとビルド作業が煩わしくなって自動化したい…なんて話になると思います。  
それで導入となるとどのCI導入するのかという話になり、Jenkinsで環境構築をするのが主流なのかなと感じています。  
（[Travis CI][1]や[Circle CI][2]とかあるけど、ローカルな環境で〜という話）  
  
ただ、世の中では  
「Jenkinsおじさん」  
という問題が各所で発生しており、辛い現状があるのも事実なようです。  
  
Unityにおいては[Unity Cloud Build][3]がありますが、Unityのサービスなので他のプロダクト（例えばサーバサイドのAPIサービス）とかには使えません。  
できるなら、1つのCI環境で管理したいですよね。  
  
私としては以下のような希望がありまして  
* UnityのiOS / Androidビルドが行える  
* Unity以外プロダクトにも使える(サーバサイドのテスト、ひいてはデプロイ)  
* 属人化しないと嬉しい（Infrastructure as Codeのようなもの）  
  
なにか他にいいしーあいないかなーと探してみたところ[concourse][4]と出会いました。
concourseは比較的新しいOSSによるCI/CDツールのようですが、ヤフー等の企業でも導入が進んでいるようです。

{{< blogcard url="https://thinkit.co.jp/article/11556">}}

目立った特徴を見た感じだと  
* dockerを使って、CI環境を構築できる  
* workerはコンテナ、ネイティブどちらでも動作可能  
* 操作はすべてCLI  
* 結果をGUIで確認できる  
(詳細は[こちら][5]にあります)  
  
workerがどちらでも動作するので、MacでUnityのビルド走らせつつ、コンテナでサーバサイドのテストとかできそう。  
ということで、要件を満たしてくれそうなのでやってみることにしました。  
ここから導入になるのですが、先人の記事を参考にしてこの記事も書いているため、新しいバージョンでは動作しない恐れがあります。  
この時、導入時にハマったところも含め記載するのですが、ご留意ください。  
  
まず、docker/docker-composeの導入ができてない方は下記を参考にして、使えるようにしましょう。（他力本願）  

{{< blogcard url="https://upd.world/docker-install-mac/">}}

{{< blogcard url="https://qiita.com/zembutsu/items/dd2209a663cae37dfa81">}}
  
次に作業用ディレクトリを作成して、動作に必要な鍵を作っておきます。ssh-keygenを使えばいいのですが、
生成時のオプション指定に注意する必要がありました。

[github issue : panic: runtime error: invalid memory address or nil...](https://github.com/concourse/concourse/issues/2590)  

```shell
mkdir concourse_sample
cd concourse_sample

mkdir -p keys/web
ssh-keygen -t rsa -f ./keys/web/tsa_host_key -N '' -m PEM
ssh-keygen -t rsa -f ./keys/web/session_signing_key -N '' -m PEM

mkdir -p keys/worker
ssh-keygen -t rsa -f ./keys/worker/worker_key -N '' -m PEM
ssh-keygen -t rsa -f ./keys/worker/darwin_worker_key -N '' -m PEM

cat ./keys/worker/worker_key.pub >> ./keys/web/authorized_worker_keys
cat ./keys/worker/darwin_worker_key.pub >> ./keys/web/authorized_worker_keys

cp ./keys/web/tsa_host_key.pub ./keys/worker
```
できたら、docker-compose用のconcourse構成を下記のようにして作成します。

environmentは環境変数から設定しても構いません。CONCOURSE_EXTERNAL_URLは他のPCをworkerとして使う場合はlocalhostではなく、
IPアドレスかホスト名を設定してください。 

```yaml
version: '3'
 services:
   concourse-db:
     image: postgres
     environment:
       POSTGRES_DB: concourse
       POSTGRES_USER: concourse_user
       POSTGRES_PASSWORD: concourse_pass
       PGDATA: /database
   
   concourse-web:
     image: concourse/concourse
     command: web
     restart: unless-stopped
     depends_on: [concourse-db]
     ports: ["8080:8080", "2222:2222"]
     volumes: ["./keys/web:/concourse-keys"]
     environment:
       CONCOURSE_EXTERNAL_URL: http://localhost:8080
       CONCOURSE_BASIC_AUTH_USERNAME: auth_user
       CONCOURSE_BASIC_AUTH_PASSWORD: auth_pass
       CONCOURSE_POSTGRES_HOST: concourse-db
       CONCOURSE_POSTGRES_USER: concourse_user
       CONCOURSE_POSTGRES_DATABASE: concourse
       CONCOURSE_POSTGRES_PASSWORD: concourse_pass
       CONCOURSE_ADD_LOCAL_USER: test:test
       CONCOURSE_MAIN_TEAM_LOCAL_USER: test
 
   concourse-worker:
     image: concourse/concourse
     privileged: true
     depends_on: [concourse-web]
     command: worker
     volumes: ["./keys/worker:/concourse-keys"]
     environment:
       CONCOURSE_TSA_HOST: concourse-web:2222
```

用意できたら、docker-compose upしてみましょう。
  
はじめは正常に起動しているかログを確認したほうがよいでしょう。  
failed や unreachable などのワードが見えたら、どこかしらがうまく行ってないと思います。  
問題なければ、[http://localhost:8080](http://localhost:8080)にアクセスすれば、welcome to concourseと表示されます。

{{< figure src="/images/posts/2019/06/concourse1_login.png" class="left" width="600" >}}

test : testでログインできるかも確認してみましょう。

{{< figure src="/images/posts/2019/06/concourse1_nopipe.png" class="left" width="600" >}}

次にネイティブで動くworkerを動作させます。  
動作させるには、concourseのCLIを使う必要があります。[こちら](https://github.com/concourse/concourse/releases)から  
から各OSに適したものを取得してください。今回はmacなのでconcourse-X.X.X-darwin-amd64.tgzを取得しました。  
展開するとconcourse / flyコマンドがありますので、適宜パスを通すなどしましょう。 

```shell
tar -xvzf concourse-X.X.X-darwin-amd64.tgz
cd concourse
tar -xvzf fly-assets/fly-darwin-amd64.tgz

# 下記のコマンドに適宜パスを通す
./bin/concourse
./fly-assets/fly
```

workerの起動はホストや鍵を設定して行います。working directoryは勝手に生成されます。  
こちらも実行直後はログをみて、起動に失敗してないかチェックしてみましょう。 

```shell
concourse worker --work-dir ./working_sample \
    --tsa-host localhost:2222 \
    --tsa-public-key ./keys/worker/tsa_host_key.pub \
    --tsa-worker-private-key ./keys/worker/darwin_worker_key
```

worker.beacon-runner.beacon.registered等出ていれば良いと思います。
  
これで導入が終わりました。  
次は、pipelineを作って実行していきます！  
  
■ 参考

{{< blogcard url="https://blog.fenrir-inc.com/jp/2017/07/how_to_build_ios_codes_by_concourse-ci.html">}}

{{< blogcard url="https://dev.classmethod.jp/server-side/network/openssh78_potentially_incompatible_changes/">}}

 [1]: https://travis-ci.org/
 [2]: https://circleci.com/
 [3]: https://unity3d.com/jp/unity/features/cloud-build
 [4]: https://concourse-ci.org/
 [5]: https://www.ossnews.jp/compare/Jenkins/Concourse_CI