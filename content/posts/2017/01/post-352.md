---
title: '[golang] zeroformatterでdelay_deserializeできるようにしてみました'
author: しゃまとん
type: post
date: 2017-01-06T15:22:32+00:00
url: /posts/352
featured_image: /images/posts/2016/12/GitHub-Mark-120px-plus.png
categories:
  - go
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

去年末にgolangでzeroformatterを作り始めたのを記事にしました。ありがたいことに、いいねをしてくださっていて嬉しい限りです。  
今回はちょっとした進捗報告的な感じです。

<http://shamaton.orz.hm/blog/posts/249>

今回は前回の記事で課題にしていた、遅延してシリアライズするというものに対応してみました。どうやればいいかなーともやもや考えていたのですが、一旦やってみるかという感じで実装しました。

DelayDeserializeを呼ぶと、遅延評価用のデシリアライザを生成して返します。使えるのはindexが存在するデータを想定していて、golangではstructを利用することにしています。対象のデータが大きいときに後から評価したい場合に有効かもしれません。

READMEに簡単に記載しているのですが、下記みたいな感じで使います。  
delay deserializeという英語が正しいのか謎です。

<pre class="lang:go decode:true " title="delay">package main;

import (
  "github.com/shamaton/zeroformatter"
  "log"
)

func how_to_use(b []byte) {
    type Struct struct {
        String string
    }

    r := Struct{}
    dds, _ := zeroformatter.DelayDeserialize(&r, b)

    // 要素を渡してデシリアライズ
    if err := dds.DeserializeByElement(&r.String); err != nil {
        log.Fatal(err)
    }

    // もしくはインデックスを指定してデシリアライズ
    if err := dds.DeserializeByIndex(0); err != nil {
      log.Fatal(err)
    }
}</pre>

ちょっとずつ多言語対応拡がってるみたいですね！  
対応したくさせるzeroformatterすごいです。

次はタグ指定できるようにしてみるかな。  
あ、知見・ご意見・まさかり募集しておりますので、是非ご連絡くださいませ！

リポジトリはこちら



以上です。