---
title: '[Go] MessagePackのコードを生成するパッケージ(msgpackgen)を作りました'
author: しゃまとん
date: 2021-03-17T10:23:09+00:00
url: /posts/676
is_comment_form_freeze:
  - on
comment_form_freeze_message:
  - ' '
categories:
  - go

---
 お世話になっております。  
しゃまとんです。

GoのMessagePackパッケージ、[msgpackgen](https://github.com/shamaton/msgpackgen)をリリースしました。
{{< blogcard url="https://github.com/shamaton/msgpackgen" >}}

以下、作った背景と使い方を書いていきます。

### なぜ作ったの？
これは前段の話がありまして、実は私自身も後続ではありますが、GoでMessagePackのシリアライザを作ってリリースしています（[msgpack](https://github.com/shamaton/msgpack)）。
速いは正義ということで、他ライブラリよりも速く実装できたので私が関与したサービスで使っていたのですが

>「ん〜、データをクライアント（フロントエンド）に返すときって、structに詰めた明瞭な型たち（intとかstringとか）を使うだろうし、それって専用のコードある方がより速く処理できるよな〜」
> 
> 「例えばAPIサーバで、返すべきデータ定義が決まっているものだったら、それを知っているコードのほうが速く処理できるはずだよな〜」

と漠然と思っていました。そこから幾月か経ち、こんな記事を目にしました。
[entityからコード自動生成した話 - Money Forward Kessai TECH BLOG](https://tech.mfkessai.co.jp/2019/09/ebgen/)
{{< blogcard url="https://tech.mfkessai.co.jp/2019/09/ebgen/" >}}
あ、これ使ったら前考えたこと出来るかも...と思い、作り始め出来上がったのがmsgpackgenです。

### ベンチマーク
今はこんな感じになっていて、`ShamatonGen`とsuffixについているものが今回紹介しているものになります。知っている範囲でGo製のMessagePackシリアライザと公式の[Protobuf](https://github.com/golang/protobuf)や[Json](https://golang.org/pkg/encoding/json/)をいれて比較してみました。今の所、一番速そうです...！
![a](https://user-images.githubusercontent.com/4637556/107843994-23439e00-6e13-11eb-9303-296be7c24282.png)

尚、結果が近しい`tinylib`というパッケージもCode Generator付きのシリアライザです。実装の際に比較対象にしていましたが、そちらよりも簡単にコード生成できるような仕組みにするのも心がけました。
ということで、ぜひお試しいただきたい...!!

### 使い方 - コード生成
まずはコード生成していきます。Goには`go generate`というコード生成を行う仕組みがあり、ファイルに決められたコメントを記載しておくことで、効力を発揮してくれます。例えばmain.goに

```go
//go:generate go run github.com/shamaton/msgpackgen
```
と記載し、シェルで
```shell
go generate
```
とすると、main.goがあるディレクトリを含め、再帰的に配下のディレクトリまでコード生成可能な構造体（struct）を検出しコード生成を行います。出力ファイルは１ファイルにまとめられ、デフォルトのファイル名は`resolver.msgpackgen.go`で生成されます。

生成例:[これ](https://github.com/shamaton/msgpack_bench/blob/master/struct.go)が[こう](https://github.com/shamaton/msgpack_bench/blob/master/resolver.msgpackgen.go)なります

### 使い方 - シリアライズ
シリアライズを有効にするために、アプリケーションの開始段階などで有効にするメソッドを呼んでおきます。例えばmain.goなどで...

```go
func main() {
	// 登録 (resolver.msgpackgen.goにある)
	RegisterGeneratedResolver()
	
	// http.Serveするなど
	service()
}
```

シリアライズ/デシリアライズは`json`パッケージのように`Marshal`/`Unmarshal`が用意されています。importに`github.com/shamaton/msgpackgen/msgpack`を追加してください。
```go
func service() {
    v := RegisteredStruct{}
    b, err := msgpack.Marshal(v)
    if err != nil {
        panic(err)
    }
    
    var vv RegisteredStruct
    err = msgpack.Unmarshal(b, &vv)
    if err != nil {
        panic(err)
    }
}
```

もし、`resolver.msgpackgen.go`に登録されていないデータが引数に渡された場合はこちらの[msgpack](https://github.com/shamaton/msgpack)が呼ばれます。

### まとめ
msgpackgenを紹介させていただきました。より細かいことは[README](https://github.com/shamaton/msgpackgen/blob/main/README.md)に書いてありますが、基本は説明したものだけで機能するようになると思います。試していただけると嬉しいです！

ぜひともよろしくお願いします！