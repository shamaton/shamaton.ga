---
title: '[Golang] gojiでも接続数を制限してみる'
author: しゃまとん
type: post
date: 2017-02-15T13:40:07+00:00
url: /archives/271
featured_image: /wp-content/uploads/2016/08/goji.png
categories:
  - go
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

go言語の軽量フレームワークであるgojiを触っておりまして、ふと大量のリクエストが来た時ってプロセス生成ってどうなっているのだろう・・と思い確認してみました。

とりあえずgojiをgo getして最低限動作する状態にします。

<pre class="lang:go decode:true" title="goji_sample.go">package main

import (
    "fmt"
    "net/http"

    "github.com/zenazn/goji"
    "github.com/zenazn/goji/web"
)

func hello(c web.C, w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, %s!", c.URLParams["name"])
}

func main() {
    goji.Get("/hello/:name", hello)
    goji.Serve()
}</pre>

起動して、プロセス数を確認してみます。確認としてabコマンドを利用しました。

<pre class="lang:sh decode:true " title="apache bench">$ ab -n 1000 -c 100 http://localhost:8000/hello/shamaton</pre>

同時リクエスト数を100として実行してみると、100を超えていてフルでプロセスが生成されているようです。

<pre class="lang:sh decode:true">$ ss -o  state established -np | grep "limit" | grep "8000" | wc -l
102</pre>

これだと状況によってはサーバがダウンしてしまいそうです。どうにかして接続制限数を指定できないかなと思っていたら、すでにパッケージがありました。

[GoDoc &#8211; netutil][1]

netutil.LimitListenerを利用してlistenerを作成し、設定すれば良さそうです。  
そこで下記のようにコードを変更しました。最大接続数を15にしてみます。

<pre class="lang:go decode:true" title="limit.go">package main

import (
    "fmt"
    "net/http"

    "github.com/zenazn/goji"
    "github.com/zenazn/goji/web"

    "github.com/zenazn/goji/bind"
    "golang.org/x/net/netutil"
)

func hello(c web.C, w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, %s!", c.URLParams["name"])
}

func main() {
    goji.Get("/hello/:name", hello)

    listener := netutil.LimitListener(bind.Default(), 15)
    goji.ServeListener(listener)
}</pre>

これを実行して再度abを実行してみます。

<pre class="lang:sh decode:true">$ ss -o  state established -np | grep "limit" | grep "8000" | wc -l
15</pre>

gojiでも接続数を制限することができました。フレームワークによって同じようにサクッとlistenerを設定できないものもあるっぽい・・ですかね。

以上です。

■ 参考  
<a href="http://heartbeats.jp/hbblog/2015/10/golang-limitlistener.html" target="_blank">Golangで作ったhttpdの接続数を制限してみよう</a>  
<a href="http://mattn.kaoriya.net/software/lang/go/20160713120926.htm" target="_blank">golang のサーバで帯域制限したい。</a>

[Qiita &#8211; Ab(Apache Bench)を使用した負荷テストのやり方][2]

 [1]: https://godoc.org/golang.org/x/net/netutil
 [2]: http://qiita.com/mmmm/items/f31b15b4f80427360207