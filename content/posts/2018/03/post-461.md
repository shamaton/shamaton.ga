---
title: '[Go] Goからredis-clusterにつないでみる'
author: しゃまとん
type: post
date: 2018-03-12T13:32:29+00:00
url: /archives/461
featured_image: /wp-content/uploads/2017/12/redis_logo.png
categories:
  - go
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

今回はredisをクラスタリングしてGoから接続してみるやつです。  
個人ではなかなか使う機会がないですが、負荷分散につかう手法みたいなものですね。

ちょっと古めに記事ですが、最初に説明がかかれています。  
<a href="https://cloudpack.media/420" target="_blank" rel="noopener">Redisってなんじゃ？（レプリケーションとクラスタリング）</a>

万が一、億が一ヒットした場合にしておくと慌てないです済むかなとおもったので理解しておいて損はないかなーと思いやってみることにしました。（実はWEB DB PRESSのKubernetes特集がきっかけでもあります）

とりあえずクラスタをつくるには複数台Redisが起動している必要があります。  
今回もサクッと環境を捨てられるDockerを利用して試してみましょう。

まずは検証用コンテナとして適当なOSイメージをもってきてコンテナを起動しましょう  
（例ではUbuntuを使っています。）

<pre class="lang:default decode:true">docker pull ubuntu
docker run --name redis_cluster_test -i -t ubuntu /bin/bash</pre>

コンテナを立ち上げたら、redisをインストールします。  
Ubuntuでデフォルトでインストールできるredisのバージョンが古いため、ソースから取得しました。（最新は[こちら][1]を確認）

<pre class="lang:default decode:true">apt-get update

# パッケージ取得
apt-get -y install gcc make python-dev tcl wget vim

# redisのインストール
wget http://download.redis.io/releases/redis-3.2.11.tar.gz
tar vxzf redis-3.2.11.tar.gz
make -C  redis-3.2.11
make PREFIX=/usr/local -C redis-3.2.11 install
rm -rf redis-3.2.11.tar.gz redis-3.2.11

# 確認
redis-server --version</pre>

次に複数立ち上げるために、設定ファイルを作成します。  
クラスタリングには最低6つ（master,slaveが3つずつ）必要なので、6つファイルをつくります。1つの例はこちら。

<pre class="lang:default decode:true" title="cluster0.conf"># portは7000 - 7005まで
port 7000
cluster-enabled yes

# node-0　 - node-5まで
cluster-config-file nodes-0.conf
cluster-node-timeout 5000
appendonly yes
protected-mode no</pre>

6ファイル作ったら、ファイルを指定して実行していきます。

<pre class="lang:default decode:true">redis-server cluster0.conf &
redis-server cluster1.conf &
redis-server cluster2.conf &
redis-server cluster3.conf &
redis-server cluster4.conf &
redis-server cluster5.conf &

# プロセス確認　
ps -aux</pre>

次にこれをクラスタリングしていきます。  
今回はクラスタを半自動的に作ってくれるredis-trib.rbを使うことにします。Rubyが使える必要があるので、合わせてインストールしていきます。

<pre class="lang:default decode:true">apt-get -y install ruby
gem install redis
wget http://download.redis.io/redis-stable/src/redis-trib.rb
chmod 755 redis-trib.rb</pre>

準備が出来たのでクラスタを作ってみましょう。  
下記コマンドを実行して返答すれば完成です。

<pre class="lang:default decode:true"># クラスタ生成
./redis-trib.rb create --replicas 1 \
  127.0.0.1:7000 \
  127.0.0.1:7001 \
  127.0.0.1:7002 \
  127.0.0.1:7003 \
  127.0.0.1:7004 \
  127.0.0.1:7005

>&gt;&gt; Creating cluster
>&gt;&gt; Performing hash slots allocation on 6 nodes...
Using 3 masters:
127.0.0.1:7000
127.0.0.1:7001
127.0.0.1:7002
Adding replica 127.0.0.1:7003 to 127.0.0.1:7000
Adding replica 127.0.0.1:7004 to 127.0.0.1:7001
Adding replica 127.0.0.1:7005 to 127.0.0.1:7002
M: 6937b5904ef232fd9e9a622da78d9cb92baff38e 127.0.0.1:7000
   slots:0-5460 (5461 slots) master
