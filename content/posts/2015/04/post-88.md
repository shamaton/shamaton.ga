---
title: '[cocos2dx]photonを使ってみた – その2'
author: しゃまとん
date: 2015-04-09T13:22:43+00:00
url: /posts/88
featured_image: /images/posts/2015/08/photon_realtime_turquoise.png
categories:
  - メモ

---
お世話になります。

しゃまとんです。

photon使ってみたの続きです。  
こちらではcocos2d-xでのプログラミングを行っていきます。  
前回通り記事の手順にそって変更点を記載していきます。

その前までの設定はこちら

{{< blogcard url="/posts/84" >}}

<!--more-->

■Photonクライアントの作成

手順は全部で7つ記載されています。2,4,5が少し違います。

  1. 説明の通り、NetworkLogic.cpp/.hをコピーします
  2. OutputListenerを削除、としていますが、メッセージを削除してしまうと、実行がうまくいかない場合にわからなくなるので、面倒ですが、CCLOGに置き換えをオススメします。
  3. 説明の通り、インクルード
  4. NetworkLogic.cppのL4にあるappIdを自分のIDにします <pre class="brush: cpp; gutter: true">static const ExitGames::Common::JString appId = L"ここをIDにする";</pre>

  5. FQDNの設定は不要です
  6. 説明の通り、インクルードします
  7. 説明の通り、名前空間を指定します。

これで、ビルドが通るようになります。

■アプリケーションへの組み込み

処理の概要については、変更はありません。

ソースの変更箇所については、記載されているのコード内の特殊文字がエスケープされてしまっているため、コピペしてもエラーになってしまいます。元サイトとコードが変わらない部分は、元サイトを参考にコピペしていただければ大丈夫です。

・NetworkLogic.h  
array,queueをインクルード

```cpp
#include <array>
#include <queue>
```

publicの最下位に追加

```cpp
// ルームが存在するか否かを返すメソッド
bool isRoomExists(void);
// イベントを送信するメソッド
void sendEvent(nByte code, ExitGames::Common::Hashtable* eventContent);

// 自分のプレイヤー番号
int playerNr = 0;
// イベントキュー
std::queue<std::array<float, 3>> eventQueue;
```

・NetworkLogic.cpp  
ファイルの最下部にでも追加する

```cpp
bool NetworkLogic::isRoomExists(void)
{
	if (mLoadBalancingClient.getRoomList().getIsEmpty()) {
		return false;
	}

	return true;
}

void NetworkLogic::sendEvent(nByte code, ExitGames::Common::Hashtable* eventContent)
{
	mLoadBalancingClient.opRaiseEvent(true, eventContent, 1, code);
}
```

customEventActionを下記のように上書きする

```cpp
void NetworkLogic::customEventAction(int playerNr, nByte eventCode, const ExitGames::Common::Object& eventContent)
{
	ExitGames::Common::Hashtable* event;

	switch (eventCode) {
		case 1:
			event = ExitGames::Common::ValueObject<ExitGames::Common::Hashtable*>(eventContent).getDataCopy();
			float x = ExitGames::Common::ValueObject<float>(event->getValue(1)).getDataCopy();
			float y = ExitGames::Common::ValueObject<float>(event->getValue(2)).getDataCopy();
			eventQueue.push({static_cast<float>(playerNr), x, y});
			break;
	}
}
```

createRoomReturn(), joinRoomReturn(), joinRandomRoomReturn() にlocalPlayerNrを追加  
こちらは元記事のままでOKです。各functionの最下部に記入すると良いと思います。

・HelloWorldScene.h  
タップ処理が漏れているので、元記事の部分と合わせて追記します。  
public
```cpp
virtual bool onTouchBegan(cocos2d::Touch *touch, cocos2d::Event *unused_event);
virtual void onTouchMoved(cocos2d::Touch *touch, cocos2d::Event *unused_event);
virtual void onTouchEnded(cocos2d::Touch *touch, cocos2d::Event *unused_event);
virtual void onTouchCancelled(cocos2d::Touch *touch, cocos2d::Event *unused_event);
```

private

