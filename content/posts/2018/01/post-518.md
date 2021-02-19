---
title: '[CC] BitZenyをDockerでマイニングしてみる'
author: しゃまとん
type: post
date: 2018-01-21T02:57:17+00:00
url: /archives/518
featured_image: /wp-content/uploads/2018/01/icon.png
categories:
  - CC

---
お世話になっております。  
しゃまとんです。

仮想通貨が盛り上がっておりますね。  
かくいう私もちょっとしたきっかけがあり、知識をつけたいと思い少しずつ調べたりしてます。なんか仮想通貨に興味があるというと嫌な顔する人が多そうですけど。。。

今回はマイニングについてです。  
とはいってもマイニングのやり方〜みたいなのはもはや記事だらけでしょうし、調べていただければと思います。やってみると数値がインクリメントされていって面白いですね。

対象の通貨はBitZenyです。BitZenyはCPUでマイニングできて誰でもやりやすいみたいですね。ただMacでマイニングするには単純にmakeするだけはダメなようで、makefile等に変更を加えないと実行まで行けないようです。

そこで今回はDockerを使って、Ubuntuのコンテナを使ってminerをビルドしてみることにしました。Ubuntu（Linux）だと特に変更を加えることなくビルドして実行することができます。

Githubはこちら  
<a href="https://github.com/shamaton/docker-miner-zeny" target="_blank" rel="noopener">shamaton/docker-miner-zeny</a>

まずはDockerfileはこんな感じです。

<pre class="lang:default decode:true " title="Dockerfile">FROM ubuntu:xenial

MAINTAINER shamaton

WORKDIR /root

RUN apt-get update \
 && apt-get -y install \
    git \
    wget \
    libjansson-dev \
    automake \
    libtool \
    curl \
    libcurl3 \
    libcurl3-dev \
    make

WORKDIR /root
COPY start.sh /root/start.sh
RUN chmod a+x /root/start.sh

RUN git clone https://github.com/bitzeny/cpuminer \
 && cd cpuminer \
 && sh autogen.sh \
 && ./configure CFLAGS="-O3 -march=native -funroll-loops -fomit-frame-pointer"  \
 && make

CMD ["/root/start.sh"]</pre>

ビルドに必要なパッケージをインストール後、minerのリポジトリを取得します。  
そのまま実行ファイル生成までを行い、起動スクリプトを最後に実行します。  
上記のコンテナイメージを手元でビルドする場合は下記のコマンドを実行します。

<pre class="lang:default decode:true ">docker build -t zeny:latest .</pre>

ビルドが完了したら、イメージを利用してコンテナを立ち上げてみましょう。  
コンテナを起動する際には環境変数を指定することでマイニングを実行できるようにしています。  
私は<a href="http://daddy-pool.work/" target="_blank" rel="noopener">Daddy-Pool</a>を使っているのでそちらを例に実行してみます。

<pre class="lang:default decode:true ">docker run --name mining --env POOL=stratum+tcp://daddy-pool.work:15020 --env WALLET=ZrVLRVeLx9MTkETu8hMk3U2ZKao5Jx4ayC --env THREAD_NUM=1 zeny</pre>

実行して（yay!!!）の表示がでれば成功です。

<pre class="lang:default decode:true ">Starting Stratum on stratum+tcp://daddy-pool.work:15021
1 miner threads started, using 'yescrypt' algorithm.
Stratum requested work restart
thread 0: 1214 hashes, 0.51 khash/s
accepted: 1/1 (100.00%), 0.51 khash/s (yay!!!)</pre>

コンテナさえ用意できれば後はどのOSでも実行することができますね。  
ちなみにこちらのDockerfileはDockerhubにも公開しているのでpullしてすぐに使うことができます。

<a href="https://hub.docker.com/r/shamaton/miner-zeny/" target="_blank" rel="noopener">shamaton/miner-zeny</a>

<pre class="lang:default decode:true ">docker pull shamaton/miner-zeny</pre>

さくっとマイニング環境が作成できるので試してみてください！  
以上です。

■参考  
<a href="https://qiita.com/honeniq/items/07ae3d73eb0e8a8fb27f" target="_blank" rel="noopener">macOSでもBitZenyを掘りたい</a>

&nbsp;