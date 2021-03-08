---
title: '[cocos2dx]TMXTiledMapの各タイルからスプライトを複製する'
author: しゃまとん
date: 2015-09-01T15:15:37+00:00
url: /posts/123
featured_image: /images/posts/2015/08/tile_photo.jpg
categories:
  - cocos2dx
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

ファミコンのドラクエやFFで使われていた、マップチップを利用したRPGを作成しようかと考えているのですが、
cocos2d-xではTiledというツールで作成したデータをそのまま利用することができるTMXTiledMapというクラスが用意されており、
簡単にマップ処理を実装することができます。

<!--more-->

{{< blogcard url="http://www.mapeditor.org/" >}}

基本的には、TMXTiledMap::createで生成したインスタンスをaddChildすれば、表示されるのですが、

 * マップが大きい時の負荷が気になる（世界マップような時）
 * マップの回り込み表示をさせたい（右端に来た時に左端を表示する）

というような理由から表示される領域のみ画像を生成する処理のメモです。  
こちらはドラクエのように、キャラが常に中心にいてマップが動くような事例に有効かと思います。
よって実現したい内容によっては、非効率かもしれません。

処理はgetTileAtを参考にしました。  
cocos2d-xのバージョンは3.7でした。

```cpp
auto tile_map = TMXTiledMap::create("test.tmx");
auto layer = tile_map->getLayer("レイヤー名");

auto gid      = layer->getTileGIDAt(Vec2(1, 2)); // x, y
auto tile_set = layer->getTileSet();

Rect rect = tile_set->getRectForGID(gid);
rect = CC_RECT_PIXELS_TO_POINTS(rect);

auto tile = Sprite::createWithTexture(layer->getTexture(), rect);
tile->setBatchNode(layer);

this->addChild(tile);
```

ほぼgetTileAtと同じ処理です。こちらで1タイル分のスプライトを作ることができます。  
気になる方はCCTMXLayer.cppを読んでみるといいかと思います。


{{< blogcard url="https://github.com/MarkusPfundstein/Cocos2DX-Extensions/blob/master/CCTMXLayer/CCTMXLayer.cpp" >}}

以上です。

参考
{{< blogcard url="http://gpu-returns.tumblr.com/post/101328220534/%E3%83%9E%E3%83%83%E3%83%97%E3%82%A8%E3%83%87%E3%82%A3%E3%82%BFtiled%E3%81%A7%E4%BD%9C%E3%81%A3%E3%81%9F%E3%83%9E%E3%83%83%E3%83%97%E3%82%92cocos2d-x%E3%81%A7%E8%AA%AD%E3%81%BF%E8%BE%BC%E3%82%80" >}}
{{< blogcard url="http://www.slideshare.net/tonosamart/e-semi-3620140304" >}}