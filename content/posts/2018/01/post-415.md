---
title: '[Go] Go + Redisで位置情報を扱ってみる'
author: しゃまとん
type: post
date: 2018-01-29T15:11:40+00:00
url: /archives/415
featured_image: /wp-content/uploads/2015/11/gopher.jpg
categories:
  - go
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

今回はGoで位置情報を扱ってみることにしました。  
ちなみに作者の稚拙アプリ「ことだまっぷ」でも似たようなことをして位置情報を利用しています。

<img src="//lh3.googleusercontent.com/AxCAhC3nUgPKVfKQlyZtGL0D0tKaghSsH-tV7NlXc5fSftWBPPBt5SHEIlHIlFexnG92=w170" alt="ことだまっぷ" style="float: left; margin: 10px; width: 25%; max-width: 120px; border-radius: 10%;" /> 

<div class="appreach-info" style="margin: 10px;">
  <div id="appreach-appname">
    ことだまっぷ
  </div>
  
  <div id="appreach-developer" style="font-size: 80%; display: inline-block; _display: inline;">
    開発元:<a id="appreach-developerurl" href="https://itunes.apple.com/jp/developer/masayuki-shamoto/id973807464?uo=4" target="_blank" rel="nofollow noopener">shamaton</a>
  </div>
  
  <div id="appreach-price" style="font-size: 80%; display: inline-block; _display: inline;">
    無料
  </div>
  
  <div class="appreach-powered" style="font-size: 80%; display: inline-block; _display: inline;">
    posted with <a title="アプリーチ" href="http://mama-hack.com/app-reach/" target="_blank" rel="nofollow noopener">アプリーチ</a>
  </div>
  
  <div class="appreach-links" style="float: left;">
    <div id="appreach-itunes-link" style="display: inline-block; _display: inline;">
      <a id="appreach-itunes" href="https://itunes.apple.com/jp/app/%E3%81%93%E3%81%A8%E3%81%A0%E3%81%BE%E3%81%A3%E3%81%B7/id1312331217?mt=8&uo=4" target="_blank" rel="nofollow noopener"><br /> <img src="https://nabettu.github.io/appreach/img/itune_ja.svg" style="height: 40px; width: 135px;" /><br /> </a>
    </div>
    
    <div id="appreach-gplay-link" style="display: inline-block; _display: inline;">
      <a id="appreach-gplay" href="https://play.google.com/store/apps/details?id=com.shamaton.kotoda.map" target="_blank" rel="nofollow noopener"><br /> <img src="https://nabettu.github.io/appreach/img/gplay_ja.png" style="height: 40px; width: 134.5px;" /><br /> </a>
    </div>
  </div>
</div>

<div class="appreach-footer" style="margin-bottom: 10px; clear: left;">
</div>

位置情報を扱うために、Redisを利用しています。  
実装の際にRDBMSでできるかな～と思っていたのですが、色々しらべたところRedisが対応していてパフォーマンスも良さそうなのでredisを使うことにしました。

Goでredisを扱えるようにするパッケージは色々あり、個人的には[redigo][1]を使っていたのですが、位置情報を扱えるようになっていませんでした。そこで別パッケージの[go-redis][2]が対応されているということで使っています。

実装の前にgo-redisを取得しておきましょう。

<pre class="lang:default decode:true ">go get -u github.com/go-redis/redis</pre>

確認用コードは下記のようになります。  
処理的には指定座標の半径10km以内のデータを取得するようなものです。

<pre class="lang:go decode:true ">package main

import (
    "fmt"
    "github.com/go-redis/redis"
)

func main() {

    client := redis.NewClient(&redis.Options{
            Addr:     "localhost:6379",
            Password: "", // no password set
            DB:       0,  // use default DB
    })

    // query
    query := &redis.GeoRadiusQuery{
        Radius:      10,
        Unit:        "km",
        WithGeoHash: false,
        WithCoord:   true,
        WithDist:    false,
        Count:       10,
        Sort:        "ASC",
    }

    // get
    res, err := client.GeoRadius("test", 12.4764785766602, 41.9107551574707, query).Result()
    if err == redis.Nil {
        fmt.Println("redis nil........")
    } else if err != nil {
        panic(err)
    } else {
        fmt.Println("result : ", len(res))
    }

    for _, v := range res {
        fmt.Println(v.Name, ":", v.Latitude, ":", v.Longitude)
    }
}</pre>

コードを実装したら、Redisに確認用データを入れておきます。  
Redisに接続して下記のコマンドを実行し位置情報を入れておきます。

<pre class="lang:default decode:true ">GEOADD test 12.4762840270996 41.910514831543  "geo_1"
GEOADD test 12.4756031036377 41.910514831543  "geo_2"
GEOADD test 12.4768190383911 41.910514831543  "geo_3"
GEOADD test 12.4764785766602 41.9107551574707 "geo_4"
GEOADD test 12.4760408401489 41.9107551574707 "geo_5"</pre>

実行してみます。  
返り値はgo-redis側で用意されているので、扱いやすい状態になっています。  
というかgo-redisが結構使いやすい気がします。結果はこんな感じ。

<pre class="lang:default decode:true "># go run main.go
result :  5
geo_4 : 41.910756366557116 : 12.476480305194855
geo_1 : 41.91051556804698 : 12.476281821727753
geo_5 : 41.910756366557116 : 12.476040422916412
geo_3 : 41.91051556804698 : 12.476818263530731
geo_2 : 41.91051556804698 : 12.47560054063797</pre>

これで位置情報も扱っていけそうです。  
ちなみに環境を汚したくないという方はDockerfileも置いておくので良かったら使ってください。

<pre class="lang:default decode:true" title="Dockerfile">FROM centos:7

MAINTAINER shamaton

RUN yum -y install wget git epel-release
RUN yum -y install redis

RUN mkdir -p /root/go \
&&  echo "GOPATH=/root/go" &gt;&gt; /root/.bashrc

RUN wget https://redirector.gvt1.com/edgedl/go/go1.9.2.linux-amd64.tar.gz \
&&  tar -C /usr/local -xzf go1.9.2.linux-amd64.tar.gz \
&&  rm go1.9.2.linux-amd64.tar.gz \
&&  echo "PATH=$PATH:/usr/local/go/bin:$GOPATH/bin" &gt;&gt; /root/.bashrc \
&&  source /root/.bashrc

RUN /usr/local/go/bin/go get -u github.com/go-redis/redis

WORKDIR /root

CMD ["/bin/sh"]</pre>

コンテナのビルドと実行は下記な感じで。

<pre class="lang:default decode:true ">docker build -t goredis .
docker run --name go_redis -i -t goredis:latest /bin/bash

# 確認
go version
ls go/src/github.com
# redisの起動（コンテナ内）
redis-server &</pre>

以上です。

■ 参考  
<a href="http://blog.shimar.me/2016/11/21/redis-georadius.html" target="_blank" rel="noopener">Redisで位置情報を扱うコマンド</a>

 [1]: https://github.com/garyburd/redigo
 [2]: https://github.com/go-redis/redis