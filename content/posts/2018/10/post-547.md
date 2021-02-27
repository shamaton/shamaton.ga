---
title: '[Golang]構造体のバイナリ化ができなかった'
author: しゃまとん
date: 2018-10-25T13:53:39+00:00
url: /posts/547
featured_image: /images/posts/2015/11/gopher.jpg
categories:
  - go

---
お世話になっております。  
しゃまとんです。

Goで構造体をバイナリ化したいなぁと思って、調べてやってみていたのですが中々うまくいかない・・・
ということがあったのでメモ。

まず、単純にバイナリ化をするためには下記のような記述をすれば出来ます。

```go
package main
     
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
```

go runすると、こんな感じですね。

```text
00000000  01 00 00 00                                       |....|
```

これを開発中のコードで試していたのですが、変換エラーが出てしまい困っていのたですが、
どうやらFixed Typeでないとダメであるということがわかりました。

簡単なところでいうと、スライスだったらstringだったりは決まった長さではないため、変換時にエラーにされてしまうようです。

自分の場合は、stringがどうやら原因だったようでできるものだと思いこんでいたのが、
ハマりポイントでした。Unity（C#）だと出来るので、まさかでした。

あと普段Goを書いていると使っているintもダメですね。
これは環境によって32/64bitが変わるそうなので、変換時にエラーになってしまいます。

```go
// 下記は変換エラー
type st struct {
    A []int32
    B string
    C int
}
```

どうしてもバイナリで扱いたい場合はシリアライザ等を使ってやるほうが良さそうですね。  
以上です。

■ 参考  

{{< blogcard url="https://qiita.com/castaneai/items/2441536860e624d3f8a0">}}

{{< blogcard url="https://github.com/golang/go/issues/13156">}}
