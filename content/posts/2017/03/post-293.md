---
title: '[Golang] EchoのUseの挙動について'
author: しゃまとん
type: post
date: 2017-03-07T14:12:52+00:00
url: /posts/293
featured_image: /images/posts/2016/09/logo.png
categories:
  - go
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

Go言語では多数のウェブフレームワークが公開されていますが、最近ホットそうなフレームワークとしてEchoというものがあります。

[Echo &#8211; A fast and unfancy micro web framework for Go.][1]

ちょっと見てみると、エンドポイントに渡す引数がシンプルだったり、公式で使い方を記載してくれていたりします。

また、スター数も5000を越えていて、支持されているのが伺えますね。（2017.03時点）

そこで、[goji][2]を使って触ってたものを置き換えてみることにしたのですが、Use関数の挙動が微妙に違っていたのでメモです。

Use関数とはリクエストが飛んできたときに事前に決められた共通の処理をさせたい場合に利用する機能です。今回はEchoを使って下記のように簡単な返答をするコードを用意しました。



アクセス時に１，２，３と表示するような機能をいれました。  
これをターミナルから実行し、各URLへアクセスしてみます。

■ http://localhost:8080/nice/middle/hoge  
１，２しか表示されません。

[<img src="https://shamaton.orz.hm/blog/images/posts/2016/09/hoge.png" alt="hoge" width="121" height="57" class="aligncenter size-full wp-image-297" />][3]

■ http://localhost:8080/nice/middle/fuga  
１，２，３と表示されます。

[<img src="https://shamaton.orz.hm/blog/images/posts/2016/09/fuga.png" alt="fuga" width="133" height="100" class="aligncenter size-full wp-image-298" />][4]

このように、Useとルーティングの順序が違うとUseしたけど呼ばれません。

導入時にUseの呼び出しがルーティングの後になっていて、呼ばれなくてハマっていました。gojiのでは順序は気にしなくても良かった気がします。フレームワークによって若干挙動は変わったりするので、注意ですね。

以上です。

 [1]: https://github.com/labstack/echo
 [2]: https://github.com/zenazn/goji
 [3]: https://shamaton.orz.hm/blog/images/posts/2016/09/hoge.png
 [4]: https://shamaton.orz.hm/blog/images/posts/2016/09/fuga.png