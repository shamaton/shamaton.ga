---
title: '[Unity] Nostalgia2 – 2Dマップエディタを触ってみた'
author: しゃまとん
type: post
date: 2016-11-19T13:49:27+00:00
url: /posts/336
featured_image: /images/posts/2016/11/nostalgia.png
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

（本記事はケットシーウェアさんの[キャンペーン][1]に関連した記事になります）

今回はUnityのアセットを紹介したいと思います。  
[Nostalgia2][2]という2Dマップを作ることができるエディタ拡張のアセットです。

まだあまり深く使えていないので、初心者がタイルマップエディタを触ってみて感じたぐらいのニュアンスで受け取って頂けるとよいかと思います。



特徴としてRPGツクールやウディタのオートタイル規格素材が使えます。  
自分はこの辺の仕組みを全く知らなかったのですが、ルールに沿って素材作成をされたものを使うと配置時にいい感じに設置してくれました。

オートタイルって何ぞやという方は、この辺を見ると理解できそうです。  
[ウディタ &#8211; 素材規格][3]  
[RPGツクール &#8211; 素材規格][4]

使い方はアセット製作者のcaitsithwareさんがムービーで紹介してくださっています。



[チュートリアル][5]は2つあってどっちも丁寧に説明されているので見ると理解が進むと思います。

ここまでふむふむとお勉強をして素材を用意すれば、  
すぐにマップ配置をすることができるようになりました。素材はねくらさんの[マップチップ][6]を利用させていただいています。

[<img src="https://shamaton.orz.hm/blog/images/posts/2016/11/nostalgia_1.gif" alt="nostalgia_1" width="322" height="321" class="aligncenter size-full wp-image-337" />][7]

ざっくり使い方な手順はこんな感じでしょうか。

  1. 素材からタイル情報を作成
  2. シーンにマップオブジェクトを配置後、1を利用して配置
  3. レイヤー状にしたい場合はマップオブジェクトを複数おく

アセットにはアクションゲームのサンプルがあるのですが、2Dのアクションゲームなら考えることなくサクサクと作っていけそうです。  
（サンプルが十分に遊べるクオリティです・・！）

とりあえず触ってみたいと思った方は[Webデモ][8]が用意されているので、  
検討されてみてはいかがでしょうか。

また色々と記事にできることがあれば載せたいと思います。  
以上です。

■ 参考  
[ケットシーウェア][9]

&nbsp;

 [1]: http://caitsithware.com/wordpress/archives/1989
 [2]: https://www.assetstore.unity3d.com/#!/content/70610?aid=1100lGtC
 [3]: http://www.silversecond.com/WolfRPGEditor/Help/06material.html
 [4]: https://tkool.jp/products/rpgvx/material
 [5]: http://caitsithware.com/wordpress/assetstore/nostalgia/tutorial
 [6]: http://nekuramap.blog.fc2.com/blog-entry-2.html
 [7]: https://shamaton.orz.hm/blog/images/posts/2016/11/nostalgia_1.gif
 [8]: http://caitsithware.com/wordpress/assetstore/nostalgia/demo
 [9]: http://caitsithware.com/wordpress/