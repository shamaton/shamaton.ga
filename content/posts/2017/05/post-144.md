---
title: '[Golang] メソッドをreflect.MethodByNameで呼んだときの性能差'
author: しゃまとん
type: post
date: 2017-05-04T03:05:34+00:00
url: /archives/144
featured_image: /wp-content/uploads/2015/11/gopher.jpg
categories:
  - go
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

ふと、golangでも[こんな感じ][1]でメソッドって呼べないのかなーと調べてみたんですが、そんないい話はありませんでした。  
golangではreflectというパッケージでリフレクションすることができるようになっています（説明になっていない）。そちらでやりたげなことは出来そうなんですが、reflectは重いという噂から、どうなんだろうということで簡単に比較してみました。

golangで文字列的にメソッドを呼ぶにはreflect.MethodByNameを利用して実現できそうです。いけるかな、いけないかな。

さて、検証簡単にベンチマークをとりました。  
コードは下記です。

<pre class="lang:go decode:true" title="bench.go">package main

import (
    "testing"
    "reflect"
    "time"
)

var t = time.Now()
var r = reflect.ValueOf(t)

func BenchmarkCall(b *testing.B) {
    for n := 0; n &lt; b.N; n++ {
        t.UnixNano()
    }
}

func BenchmarkReflect(b *testing.B) {
    for n := 0; n &lt; b.N; n++ {
        r.MethodByName("UnixNano").Call(nil)
    }
}</pre>

実行結果はこんな感じ。  
全然速度違いますね。。

<pre class="lang:default decode:true ">BenchmarkCall-4                          2000000000           0.49 ns/op        0 B/op          0 allocs/op
BenchmarkReflect-4                       1000000          1577 ns/op         192 B/op          7 allocs/op</pre>

普通に使っている程度だと、使わないですが込み入ったことをするとreflectが登場してくるので使うときは気をつけないとですね。

以上です。

■ 参考

 <a href="http://k0kubun.hatenablog.com/entry/2014/12/06/173227" target="_blank" rel="noopener noreferrer">RubyistのためのGolangメタプログラミング</a>

<a href="http://ameblo.jp/principia-ca/entry-11929774278.html" target="_blank" rel="noopener noreferrer">golangのある生活</a>

&nbsp;

 [1]: http://d.hatena.ne.jp/fbis/20060522/1148294458