---
title: '[Golang] メソッドをreflect.MethodByNameで呼んだときの性能差'
author: しゃまとん
date: 2017-05-04T03:05:34+00:00
url: /posts/144
featured_image: /images/posts/2015/11/gopher.jpg
categories:
  - go
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

ふと、golangでもこんな感じでメソッドって呼べないのかなーと調べてみたんですが、そんないい話はありませんでした。 

{{< blogcard url="http://k0kubun.hatenablog.com/entry/2014/12/06/173227" >}}

Goではreflectというパッケージでリフレクションすることができるようになっています（説明になっていない）。
そちらでやりたげなことは出来そうなんですが、reflectは重いという噂から、どうなんだろうということで簡単に比較してみました。

{{< blogcard url="https://golang.org/pkg/reflect/" >}}

golangで文字列的にメソッドを呼ぶには`reflect.MethodByName`を利用して実現できそうです。いけるかな、いけないかな。

さて、検証簡単にベンチマークをとりました。  
コードは下記です。

```go
package main

import (
    "testing"
    "reflect"
    "time"
)

var t = time.Now()
var r = reflect.ValueOf(t)

func BenchmarkCall(b *testing.B) {
    for n := 0; n < b.N; n++ {
        t.UnixNano()
    }
}

func BenchmarkReflect(b *testing.B) {
    for n := 0; n < b.N; n++ {
        r.MethodByName("UnixNano").Call(nil)
    }
}
```


実行結果はこんな感じ。  
全然速度違いますね。。

```text
BenchmarkCall-4                          2000000000           0.49 ns/op        0 B/op          0 allocs/op
BenchmarkReflect-4                       1000000          1577 ns/op         192 B/op          7 allocs/op
```

普通に使っている程度だと、使わないですが込み入ったことをするとreflectが登場してくるので使うときは気をつけないとですね。

以上です。

■ 参考

{{< blogcard url="http://ameblo.jp/principia-ca/entry-11929774278.html" >}}
{{< blogcard url="http://d.hatena.ne.jp/fbis/20060522/1148294458" >}}
