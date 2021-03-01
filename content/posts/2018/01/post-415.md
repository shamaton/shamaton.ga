---
title: '[Go] Go + Redisで位置情報を扱ってみる'
author: しゃまとん
date: 2018-01-29T15:11:40+00:00
url: /posts/415
featured_image: /images/posts/2015/11/gopher.jpg
categories:
  - go
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

今回はGoで位置情報を扱ってみることにしました。  
ちなみに作者の稚拙アプリ「ことだまっぷ」でも似たようなことをして位置情報を利用しています。

{{< appbanner src="https://lh3.googleusercontent.com/AxCAhC3nUgPKVfKQlyZtGL0D0tKaghSsH-tV7NlXc5fSftWBPPBt5SHEIlHIlFexnG92=w170" title="ことだまっぷ" user="https://itunes.apple.com/jp/developer/masayuki-shamoto/id973807464?uo=4" ios="https://itunes.apple.com/jp/app/%E3%81%93%E3%81%A8%E3%81%A0%E3%81%BE%E3%81%A3%E3%81%B7/id1312331217?mt=8&uo=4" android="https://play.google.com/store/apps/details?id=com.shamaton.kotoda.map">}}

位置情報を扱うために、Redisを利用しています。  
実装の際にRDBMSでできるかな～と思っていたのですが、
色々しらべたところRedisが対応していてパフォーマンスも良さそうなのでredisを使うことにしました。

Goでredisを扱えるようにするパッケージは色々あり、個人的には[redigo][1]を使っていたのですが、
位置情報を扱えるようになっていませんでした。そこで別パッケージのgo-redisが対応されているということで使っています。

{{< blogcard url="https://github.com/go-redis/redis" >}}

実装の前にgo-redisを取得しておきましょう。

```shell
go get -u github.com/go-redis/redis
```

確認用コードは下記のようになります。  
処理的には指定座標の半径10km以内のデータを取得するようなものです。

```go
package main

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
}
```

コードを実装したら、Redisに確認用データを入れておきます。  
Redisに接続して下記のコマンドを実行し位置情報を入れておきます。

```text
GEOADD test 12.4762840270996 41.910514831543  "geo_1"
GEOADD test 12.4756031036377 41.910514831543  "geo_2"
GEOADD test 12.4768190383911 41.910514831543  "geo_3"
GEOADD test 12.4764785766602 41.9107551574707 "geo_4"
GEOADD test 12.4760408401489 41.9107551574707 "geo_5"
```

実行してみます。  
返り値はgo-redis側で用意されているので、扱いやすい状態になっています。  
というかgo-redisが結構使いやすい気がします。結果はこんな感じ。

```shell
go run main.go
result :  5
geo_4 : 41.910756366557116 : 12.476480305194855
geo_1 : 41.91051556804698 : 12.476281821727753
geo_5 : 41.910756366557116 : 12.476040422916412
geo_3 : 41.91051556804698 : 12.476818263530731
geo_2 : 41.91051556804698 : 12.47560054063797
```

これで位置情報も扱っていけそうです。  
ちなみに環境を汚したくないという方はDockerfileも置いておくので良かったら使ってください。

```dockerfile
FROM centos:7

MAINTAINER shamaton

RUN yum -y install wget git epel-release
RUN yum -y install redis

RUN mkdir -p /root/go \
&&  echo "GOPATH=/root/go" >> /root/.bashrc

RUN wget https://redirector.gvt1.com/edgedl/go/go1.9.2.linux-amd64.tar.gz \
&&  tar -C /usr/local -xzf go1.9.2.linux-amd64.tar.gz \
&&  rm go1.9.2.linux-amd64.tar.gz \
&&  echo "PATH=$PATH:/usr/local/go/bin:$GOPATH/bin" >> /root/.bashrc \
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
redis-server &
```

以上です。

■ 参考

{{< blogcard url="http://blog.shimar.me/2016/11/21/redis-georadius.html" >}}

 [1]: https://github.com/garyburd/redigo