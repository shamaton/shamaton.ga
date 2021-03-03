---
title: '[Unity5.5] プロジェクトビューにアセットのファイルサイズを表示する'
author: しゃまとん
date: 2017-01-12T15:01:12+00:00
url: /posts/362
featured_image: /images/posts/2016/03/unity-logo.png
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

（久しぶりのUnityに関する記事だ(*´∀｀)）

こちらの記事ですが、[kyusyukeigo](https://twitter.com/kyusyukeigo)さんが2013からブログにて公開されている、
「プロジェクトビューにファイルサイズを表示する」という拡張エディタをUnity5.5用に書き換えたものになります。

{{< blogcard url="http://anchan828.hatenablog.jp/entry/2013/05/12/044215" >}}

ですので、5.4までは上記リンク先を参照してくださるのがよいかと思います。

関係ねぇ！とにかく表示させろい！って方はご利用くださいませ。  
こんな感じで表示されるようになります。

{{< figure src="/images/posts/2017/01/disp_size.png" >}}

書き換えた理由としては試したらエラーが出てしまって動かなくなっていたためです。  
[この辺とか][4]が理由になっていたり。とりあえず同じように表示されるようになったか・・な・・？

{{< gist shamaton 5300cc240f3c4d4b132cd9b737969523 >}}

使うときはEditorフォルダ配下に入れるようにしてくださいね〜  
以上です。

 [4]: http://tsubakit1.hateblo.jp/entry/2016/11/25/235315