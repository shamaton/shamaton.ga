---
title: '[Golang]構造体のバイナリ化ができなかった'
author: しゃまとん
type: post
date: 2018-10-25T13:53:39+00:00
url: /archives/547
featured_image: /wp-content/uploads/2015/11/gopher.jpg
categories:
  - go

---
お世話になっております。  
しゃまとんです。

Goで構造体をバイナリ化したいなぁと思って、調べてやってみていたのですが中々うまくいかない・・・ということがあったのでメモ。

まず、単純にバイナリ化をするためには下記のような記述をすれば出来ます。

<pre class="lang:default decode:true ">package main

import (
    "encoding/binary"
    "encoding/hex"
    "os"
)

type st struct {
    Hoge int32
}

func main() {
    a := st{Hoge: 1}

    out := hex.Dumper(os.Stdout)
    defer out.Close()

    binary.Write(out, binary.LittleEndian, &a)
}
</pre>

go runすると、こんな感じですね。

<pre class="lang:default decode:true">00000000  01 00 00 00                                       |....|</pre>

これを開発中のコードで試していたのですが、変換エラーが出てしまい困っていのたですが、どうやらFixed Typeでないとダメであるということがわかりました。

簡単なところでいうと、スライスだったらstringだったりは決まった長さではないため、変換時にエラーにされてしまうようです。

自分の場合は、stringがどうやら原因だったようでできるものだと思いこんでいたのが、ハマりポイントでした。Unity（C#）だと出来るので、まさかでした。

あと普段Goを書いていると使っているintもダメですね。これは環境によって32/64bitが変わるそうなので、変換時にエラーになってしまいます。

<pre class="lang:default decode:true ">// 下記は変換エラー
type st struct {
    A []int32
    B string
    C int
}</pre>

どうしてもバイナリで扱いたい場合はシリアライザ等を使ってやるほうが良さそうですね。  
以上です。

■ 参考  
[Go言語でシンプルに構造体⇔バイナリの相互変換][1]  
<a href="https://github.com/golang/go/issues/13156" target="_blank" rel="noopener">encoding/binary: Write() struct int type no output.</a>

&nbsp;

 [1]: https://qiita.com/castaneai/items/2441536860e624d3f8a0