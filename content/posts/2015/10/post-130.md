---
title: '[cocos2dx]タッチイベント処理を削除する'
author: しゃまとん
date: 2015-10-04T04:36:57+00:00
url: /posts/130
featured_image: /images/posts/2015/08/Icon-cocos.png
categories:
  - cocos2dx
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

レイヤーを複数に分けていて、あるレイヤーのタッチ処理を切り分けたかったのですが、
検索してもあまり出てこなかったので、実装できた際のメモです。

cocos2d-xのver3.7での実装例です。

レイヤーにてタッチ処理を登録して、処理の優先度をあげておくと、仮にレイヤーをsetVisible(false)にしても、
優先度をあげた方が呼ばれてしまいうまくいかなかったため、登録を外すようにしてレイヤーを非表示にするようにしました。

下記は関数化されていますが、trueで呼ぶとタッチ処理をレイヤーに登録し、falseで呼ぶと切り離すようになっています。

_touchListenerとか_eventDispatcherとか存在してたんですね。知らなかった。。

```cpp
void HelloWorld::setTouchListener(bool enabled)
{
    if (enabled) {
        _touchListener = EventListenerTouchAllAtOnce::create();
        _touchListener->retain();
        _touchListener->onTouchesBegan = CC_CALLBACK_2(HelloWorld::onTouchesBegan, this);
        _eventDispatcher->addEventListenerWithSceneGraphPriority(_touchListener, this);
    }
    else {
        _eventDispatcher->removeEventListener(_touchListener);
        _touchListener->release();
        _touchListener = nullptr;
    }
}
```

以上です。