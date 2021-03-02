---
title: '[golang] zeroformatterを作成してみる'
author: しゃまとん
type: post
date: 2016-12-21T15:14:56+00:00
url: /posts/249
featured_image: /images/posts/2016/12/GitHub-Mark-120px-plus.png
categories:
  - go
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

タイトルの通りですが、ZeroFormatterをGo言語で使えるようにしたかったので、作ってみています。



そもそもZeroFormatterとは何ぞやというところですが、infinityにFastなDeserializerです。  
C#でserialize/deserializeを爆速で行えるようで、これはすごい・・・！と思った次第です。

詳細はgithubで。



そんなZeroFormatterはもちろんUnityでも使用することができます（zfc等の若干の利用手順があるっぽい）。作者の[neuecc][1]さんが所属されている[グラニ][2]さんで使われているはずなので、実績はこれから積まれていくでしょう。

現状、多言語対応は進行中（有志によって）なので、他言語で使いたい場合は待つか、フォーマットが公開されているので実装する必要があります。  
golangでも誰かやってくれないかなーと思っていたのですが、いなかったのでやり始めてみたのがきっかけです。使えたら面白いんじゃないか的な。

対応状況はリポジトリを見ていただくのがいいのですが、ざっくりいうとプリミティブな型に関してはある程度対応するところまで出来ました。それと少しばかりのチューニングをし、ざっくりとしたベンチマークを取ってみました。

<pre class="lang:go decode:true" title="BenchMarkStruct">type BenchChild struct {
    Int    int
    String string
}

type BenchMarkStruct struct {
    Int    int
    Uint   uint
    Float  float32
    Double float64
    Bool   bool
    String string
    Array  []int
    Map    map[string]int
    Child  BenchChild
}

var s = BenchMarkStruct{
    Int:    -123,
    Uint:   456,
    Float:  1.234,
    Double: 6.789,
    Bool:   true,
    String: "this is text.",
    Array:  []int{1, 2, 3, 4, 5, 6, 7, 8, 9},
    Map:    map[string]int{"this": 1, "is": 2, "map": 3},
    Child:  BenchChild{Int: 123456, String: "this is struct of child"},
}</pre>

&nbsp;

比較は日本でよく使われているっぽいMessagePackと。  
ちなみにパッケージは[vmihailenco/msgpack][3]を使いました。

<pre class="lang:default decode:true" title="benchmark">BenchmarkPackZeroformatter-4          500000          2919 ns/op         760 B/op         14 allocs/op
BenchmarkPackMsgpack-4                200000          6416 ns/op        1000 B/op         14 allocs/op
BenchmarkUnpackZeroformatter-4        300000          3482 ns/op         976 B/op         29 allocs/op
BenchmarkUnpackMsgpack-4              200000          8353 ns/op        1024 B/op         33 allocs/op</pre>

結果としてはMsgPackよりも処理速度はおよそ半分ぐらいになりました。今後の更新で変わるかもしれないのですが。。。

今後としては、本家がデシリアライズしない仕組みになっているので、こちらでいい感じに&#8230;は難しいので明示的にでもやれるようにしようかなと思っています。それとタグ指定で構造体の特定のプロパティのみ評価対象にできるようにしたいと思っています。

それとmapのシリアライズがMsgPackに比べて遅い感じがあるので、改善する予定です。  
ということで報告みたいな感じですが、以上です。

 [1]: https://twitter.com/neuecc
 [2]: http://grani.jp/
 [3]: https://github.com/vmihailenco/msgpack