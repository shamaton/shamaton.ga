---
title: '[cocos2dx]内部でテクスチャを作成して、クリッピングする'
author: しゃまとん
date: 2015-03-02T09:46:29+00:00
url: /posts/73
featured_image: /images/posts/2015/08/Icon-cocos.png
categories:
  - cocos2dx
  - プログラミング関連

---
お世話になっております。

しゃまとんです。

cocos2dxをいじっていて、画面の一定領域を流れるメッセージを作りたいなぁと思い、
クリッピング処理はできないだろうかと模索していたときのメモです。  
この時は横に流れるメッセージを作ろうとしていました。

cocosのバージョンは3.2です。

<!--more-->

クリッピングのやり方に関するwebページはいろいろと出てくるのですが、
マスク用の画像を用意して、クリッピング処理するようなやり方だったのですが、
これのためだけに画像用意するのもなぁ・・・と思い調べていたら、プログラムでスプライトを合成して 、
一枚のスプライトを生成できるみたいなので、利用してみました。

RenderTextureを利用して、テクスチャを生成し、スプライトを作成します。  
流れるメッセージはそのままLabelを利用し、クリッピング処理をして、addChildします。

下記サンプルコードです。作成しているSceneから呼び出せば、表示される(はず)と思います。

```cpp
// ヘッダへの定義も忘れずに
void HelloWorld::_clippingTest() {

    auto layer_size = this->getContentSize();
    auto pos_y = layer_size.height - 50.0f;

    auto font_size = 30;

    auto label = Label::createWithSystemFont("abcdefghijklmnopqrstuvwxvz", "Arial", font_size);
    auto init_pos = Vec2(layer_size.width * 3/4, pos_y);
    label->setPosition(init_pos);
    label->setAnchorPoint(Vec2(0, 0.5));

    // 説明文を右から左に流す
    auto move = MoveTo::create(12.0f, Vec2(layer_size.width/4 - label->getContentSize().width, pos_y));
    auto res  = MoveTo::create(0.0f, init_pos);
    auto seq  = RepeatForever::create(Sequence::create(move, res, nullptr));
    label->runAction(seq);

    // クリッピングマスク用のテクスチャ作成
    // NOTE:addChildしないものはretainする
    auto render_tex = RenderTexture::create(layer_size.width, layer_size.height);
    render_tex->retain();
    {
        render_tex->begin();

        // 表示しない領域(透明画像)
        auto invisible = Sprite::create();
        invisible->setTextureRect(Rect(0.0f ,0.0f ,layer_size.width, font_size));
        invisible->setColor(Color3B::GREEN);
        invisible->setOpacity(0);
        invisible->setPosition(layer_size.width/2, layer_size.height/2);
        invisible->retain();

        // 表示する領域(黒)
        auto visible = Sprite::create();
        visible->setTextureRect(Rect(0.0f ,0.0f ,layer_size.width/4, font_size));
        visible->setColor(Color3B::BLACK);
        visible->setPosition(layer_size.width/2, layer_size.height/2);
        visible->retain();

        // render_texに焼き付ける
        invisible->visit();
        visible->visit();

        render_tex->end();
    }
    // 生成したテクスチャでスプライト作成
    auto stencil = Sprite::createWithTexture(render_tex->getSprite()->getTexture());
    stencil->setPosition(layer_size.width/2, pos_y);

    // クリッピング処理
    auto clipping = ClippingNode::create();
    clipping->setAnchorPoint(Vec2(0, 0));
    clipping->setPosition(0, 0);

    // くり抜く設定
    clipping->setStencil(stencil);
    clipping->setInverted(false); // 変数visibleの部分を可視化する
    clipping->setAlphaThreshold(0.01f);

    // クリッピングして表示したいもの
    clipping->addChild(label);
    this->addChild(clipping, 100);
}
```

こちらのようにクリッピングされるかと思います。

{{< figure src="/images/posts/2015/03/cliping_test.png" >}}

ちなみにAndroidではクリッピング処理がうまくいかない場合があるようですので、
下記のサイトを参考にさせていただきました。

{{< blogcard url="http://anz-note.tumblr.com/post/90781838271/cocos2dx-clippingnode-for-android" >}}
{{< blogcard url="http://hamken100.blogspot.jp/2014/08/cocos2d-x-androidstencil.html" >}}

以上です。

■参考リンク

Clipping  
{{< blogcard url="http://qiita.com/senchan05/items/67e03d8b4dcb5eb30fdf" >}}
{{< blogcard url="http://joujaku845.blog.fc2.com/blog-category-32.html" >}}
{{< blogcard url="http://befool.co.jp/blog/chainzhang/cocos2dx-clipping-mask/" >}}

RenderTexture  
{{< blogcard url="http://qiita.com/U-TAS/items/39f21a78e99c1fcc7545" >}}