```cpp
virtual void update(float delta);

void addParticle(int playerNr, float x, float y);

NetworkLogic* networkLogic;
```

・HelloWorldScene.cpp  
各タッチイベント処理を追加します。Beganには、イベント送信処理を記載します。

```cpp
bool HelloWorld::onTouchBegan(cocos2d::Touch* touch, cocos2d::Event* event)
{

	if (networkLogic->playerNr) {
		this->addParticle(networkLogic->playerNr, touch->getLocation().x, touch->getLocation().y);

		// イベント（タッチ座標）を送信
		ExitGames::Common::Hashtable* eventContent = new ExitGames::Common::Hashtable();
		eventContent->put<int, float>(1, touch->getLocation().x);
		eventContent->put<int, float>(2, touch->getLocation().y);
		networkLogic->sendEvent(1, eventContent);
	}

	return true;
}

void HelloWorld::onTouchMoved(cocos2d::Touch *touch, cocos2d::Event *unused_event) {

}

void HelloWorld::onTouchEnded(cocos2d::Touch *touch, cocos2d::Event *unused_event) {

}

void HelloWorld::onTouchCancelled(cocos2d::Touch *touch, cocos2d::Event *unused_event){

}
```

描画更新処理(update)は下記のようにします。Inputのところが少し違います。

```cpp
void HelloWorld::update(float delta)
{
	networkLogic->run();

	switch (networkLogic->getState()) {
		case STATE_CONNECTED:
		case STATE_LEFT:
			// ルームが存在すればジョイン、なければ作成する
			if (networkLogic->isRoomExists()) {
				networkLogic->setLastInput(INPUT_2);
			} else {
				networkLogic->setLastInput(INPUT_1);
			}
			break;
		case STATE_DISCONNECTED:
			// 接続が切れたら再度接続
			networkLogic->connect();
			break;
		case STATE_CONNECTING:
		case STATE_JOINING:
		case STATE_JOINED:
		case STATE_LEAVING:
		case STATE_DISCONNECTING:
		default:
			break;
	}

	while (!networkLogic->eventQueue.empty()) {
		std::array<float, 3> arr = networkLogic->eventQueue.front();
		networkLogic->eventQueue.pop();

		int playerNr = static_cast<int>(arr[0]);
		float x = arr[1];
		float y = arr[2];
		CCLOG("%d, %f, %f", playerNr, x, y);

		this->addParticle(playerNr, x, y);
	}
}
```

パーティクル処理(addParticle)はそのまま使えるので、コピーして使いましょう。

init()にタッチ処理の設定とupdate処理の設定を追加します。  
NetworkLogicの生成も少しだけ違っています。

```cpp
// シングルタップリスナーを用意する
auto listener = EventListenerTouchOneByOne::create();
listener->setSwallowTouches(_swallowsTouches);

// 各イベントの割り当て
listener->onTouchBegan     = CC_CALLBACK_2(HelloWorld::onTouchBegan, this);
listener->onTouchMoved     = CC_CALLBACK_2(HelloWorld::onTouchMoved, this);
listener->onTouchEnded     = CC_CALLBACK_2(HelloWorld::onTouchEnded, this);
listener->onTouchCancelled = CC_CALLBACK_2(HelloWorld::onTouchCancelled, this);

// イベントディスパッチャにシングルタップ用リスナーを追加する
_eventDispatcher->addEventListenerWithSceneGraphPriority(listener, this);

// Photonネットワーククラスのインスタンスを作成
networkLogic = new NetworkLogic();

// 毎フレームでupdateを実行させる
this->schedule(schedule_selector(HelloWorld::update));
this->scheduleUpdate();
```

これでビルドできれば、photon realtimeを利用した、マルチプレイヤーのサンプルが動作すると思います。  
Androidでもビルドできると思いますので、やってみてください。

{{< figure src="/images/posts/2015/03/2015_0329_photon_test_exec.png" >}}

お疲れ様でした。

参考サイト：  
{{< blogcard url="http://recruit.gmo.jp/engineer/jisedai/blog/cocos2d-x_photon/" >}}