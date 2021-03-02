---
title: '[golang]Go1.6で正式採用されたvendorの参照の挙動をみてみた'
author: しゃまとん
type: post
date: 2016-07-08T14:59:50+00:00
url: /posts/223
featured_image: /images/posts/2016/07/vendor.png
categories:
  - go
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

先日、Go1.5から1.6にアップデートしたときのあれこれという記事を書きまして、vendorがいいねと思っていたのですが、どんな仕組みで参照しているんだろうともやもやしたので挙動を少し確認してみました。

<blockquote class="wp-embedded-content">
  <p>
    <a href="http://shamaton.orz.hm/blog/posts/217">[golang]Go1.5から1.6に更新した際のあれこれ（IntelliJとか）</a>
  </p>
</blockquote>



ちなみに検証が結構ながいので、先にまとめっぽことを書いておくと&#8230;

* * *

${GOPATH}/src/{package}内に存在するコードhoge.goのビルド時に

、自分の位置からvendorを検索し、見つからなかったら上位層でのvendorを探す。  
最終的にはsrc/vendorまで検索する。

ただし、検索は他のGOPATHや上位層に存在する他ディレクトリまでは見に行かない。

また、src直下にコードをおいてbuildしてもsrc/vendorは参照しようとしない。（そもそもそこにコードは配置される想定ではないと思われる）

* * *

かなと思います。  
では以下検証です。

わかっていることとして、vendorフォルダがあると配下のパッケージが簡単にimportできるということだったので、下記のようにディレクトリとコードを配置しました。色々なところに空のvendorフォルダを作成してどうなるか確認します。

[<img src="http://shamaton.orz.hm/blog/images/posts/2016/06/tree.png" alt="tree" width="229" height="530" class="aligncenter size-full wp-image-224" />][1]

その前に、コードですが以下の様な感じです。vendor配下にあるであろうprintパッケージを呼び出すようなテストコードです。

■ test.go

<pre class="lang:default decode:true " title="test.go">package main

import "print"

func main() {
  print.Hello()
}</pre>

■ vendor/print/print.go

<pre class="lang:default decode:true " title="vendor/print/print.go">package print

import "fmt"

func Hello() {
  fmt.Println("hello!!")
}</pre>

■ 何もない状態  
**1. go run src/hoge/fuga/test.go（GOPATH未定義）**  
GOPATHが無いので、エラーになります。vendorに関する表示もなし。

[<img src="http://shamaton.orz.hm/blog/images/posts/2016/06/vendor_check_1.png" alt="vendor_check_1" width="428" height="59" class="aligncenter size-full wp-image-225" />][2]

そこでGOPATHを下記のように設定します。

<pre class="brush: bash; gutter: true">export GOPATH=~/vendor_check</pre>

**2. go run src/hoge/fuga/test.go**  
実行すると、vendorが表示されるようになります。ただpiyoは検索対象にはならないようです。どうやら実行するファイルから上に辿って検索するっぽい。

[<img src="http://shamaton.orz.hm/blog/images/posts/2016/06/vendor_check_2.png" width="528" height="99" class="aligncenter wp-image-236 size-full" />][3]

**3. go run src/test.go**  
src/vendorが表示されるのかと思ったら、何もでませんでした。

[<img src="http://shamaton.orz.hm/blog/images/posts/2016/06/vendor_check_3.png" width="451" height="60" class="aligncenter wp-image-237 size-full" />][4]

**4. go run lib/src/hoge/fuga/test.go**  
そもそも、GOPATHの定義外なのでダメなようです。

[<img src="http://shamaton.orz.hm/blog/images/posts/2016/06/vendor_check_4.png" width="432" height="58" class="aligncenter wp-image-238 size-full" />][5]

そこでGOPATHを追加します。

<pre class="brush: bash; gutter: true">export GOPATH=$GOPATH:~/vendor_check/lib</pre>

**5. go run lib/src/hoge/fuga/test.go**  
vendorの表示がでるようになりました。こちらではlib/src/vendorまで見ているようです。src/は見ないようですね。

[<img src="http://shamaton.orz.hm/blog/images/posts/2016/06/vendor_check_5.png" width="536" height="98" class="aligncenter wp-image-239 size-full" />][6]

**6. go run lib/src/test.go**  
src/test.goと同じでvendorは出ません。

[<img src="http://shamaton.orz.hm/blog/images/posts/2016/06/vendor_check_5.5.png" width="425" height="71" class="aligncenter wp-image-240 size-full" />][7]

**7. go run lib/hoge/fuga/test.go**  
やはり、どこかしらのsrc配下にないとダメなようですね。

[<img src="http://shamaton.orz.hm/blog/images/posts/2016/06/vendor_check_6.png" width="421" height="70" class="aligncenter wp-image-241 size-full" />][8]

**8. go run test.go**  
当然なにも出ません。

[<img src="http://shamaton.orz.hm/blog/images/posts/2016/06/vendor_check_7.png" width="425" height="73" class="aligncenter wp-image-242 size-full" />][9]

全体的に確認したのでprintを有効にします

<pre class="brush: bash; gutter: true">mv src/vendor/_print src/vendor/print</pre>

**9. go run src/hoge/fuga/test.go**  
helloと表示されました。ちゃんと参照できていますね。

[<img src="http://shamaton.orz.hm/blog/images/posts/2016/06/vendor_check_8.png" alt="vendor_check_8" width="211" height="30" class="size-full wp-image-233 aligncenter" />][10]

**10. go run src/test.go**  
こちらは参照できませんでした。src直下は想定外っぽい・・・？

[<img src="http://shamaton.orz.hm/blog/images/posts/2016/06/vendor_check_9.png" width="422" height="75" class="aligncenter wp-image-235 size-full" />][11]

src直下は利用するべきではないようですね。  
実行できるっぽいlib/src配下もダメなよう。（vendor配下に配置すればいけます）  
挙動からするとGOPATH/src配下で自分が関係している上位層のvendorまで探しに行くようです。

これからするとGOPATHは複数にせず、1つのsrc内で完結させるのがよさそうですね。  
以上です。

 [1]: http://shamaton.orz.hm/blog/images/posts/2016/06/tree.png
 [2]: http://shamaton.orz.hm/blog/images/posts/2016/06/vendor_check_1.png
 [3]: http://shamaton.orz.hm/blog/images/posts/2016/06/vendor_check_2.png
 [4]: http://shamaton.orz.hm/blog/images/posts/2016/06/vendor_check_3.png
 [5]: http://shamaton.orz.hm/blog/images/posts/2016/06/vendor_check_4.png
 [6]: http://shamaton.orz.hm/blog/images/posts/2016/06/vendor_check_5.png
 [7]: http://shamaton.orz.hm/blog/images/posts/2016/06/vendor_check_5.5.png
 [8]: http://shamaton.orz.hm/blog/images/posts/2016/06/vendor_check_6.png
 [9]: http://shamaton.orz.hm/blog/images/posts/2016/06/vendor_check_7.png
 [10]: http://shamaton.orz.hm/blog/images/posts/2016/06/vendor_check_8.png
 [11]: http://shamaton.orz.hm/blog/images/posts/2016/06/vendor_check_9.png