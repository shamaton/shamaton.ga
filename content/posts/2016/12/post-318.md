---
title: '[Docker] CentOS6でMySQLコンテナを作成してみる'
author: しゃまとん
type: post
date: 2016-12-13T02:30:28+00:00
url: /posts/318
featured_image: /images/posts/2016/10/small_v-dark.png
categories:
  - docker
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

前回はRedisと連携をやってみたので、今回はMySQLコンテナを作ってみようと思います。  
MySQLは試しにAWSのRDSでデフォルトっぽい5.6.27（2016/12/13時点）で作成してみます。

OSは前回と同じでCentOS6を使います。  
ちなみに前回の記事はこちら

<blockquote class="wp-embedded-content">
  <p>
    <a href="http://shamaton.orz.hm/blog/posts/310">[Docker] golangとredisのコンテナを繋いでみた</a>
  </p>
</blockquote>



MySQLは公式でもイメージが用意されていますが、[Dockerfile][1]を見てもややこしい印象。  
で他の方々のを拝見している感じだと

・my.cnf（MySQLの設定ファイル）  
・mysql用のスクリプト

を事前に作成するなどして、コンテナ起動させている感じだな〜ということで参考にしつつDockerfileから。



CentOS6では標準でインストールされるMySQLが古いためrpmを取得してMySQL公式のリポジトリを有効にします。  
そして、指定のバージョンでyum installします。  
もしかしてmakeとかしないと行けないのかと思っていましたが、公式ありがとう。

そして、MySQLの設定ファイルを置き換えて、起動用スクリプトをコピーします。  
最後にPORTを設定し、スクリプトを実行するようにします。

my.cnfはでデフォルトで用意されているものにエンコード設定などを少し追加しました。  
なので純粋に起動確認するだけなら、置き換えはしなくていいと思います。



そして、起動スクリプトです。起動時にファイルがあるか確認してない場合、初期化を実行させるようにしました。  
今回は初期化とユーザー、データベース作成までおこなってみました。  
sleepは進捗がわかりやすいようにわざと入れています。



準備が整ったので、実行してみます。  
成功するとコンテナが生成されmysqlのプロセスが起動した状態になります。

<pre class="lang:sh decode:true ">$ docker build -t mysql56 .
Sending build context to Docker daemon 13.31 kB
... skip ...

$ docker run --name mysql_test mysql56
initialize...
Installing MySQL system tables... [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. 
Please use --explicit_defaults_for_timestamp server option (see documentation for more details).

... skip ...

mysqld_safe Logging to '/var/log/mysqld.log'.
mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql</pre>

&nbsp;

もう1つターミナルを開いて、中身を覗いてみます。

[<img src="http://shamaton.orz.hm/blog/images/posts/2016/10/docker_mysql_check.png" alt="docker_mysql_check" width="487" height="383" class="aligncenter size-full wp-image-323" />][2]

MySQLが起動してユーザーとデータベースが作成されているのが確認できました。  
このコンテナですが、消してしまうとデータが全て消えてしまいます。  
データを残して他コンテナで使いたい場合は別でデータ保存部分を用意する必要があるようです。またこの辺もやってみたいと思います。

以上です。  
[docker-mysql56-centos6][3]

■ 参考  
<a href="http://j-caw.co.jp/blog/?p=1583" target="_blank">Dockerでmysqlサーバコンテナへ別のコンテナからアクセスする<br /> </a><a href="http://weblabo.oscasierra.net/installing-mysql56-centos7-yum/" target="_blank">MySQL 5.6をCentOS 7にyum インストールする手順<br /> </a><a href="http://qiita.com/akinoriikeda/items/db4291dc4db3aa7c4b55" target="_blank">dockerで、いい感じでMySQL環境を作る<br /> </a><a href="http://qiita.com/TakamiChie/items/a7437b1a24961ba9c83e" target="_blank">DockerでMySQLを動かす(気を付けるべきところとかいろいろ)</a>

 [1]: https://github.com/docker-library/mysql/blob/a03bccc7dc259d817643b0ca0bfcf7ce52ea3906/5.6/Dockerfile
 [2]: http://shamaton.orz.hm/blog/images/posts/2016/10/docker_mysql_check.png
 [3]: https://github.com/shamaton/docker-mysql56-centos6