M: f4b3c573408ca399ac3f4370413946550c109753 127.0.0.1:7001
   slots:5461-10922 (5462 slots) master
M: 552810f0a7fdd2e320fa15dbfa22385b6e73f5ad 127.0.0.1:7002
   slots:10923-16383 (5461 slots) master
S: 5ba7ef8bbb45d8fcc60ddbe67e7f3103a9a56caa 127.0.0.1:7003
   replicates 6937b5904ef232fd9e9a622da78d9cb92baff38e
S: 2bbec261f0ae973520da1c089dde6fe2ff134085 127.0.0.1:7004
   replicates f4b3c573408ca399ac3f4370413946550c109753
S: add1867a1d5b93dd83d5f208ae0e2da758b86aba 127.0.0.1:7005
   replicates 552810f0a7fdd2e320fa15dbfa22385b6e73f5ad

# yesとする
Can I set the above configuration? (type 'yes' to accept): yes


>&gt;&gt; Nodes configuration updated

...（省略）...

[OK] All nodes agree about slots configuration.
>&gt;&gt; Check for open slots...
>&gt;&gt; Check slots coverage...
[OK] All 16384 slots covered.</pre>

一応、確認してみましょう。  
コマンドを別の場所から実行するとRedirectしているのがわかります。

<pre class="lang:default decode:true"># 接続
redis-cli -c -h localhost -p 7000

# 値を設定
localhost:7000&gt; set hoge fuga

# 値を取得
localhost:7000&gt; get hoge
"fuga"

# 別のところに接続
redis-cli -c -h localhost -p 7001

# redirectする
localhost:7001&gt; get hoge
-&gt; Redirected to slot [1525] located at 127.0.0.1:7000
"fuga"</pre>

あとはGoから確認してみましょう。  
乱暴ですが下記コマンドでまとめて Goを使えるようにします。

<pre class="lang:default decode:true">apt-get -y install git \
&& wget https://redirector.gvt1.com/edgedl/go/go1.9.2.linux-amd64.tar.gz \
&&  tar -C /usr/local -xzf go1.9.2.linux-amd64.tar.gz \
&&  rm go1.9.2.linux-amd64.tar.gz \
&&  mkdir -p /root/go \
&&  echo "GOPATH=/root/go" &gt;&gt; /root/.bashrc \
&&  echo "PATH=$PATH:/usr/local/go/bin:$GOPATH/bin" &gt;&gt; /root/.bashrc \
&&  source /root/.bashrc \
&&  go get -u github.com/go-redis/redis</pre>

サンプルコードはこちらになります。（今回もgo-redisを使います）

<pre class="lang:go decode:true " title="main.go">package main

import (
    "fmt"
    "github.com/go-redis/redis"
)

func main() {
    client := redis.NewClusterClient(&redis.ClusterOptions{
                Addrs: []string{"localhost:7002"},
    })

    err := client.Set("key", "value", 0).Err()
    if err != nil {
            panic(err)
    }

    val, err := client.Get("key").Result()
    if err != nil {
            panic(err)
    }
    fmt.Println("key", val)

    val2, err := client.Get("key2").Result()
    if err == redis.Nil {
            fmt.Println("key2 does not exists")
    } else if err != nil {
            panic(err)
    } else {
            fmt.Println("key2", val2)
    }
}</pre>

実行すると、値が取得できていますね。

<pre class="lang:default decode:true">＃ 実行
go run main.go

#  結果
key value
key2 does not exists</pre>

ちなみにこのときNewClusterClientでないとうまく取得することができません。クラスタを生成してない場合は逆もしかりですね。  
前準備のほうが全然長くなってしまいましたが、以上です。

 [1]: http://download.redis.io/releases/