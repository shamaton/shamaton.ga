---
title: '[cocos2dx]内部でテクスチャを作成して、クリッピングする'
author: しゃまとん
type: post
date: 2015-03-02T09:46:29+00:00
url: /posts/73
featured_image: /images/posts/2015/08/Icon-cocos.png
categories:
  - cocos2dx
  - プログラミング関連

---
お世話になっております。

しゃまとんです。

cocos2dxをいじっていて、画面の一定領域を流れるメッセージを作りたいなぁと思い、クリッピング処理はできないだろうかと模索していたときのメモです。  
この時は横に流れるメッセージを作ろうとしていました。

cocosのバージョンは3.2です。

<!--more-->

クリッピングのやり方に関するwebページはいろいろと出てくるのですが、マスク用の画像を用意して、クリッピング処理するようなやり方だったのですが、これのためだけに画像用意するのもなぁ・・・と思い調べていたら、プログラムでスプライトを合成して、一枚のスプライトを生成できるみたいなので、利用してみました。

RenderTextureを利用して、テクスチャを生成し、スプライトを作成します。  
流れるメッセージはそのままLabelを利用し、クリッピング処理をして、addChildします。

下記サンプルコードです。作成しているSceneから呼び出せば、表示される(はず)と思います。

<pre class="lang:default decode:true brush: cpp; gutter: true ">// ヘッダへの定義も忘れずに
void HelloWorld::_clippingTest() {

    auto layer_size = this-&gt;getContentSize();
    auto pos_y = layer_size.height - 50.0f;

    auto font_size = 30;

    auto label = Label::createWithSystemFont("abcdefghijklmnopqrstuvwxvz", "Arial", font_size);
    auto init_pos = Vec2(layer_size.width * 3/4, pos_y);
    label-&gt;setPosition(init_pos);
    label-&gt;setAnchorPoint(Vec2(0, 0.5));

    // 説明文を右から左に流す
    auto move = MoveTo::create(12.0f, Vec2(layer_size.width/4 - label-&gt;getContentSize().width, pos_y));
    auto res  = MoveTo::create(0.0f, init_pos);
    auto seq  = RepeatForever::create(Sequence::create(move, res, nullptr));
    label-&gt;runAction(seq);

    // クリッピングマスク用のテクスチャ作成
    // NOTE:addChildしないものはretainする
    auto render_tex = RenderTexture::create(layer_size.width, layer_size.height);
    render_tex-&gt;retain();
    {
        render_tex-&gt;begin();

        // 表示しない領域(透明画像)
        auto invisible = Sprite::create();
        invisible-&gt;setTextureRect(Rect(0.0f ,0.0f ,layer_size.width, font_size));
        invisible-&gt;setColor(Color3B::GREEN);
        invisible-&gt;setOpacity(0);
        invisible-&gt;setPosition(layer_size.width/2, layer_size.height/2);
        invisible-&gt;retain();

        // 表示する領域(黒)
        auto visible = Sprite::create();
        visible-&gt;setTextureRect(Rect(0.0f ,0.0f ,layer_size.width/4, font_size));
        visible-&gt;setColor(Color3B::BLACK);
        visible-&gt;setPosition(layer_size.width/2, layer_size.height/2);
        visible-&gt;retain();

        // render_texに焼き付ける
        invisible-&gt;visit();
        visible-&gt;visit();

        render_tex-&gt;end();
    }
    // 生成したテクスチャでスプライト作成
    auto stencil = Sprite::createWithTexture(render_tex-&gt;getSprite()-&gt;getTexture());
    stencil-&gt;setPosition(layer_size.width/2, pos_y);

    // クリッピング処理
    auto clipping = ClippingNode::create();
    clipping-&gt;setAnchorPoint(Vec2(0, 0));
    clipping-&gt;setPosition(0, 0);

    // くり抜く設定
    clipping-&gt;setStencil(stencil);
    clipping-&gt;setInverted(false); // 変数visibleの部分を可視化する
    clipping-&gt;setAlphaThreshold(0.01f);

    // クリッピングして表示したいもの
    clipping-&gt;addChild(label);
    this-&gt;addChild(clipping, 100);
}</pre>

こちらのようにクリッピングされるかと思います。

<p style="text-align: center;">
  <a href="https://shamaton.orz.hm/blog/images/posts/2015/03/cliping_test.png"><img src="https://shamaton.orz.hm/blog/images/posts/2015/03/cliping_test.png" alt="clipping_test" width="469" height="307" class="aligncenter" /></a>
</p>

ちなみにAndroidではクリッピング処理がうまくいかない場合があるようですので、下記のサイトを参考にさせていただきました。

<a href="http://anz-note.tumblr.com/post/90781838271/cocos2dx-clippingnode-for-android" target="_blank" rel="noopener">cocos2dxのClippingNodeが変になったときの対処(for android) (｡･ω･｡)</a>

<a href="http://hamken100.blogspot.jp/2014/08/cocos2d-x-androidstencil.html" target="_blank" rel="noopener">ハマケン100%開発: cocos2d-x Androidプロジェクトの場合はStencilバッファを明示的に有効化しておかないとクリッピングできない事がある</a>

以上です。

■参考リンク  
Clipping  
<a href="http://qiita.com/senchan05/items/67e03d8b4dcb5eb30fdf" target="_blank" rel="noopener">スプライトのマスク処理（Cocos Code IDE, Lua言語）<br /> </a><a href="http://joujaku845.blog.fc2.com/blog-category-32.html" target="_blank" rel="noopener">chroot &#8211; cocos2dx3.0 スポットライトみたいなのを作る<br /> </a><a href="http://befool.co.jp/blog/chainzhang/cocos2dx-clipping-mask/" target="_blank" rel="noopener">Cocos2dx 3.0でクリッピングマスクの実現</a>

RenderTexture  
<a href="http://ladywendy.com/lab/cocos2d-x/76.html" target="_blank" rel="noopener">ladywendy &#8211; Cocos2d-x：動的にテクスチャを作成<br /> </a><a href="http://qiita.com/U-TAS/items/39f21a78e99c1fcc7545" target="_blank" rel="noopener">【Cocos2d-x 3.x】複数の画像を1枚のSpriteに合成する</a>