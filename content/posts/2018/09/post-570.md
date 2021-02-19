---
title: '[golang] MessagePackのシリアライザ(msgpack)を作りました'
author: しゃまとん
type: post
date: 2018-09-28T13:02:59+00:00
url: /archives/570
featured_image: /wp-content/uploads/2016/12/GitHub-Mark-120px-plus.png
categories:
  - go

---
お世話になっております。  
しゃまとんです。

ちょっとしたお知らせです。  
タイトルの通りなんですが、golangで使えるMessagePackのシリアライザを作りました。  
（&#x1f389;個人的にはパンパカパーン&#x1f389;）



まず、MessagePackとはデータのシリアライズフォーマットの1つで効率の良い形式として知られています。よくJSONと比較・検証されて使われていることが多いですね。  
現在は多くの言語で使えるようになっており、言語間で何かデータのやり取りをするときに候補の1つになっています。

[公式][1]にもある通り、JSONよりも軽量で速いのが特徴です。

もちろんGo言語でもMessagePackが扱えるパッケージがすでにいくつか存在しており、使いたければ使える状況になっています。自分自身も使っていたのですが、とあるきっかけからpackageを覗いてみたところ、なんとなくもう少し速くできそうかなぁ&#8230;と思いました。

で、簡単な検証用ベンチマークコードを書いてみたところ、速くなりそうかなぁ&#8230;ということで書き始めたのが最初でした。コードを書き換えてはベンチマークを確認して&#8230;実装して&#8230;を繰り返しすすめていき、現在（2018.09時点）v1.0としてリリースできました。

エンドポイントの名称は迷ったんですが、いろいろ考えてEncode / Decodeにしました。  
呼び出しはこんな感じで。

<pre class="lang:go decode:true " title="sample.go">package main;

import (
  "github.com/shamaton/msgpack"
)

func main() {
    type Struct struct {
        String string
    }
    v := Struct{String: "msgpack"}

    d, err := msgpack.Encode(v)
    if err != nil {
        panic(err)
    }
    r := Struct{}
    err = msgpack.Decode(d, &r)
    if err != nil {
        panic(err)
    }
}</pre>

単純に使うだけならEncode/Decodeを呼び出すだけです！  
引数は標準のパッケージencoding/jsonと同じ形にしてます。（置き換えが容易かも&#8230;！）

で、肝心のパフォーマンスですが、まずはシリアライズ。  
自分が知っている範囲で他のMessagePackやJSON等の別フォーマットと比較してみます。ベンチマークはおそらくユーザーが呼び出すであろうエンドポイントでとってみました。

<pre class="lang:default mark:5,7 decode:true ">$ go test -bench CompareEncode -benchmem
goos: darwin
goarch: amd64
pkg: github.com/shamaton/msgpack_bench
BenchmarkCompareEncodeShamaton-4             1000000          1255 ns/op         320 B/op          3 allocs/op
BenchmarkCompareEncodeVmihailenco-4           300000          4645 ns/op         968 B/op         14 allocs/op
BenchmarkCompareEncodeArrayShamaton-4        1000000          1110 ns/op         256 B/op          3 allocs/op
BenchmarkCompareEncodeArrayVmihailenco-4      300000          4387 ns/op         968 B/op         14 allocs/op
BenchmarkCompareEncodeUgorji-4               1000000          1921 ns/op         986 B/op         11 allocs/op
BenchmarkCompareEncodeZeroformatter-4        1000000          1890 ns/op         744 B/op         13 allocs/op
BenchmarkCompareEncodeJson-4                  500000          3428 ns/op        1224 B/op         16 allocs/op
BenchmarkCompareEncodeGob-4                   200000         11537 ns/op        2824 B/op         50 allocs/op
BenchmarkCompareEncodeProtocolBuffer-4        500000          2338 ns/op         792 B/op         29 allocs/op
PASS
ok      github.com/shamaton/msgpack_bench   14.481s</pre>

Shamatonとついているものが今回リリースしたものになります。ArrayShamatonというのはMessagePackのフォーマットが軽量なパターンを採用しているものです（Unityでは[MessagePack-CSharp][2]でおなじみですね？）。今回のケースでは通常のEncodeでも他のパッケージよりも性能よく動作させられたっぽいです。

次にデシリアライズです。

<pre class="lang:default mark:5,7 decode:true ">$ go test -bench CompareDecode -benchmem
goos: darwin
goarch: amd64
pkg: github.com/shamaton/msgpack_bench
BenchmarkCompareDecodeShamaton-4             1000000          1393 ns/op         512 B/op          6 allocs/op
BenchmarkCompareDecodeVmihailenco-4           300000          5393 ns/op        1056 B/op         33 allocs/op
BenchmarkCompareDecodeArrayShamaton-4        2000000           990 ns/op         512 B/op          6 allocs/op
BenchmarkCompareDecodeArrayVmihailenco-4      300000          4397 ns/op         992 B/op         22 allocs/op
BenchmarkCompareDecodeUgorji-4                500000          2587 ns/op         845 B/op         12 allocs/op
BenchmarkCompareDecodeZeroformatter-4        1000000          2350 ns/op         976 B/op         29 allocs/op
BenchmarkCompareDecodeJson-4                  200000          8904 ns/op        1216 B/op         43 allocs/op
BenchmarkCompareDecodeGob-4                    50000         34805 ns/op       10172 B/op        275 allocs/op
BenchmarkCompareDecodeProtocolBuffer-4       1000000          1759 ns/op         656 B/op         19 allocs/op
PASS
ok      github.com/shamaton/msgpack_bench   16.946s</pre>

こちらも性能良く動作させられたっぽいです。ただProtocol Bufferはデータの形次第でパフォーマンス良くなることがありました（struct onlyな構成など）。まぁprotoファイルとかを準備しないといけないので、その点よいかなーと。

結果を見る感じだとArray呼び出しがパフォーマンスがやはり良いので、使えるならAsArrayを使うのが良さそうかなと思います。ベンチマークですが、こちらにコードがあるので何かあれば教えてください。



ということで、単純な呼び出しでMessagePack系では割と速いシリアライザを作ることが出来ました。よかったよかった。それと今回の実装で[zeroformatter][3]側にも反映できそうなところもわかったので、対応できればなーと思っています。

あと余談ですが、タグでシリアライズ対象の設定ができたりとか、拡張エンコード・デコードにも対応させたりもしてます。（この辺はREADMEとかExampleに書く予定です）

よかったら使ってみてください！  
以上です。

&nbsp;

<blockquote class="twitter-tweet" data-lang="ja">
  <p dir="ltr" lang="en">
    I&#8217;ve just published MessagePack Serializer(v1.0.0) for golang. Its performance is faster than the others. You can use simply. Please see and try it!! &#x1f389;&#x1f389;&#x1f389;<a href="https://t.co/X4P5qNWOzH">https://t.co/X4P5qNWOzH</a><a href="https://twitter.com/hashtag/Golang?src=hash&ref_src=twsrc%5Etfw">#Golang</a>
  </p>
  
  <p>
    — しゃまとん ʕ ◔ϖ◔ʔ (@shamaton) <a href="https://twitter.com/shamaton/status/1045659310737346561?ref_src=twsrc%5Etfw">2018年9月28日</a>
  </p>
</blockquote>

 [1]: https://msgpack.org/ja.html
 [2]: https://github.com/neuecc/MessagePack-CSharp
 [3]: https://github.com/shamaton/zeroformatter