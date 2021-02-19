---
title: '[CI] concourseでUnityのビルドをCIしてみる – 導入編'
author: しゃまとん
type: post
date: 2019-07-02T13:49:22+00:00
url: /archives/664
featured_image: /wp-content/uploads/2019/06/concourse_icon.png
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
・UnityのiOS / Androidビルドが行える  
・Unity以外プロダクトにも使える(サーバサイドのテスト、ひいてはデプロイ)  
・属人化しないと嬉しい（Infrastructure as Codeのようなもの）  
  
なにか他にいいしーあいないかなーと探してみたところ[concourse][4]と出会いました。concourseは比較的新しいOSSによるCI/CDツールのようですが、ヤフー等の企業でも導入が進んでいるようです。 目立った特徴を見た感じだと  
・dockerを使って、CI環境を構築できる  
・workerはコンテナ、ネイティブどちらでも動作可能  
・操作はすべてCLI  
・結果をGUIで確認できる  
(詳細は[こちら][5]にあります)  
  
workerがどちらでも動作するので、MacでUnityのビルド走らせつつ、コンテナでサーバサイドのテストとかできそう。  
ということで、要件を満たしてくれそうなのでやってみることにしました。  
ここから導入になるのですが、先人の記事を参考にしてこの記事も書いているため、新しいバージョンでは動作しない恐れがあります。  
この時、導入時にハマったところも含め記載するのですが、ご留意ください。  
  
まず、docker/docker-composeの導入ができてない方は下記を参考にして、使えるようにしましょう。（他力本願）  
  
次に作業用ディレクトリを作成して、動作に必要な鍵を作っておきます。ssh-keygenを使えばいいのですが、 生成時のオプション指定に注意する必要がありました。  
<a rel="noreferrer noopener" aria-label="github issue : panic: runtime error: invalid memory address or nil... (新しいタブで開く)" href="https://github.com/concourse/concourse/issues/2590" target="_blank">github issue : panic: runtime error: invalid memory address or nil&#8230;</a>  


<pre class="wp-block-code"><code>mkdir concourse_sample
cd concourse_sample

mkdir -p keys/web
ssh-keygen -t rsa -f ./keys/web/tsa_host_key -N '' -m PEM
ssh-keygen -t rsa -f ./keys/web/session_signing_key -N '' -m PEM

mkdir -p keys/worker
ssh-keygen -t rsa -f ./keys/worker/worker_key -N '' -m PEM
ssh-keygen -t rsa -f ./keys/worker/darwin_worker_key -N '' -m PEM

cat ./keys/worker/worker_key.pub >> ./keys/web/authorized_worker_keys
cat ./keys/worker/darwin_worker_key.pub >> ./keys/web/authorized_worker_keys

cp ./keys/web/tsa_host_key.pub ./keys/worker</code></pre> できたら、docker-compose用のconcourse構成を下記のようにして作成します。

  
environmentは環境変数から設定しても構いません。CONCOURSE\_EXTERNAL\_URLは他のPCをworkerとして使う場合はlocalhostではなく、  
IPアドレスかホスト名を設定してください。 

<hr class="wp-block-separator" />

<pre class="wp-block-preformatted">version: '3'
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
       CONCOURSE_TSA_HOST: concourse-web:2222 </pre>

<hr class="wp-block-separator" />
用意できたら、docker-compose upしてみましょう。
  
はじめは正常に起動しているかログを確認したほうがよいでしょう。  
failed や unreachable などのワードが見えたら、どこかしらがうまく行ってないと思います。  
問題なければ、<a rel="noreferrer noopener" aria-label="http://localhost:8080 (新しいタブで開く)" href="http://localhost:8080" target="_blank">http://localhost:8080</a>にアクセスすれば、welcome to concourseと表示されます。 <figure class="wp-block-image"><img src="https://shamaton.orz.hm/blog/wp-content/uploads/2019/06/concourse1_login.png" alt="" class="wp-image-665" /></figure> test : testでログインできるかも確認してみましょう。 <figure class="wp-block-image"><img src="https://shamaton.orz.hm/blog/wp-content/uploads/2019/06/concourse1_nopipe.png" alt="" class="wp-image-666" /></figure> 次にネイティブで動くworkerを動作させます。  
動作させるには、concourseのCLIを使う必要があります。<a href="https://github.com/concourse/concourse/releases" target="_blank" rel="noreferrer noopener" aria-label="こちら (新しいタブで開く)">こちら</a>から  
から各OSに適したものを取得してください。今回はmacなのでconcourse-X.X.X-darwin-amd64.tgzを取得しました。  
展開するとconcourse / flyコマンドがありますので、適宜パスを通すなどしましょう。 

<pre class="wp-block-preformatted">tar -xvzf concourse-X.X.X-darwin-amd64.tgz
 cd concourse
 tar -xvzf fly-assets/fly-darwin-amd64.tgz
# 下記のコマンドに適宜パスを通す
 ./bin/concourse
 ./fly-assets/fly </pre> workerの起動はホストや鍵を設定して行います。working directoryは勝手に生成されます。

  
こちらも実行直後はログをみて、起動に失敗してないかチェックしてみましょう。 

<pre class="wp-block-preformatted">concourse worker --work-dir ./working_sample \
    --tsa-host localhost:2222 \
    --tsa-public-key ./keys/worker/tsa_host_key.pub \
    --tsa-worker-private-key ./keys/worker/darwin_worker_key</pre> worker.beacon-runner.beacon.registered等出ていれば良いと思います。

  
  
これで導入が終わりました。  
次は、pipelineを作って実行していきます！  
  
■ 参考

 [1]: https://travis-ci.org/
 [2]: https://circleci.com/
 [3]: https://unity3d.com/jp/unity/features/cloud-build
 [4]: https://concourse-ci.org/
 [5]: https://www.ossnews.jp/compare/Jenkins/Concourse_CI