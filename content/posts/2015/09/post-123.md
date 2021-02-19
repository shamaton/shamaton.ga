---
title: '[cocos2dx]TMXTiledMapの各タイルからスプライトを複製する'
author: しゃまとん
type: post
date: 2015-09-01T15:15:37+00:00
url: /archives/123
featured_image: /wp-content/uploads/2015/08/tile_photo.jpg
categories:
  - cocos2dx
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

ファミコンのドラクエやFFで使われていた、マップチップを利用したRPGを作成しようかと考えているのですが、cocos2d-xではTiledというツールで作成したデータをそのまま利用することができるTMXTiledMapというクラスが用意されており、簡単にマップ処理を実装することができます。

<!--more-->

<a href="http://www.mapeditor.org/" target="_blank">Tiled</a>

基本的には、TMXTiledMap::createで生成したインスタンスをaddChildすれば、表示されるのですが、

  * マップが大きい時の負荷が気になる（世界マップような時）
  * マップの回り込み表示をさせたい（右端に来た時に左端を表示する）

というような理由から表示される領域のみ画像を生成する処理のメモです。  
こちらはドラクエのように、キャラが常に中心にいてマップが動くような事例に有効かと思います。よって実現したい内容によっては、非効率かもしれません。

処理はgetTileAtを参考にしました。  
cocos2d-xのバージョンは3.7でした。

<pre class="brush: cpp; gutter: true">auto tile_map = TMXTiledMap::create("test.tmx");
    auto layer = tile_map-&gt;getLayer("レイヤー名");

    auto gid      = layer-&gt;getTileGIDAt(Vec2(1, 2)); // x, y
    auto tile_set = layer-&gt;getTileSet();
    
    Rect rect = tile_set-&gt;getRectForGID(gid);
    rect = CC_RECT_PIXELS_TO_POINTS(rect);
    
    auto tile = Sprite::createWithTexture(layer-&gt;getTexture(), rect);
    tile-&gt;setBatchNode(layer);

    this-&gt;addChild(tile);</pre>

ほぼgetTileAtと同じ処理です。こちらで1タイル分のスプライトを作ることができます。  
気になる方は<a href="https://github.com/MarkusPfundstein/Cocos2DX-Extensions/blob/master/CCTMXLayer/CCTMXLayer.cpp" target="_blank">CCTMXLayer.cpp</a>を読んでみるといいかと思います。

以上です。

参考  
<a href="http://gpu-returns.tumblr.com/post/101328220534/%E3%83%9E%E3%83%83%E3%83%97%E3%82%A8%E3%83%87%E3%82%A3%E3%82%BFtiled%E3%81%A7%E4%BD%9C%E3%81%A3%E3%81%9F%E3%83%9E%E3%83%83%E3%83%97%E3%82%92cocos2d-x%E3%81%A7%E8%AA%AD%E3%81%BF%E8%BE%BC%E3%82%80" target="_blank">GPU_RETURNS:マップエディタtiledで作ったマップをcocos2d-xで読み込む</a>  
<a href="http://www.slideshare.net/tonosamart/e-semi-3620140304" target="_blank">SlideShare &#8211; タイルマップに挑戦</a>