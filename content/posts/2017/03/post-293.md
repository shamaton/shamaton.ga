---
title: '[Golang] EchoのUseの挙動について'
author: しゃまとん
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

{{< blogcard url="https://github.com/labstack/echo" >}}

ちょっと見てみると、エンドポイントに渡す引数がシンプルだったり、公式で使い方を記載してくれていたりします。

また、スター数も5000を越えていて、支持されているのが伺えますね。（2017.03時点）

そこで、gojiを使って触ってたものを置き換えてみることにしたのですが、Use関数の挙動が微妙に違っていたのでメモです。

{{< blogcard url="https://github.com/zenazn/goji" >}}

Use関数とはリクエストが飛んできたときに事前に決められた共通の処理をさせたい場合に利用する機能です。
今回はEchoを使って下記のように簡単な返答をするコードを用意しました。

{{< gist shamaton 479c6fd1780b12b8e108f003d4103371 >}}

アクセス時に１，２，３と表示するような機能をいれました。  
これをターミナルから実行し、各URLへアクセスしてみます。

■ http://localhost:8080/nice/middle/hoge  
１，２しか表示されません。

{{< figure src="/images/posts/2016/09/hoge.png"  >}}

■ http://localhost:8080/nice/middle/fuga  
１，２，３と表示されます。

{{< figure src="/images/posts/2016/09/fuga.png"  >}}

このように、Useとルーティングの順序が違うとUseしたけど呼ばれません。

導入時にUseの呼び出しがルーティングの後になっていて、呼ばれなくてハマっていました。
gojiのでは順序は気にしなくても良かった気がします。フレームワークによって若干挙動は変わったりするので、注意ですね。

以上です。
