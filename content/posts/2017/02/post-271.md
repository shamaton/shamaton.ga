---
title: '[Golang] gojiでも接続数を制限してみる'
author: しゃまとん
date: 2017-02-15T13:40:07+00:00
url: /posts/271
featured_image: /images/posts/2016/08/goji.png
categories:
  - go
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

go言語の軽量フレームワークであるgojiを触っておりまして、
ふと大量のリクエストが来た時ってプロセス生成ってどうなっているのだろう・・と思い確認してみました。

{{< blogcard url="https://github.com/zenazn/goji" >}}

とりあえずgojiをgo getして最低限動作する状態にします。

```go
package main

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
}
```

起動して、プロセス数を確認してみます。確認としてabコマンドを利用しました。

```shell
ab -n 1000 -c 100 http://localhost:8000/hello/shamaton
```

同時リクエスト数を100として実行してみると、100を超えていてフルでプロセスが生成されているようです。

```shell
ss -o  state established -np | grep "limit" | grep "8000" | wc -l
102
```

これだと状況によってはサーバがダウンしてしまいそうです。
どうにかして接続制限数を指定できないかなと思っていたら、すでにパッケージがありました。

{{< blogcard url="https://pkg.go.dev/golang.org/x/net/netutil" >}}

netutil.LimitListenerを利用してlistenerを作成し、設定すれば良さそうです。  
そこで下記のようにコードを変更しました。最大接続数を15にしてみます。

```go
package main

import (
    "fmt"
    "net/http"

    "github.com/zenazn/goji"
    "github.com/zenazn/goji/bind"
    "github.com/zenazn/goji/web"
    "golang.org/x/net/netutil"
)

func hello(c web.C, w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, %s!", c.URLParams["name"])
}

func main() {
    goji.Get("/hello/:name", hello)

    listener := netutil.LimitListener(bind.Default(), 15)
    goji.ServeListener(listener)
}
```

これを実行して再度abを実行してみます。

```shell
ss -o  state established -np | grep "limit" | grep "8000" | wc -l
15
```

gojiでも接続数を制限することができました。
フレームワークによって同じようにサクッとlistenerを設定できないものもあるっぽい・・ですかね。

以上です。

■ 参考  

{{< blogcard url="http://heartbeats.jp/hbblog/2015/10/golang-limitlistener.html" >}}
{{< blogcard url="http://mattn.kaoriya.net/software/lang/go/20160713120926.htm" >}}
{{< blogcard url="http://qiita.com/mmmm/items/f31b15b4f80427360207" >}}
