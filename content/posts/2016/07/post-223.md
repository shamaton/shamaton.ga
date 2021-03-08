---
title: '[golang]Go1.6で正式採用されたvendorの参照の挙動をみてみた'
author: しゃまとん
date: 2016-07-08T14:59:50+00:00
url: /posts/223
featured_image: /images/posts/2016/07/vendor.png
categories:
  - go
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

先日、Go1.5から1.6にアップデートしたときのあれこれという記事を書きまして、
vendorがいいねと思っていたのですが、どんな仕組みで参照しているんだろうともやもやしたので挙動を少し確認してみました。

{{< blogcard url="/posts/217" >}}

ちなみに検証が結構ながいので、先にまとめっぽことを書いておくと...

* * *

${GOPATH}/src/{package}内に存在するコードhoge.goのビルド時に、自分の位置からvendorを検索し、
見つからなかったら上位層でのvendorを探す。最終的にはsrc/vendorまで検索する。

ただし、検索は他のGOPATHや上位層に存在する他ディレクトリまでは見に行かない。

また、src直下にコードをおいてbuildしてもsrc/vendorは参照しようとしない。
（そもそもそこにコードは配置される想定ではないと思われる）

* * *

かなと思います。  
では以下検証です。

わかっていることとして、vendorフォルダがあると配下のパッケージが簡単にimportできるということだったので、
下記のようにディレクトリとコードを配置しました。色々なところに空のvendorフォルダを作成してどうなるか確認します。

{{< figure src="/images/posts/2016/06/tree.png" >}}

その前に、コードですが以下の様な感じです。vendor配下にあるであろうprintパッケージを呼び出すようなテストコードです。

■ test.go

```go
package main

import "print"

func main() {
  print.Hello()
}
```

■ vendor/print/print.go

```go
package print

import "fmt"

func Hello() {
  fmt.Println("hello!!")
}
```

■ 何もない状態  
**1. go run src/hoge/fuga/test.go（GOPATH未定義）**  
GOPATHが無いので、エラーになります。vendorに関する表示もなし。

{{< figure src="/images/posts/2016/06/vendor_check_1.png" >}}

そこでGOPATHを下記のように設定します。

```text
export GOPATH=~/vendor_check
```

**2. go run src/hoge/fuga/test.go**  
実行すると、vendorが表示されるようになります。ただpiyoは検索対象にはならないようです。どうやら実行するファイルから上に辿って検索するっぽい。

{{< figure src="/images/posts/2016/06/vendor_check_2.png" >}}

**3. go run src/test.go**  
src/vendorが表示されるのかと思ったら、何もでませんでした。

{{< figure src="/images/posts/2016/06/vendor_check_3.png" >}}

**4. go run lib/src/hoge/fuga/test.go**  
そもそも、GOPATHの定義外なのでダメなようです。

{{< figure src="/images/posts/2016/06/vendor_check_4.png" >}}

そこでGOPATHを追加します。

<pre class="brush: bash; gutter: true">export GOPATH=$GOPATH:~/vendor_check/lib</pre>

**5. go run lib/src/hoge/fuga/test.go**  
vendorの表示がでるようになりました。こちらではlib/src/vendorまで見ているようです。src/は見ないようですね。

{{< figure src="/images/posts/2016/06/vendor_check_5.png" >}}

**6. go run lib/src/test.go**  
src/test.goと同じでvendorは出ません。

{{< figure src="/images/posts/2016/06/vendor_check_5.5.png" >}}

**7. go run lib/hoge/fuga/test.go**  
やはり、どこかしらのsrc配下にないとダメなようですね。

{{< figure src="/images/posts/2016/06/vendor_check_6.png" >}}

**8. go run test.go**  
当然なにも出ません。

{{< figure src="/images/posts/2016/06/vendor_check_7.png" >}}

全体的に確認したのでprintを有効にします

<pre class="brush: bash; gutter: true">mv src/vendor/_print src/vendor/print</pre>

**9. go run src/hoge/fuga/test.go**  
helloと表示されました。ちゃんと参照できていますね。

{{< figure src="/images/posts/2016/06/vendor_check_8.png" >}}

**10. go run src/test.go**  
こちらは参照できませんでした。src直下は想定外っぽい・・・？

{{< figure src="/images/posts/2016/06/vendor_check_9.png" >}}

src直下は利用するべきではないようですね。  
実行できるっぽいlib/src配下もダメなよう。（vendor配下に配置すればいけます）  
挙動からするとGOPATH/src配下で自分が関係している上位層のvendorまで探しに行くようです。

これからするとGOPATHは複数にせず、1つのsrc内で完結させるのがよさそうですね。  
以上です。
