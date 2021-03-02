---
title: '[cocos2dx]photonを使ってみた – その2'
author: しゃまとん
type: post
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

<pre class="brush: cpp; gutter: true">#include &lt;array&gt;
#include &lt;queue&gt;</pre>

publicの最下位に追加

<pre class="brush: cpp; gutter: true">// ルームが存在するか否かを返すメソッド
	bool isRoomExists(void);
	// イベントを送信するメソッド
	void sendEvent(nByte code, ExitGames::Common::Hashtable* eventContent);

	// 自分のプレイヤー番号
	int playerNr = 0;
	// イベントキュー
	std::queue&lt;std::array&lt;float, 3&gt;&gt; eventQueue;</pre>

・NetworkLogic.cpp  
ファイルの最下部にでも追加する

<pre class="brush: cpp; gutter: true">bool NetworkLogic::isRoomExists(void)
{
	if (mLoadBalancingClient.getRoomList().getIsEmpty()) {
		return false;
	}

	return true;
}

void NetworkLogic::sendEvent(nByte code, ExitGames::Common::Hashtable* eventContent)
{
	mLoadBalancingClient.opRaiseEvent(true, eventContent, 1, code);
}</pre>

customEventActionを下記のように上書きする

<pre class="brush: cpp; gutter: true">void NetworkLogic::customEventAction(int playerNr, nByte eventCode, const ExitGames::Common::Object& eventContent)
{
	ExitGames::Common::Hashtable* event;

	switch (eventCode) {
		case 1:
			event = ExitGames::Common::ValueObject&lt;ExitGames::Common::Hashtable*&gt;(eventContent).getDataCopy();
			float x = ExitGames::Common::ValueObject&lt;float&gt;(event-&gt;getValue(1)).getDataCopy();
			float y = ExitGames::Common::ValueObject&lt;float&gt;(event-&gt;getValue(2)).getDataCopy();
			eventQueue.push({static_cast&lt;float&gt;(playerNr), x, y});
			break;
	}
}</pre>

createRoomReturn(), joinRoomReturn(), joinRandomRoomReturn() にlocalPlayerNrを追加  
こちらは元記事のままでOKです。各fuctionの最下部に記入すると良いと思います。

・HelloWorldScene.h  
タップ処理が漏れているので、元記事の部分と合わせて追記します。  
public

<pre class="brush: actionscript3; gutter: true">virtual bool onTouchBegan(cocos2d::Touch *touch, cocos2d::Event *unused_event);
	virtual void onTouchMoved(cocos2d::Touch *touch, cocos2d::Event *unused_event);
	virtual void onTouchEnded(cocos2d::Touch *touch, cocos2d::Event *unused_event);
	virtual void onTouchCancelled(cocos2d::Touch *touch, cocos2d::Event *unused_event);</pre>

private

<pre class="brush: cpp; gutter: true">virtual void update(float delta);

	void addParticle(int playerNr, float x, float y);

	NetworkLogic* networkLogic;</pre>

・HelloWorldScene.cpp  
各タッチイベント処理を追加します。Beganには、イベント送信処理を記載します。

<pre class="brush: cpp; gutter: true">bool HelloWorld::onTouchBegan(cocos2d::Touch* touch, cocos2d::Event* event)
{

	if (networkLogic-&gt;playerNr) {
		this-&gt;addParticle(networkLogic-&gt;playerNr, touch-&gt;getLocation().x, touch-&gt;getLocation().y);

		// イベント（タッチ座標）を送信
		ExitGames::Common::Hashtable* eventContent = new ExitGames::Common::Hashtable();
		eventContent-&gt;put&lt;int, float&gt;(1, touch-&gt;getLocation().x);
		eventContent-&gt;put&lt;int, float&gt;(2, touch-&gt;getLocation().y);
		networkLogic-&gt;sendEvent(1, eventContent);
	}

	return true;
}

void HelloWorld::onTouchMoved(cocos2d::Touch *touch, cocos2d::Event *unused_event) {

}

void HelloWorld::onTouchEnded(cocos2d::Touch *touch, cocos2d::Event *unused_event) {

}

void HelloWorld::onTouchCancelled(cocos2d::Touch *touch, cocos2d::Event *unused_event){

}</pre>

描画更新処理(update)は下記のようにします。Inputのところが少し違います。

<pre class="brush: cpp; gutter: true">void HelloWorld::update(float delta)
{
	networkLogic-&gt;run();

	switch (networkLogic-&gt;getState()) {
		case STATE_CONNECTED:
		case STATE_LEFT:
			// ルームが存在すればジョイン、なければ作成する
			if (networkLogic-&gt;isRoomExists()) {
				networkLogic-&gt;setLastInput(INPUT_2);
			} else {
				networkLogic-&gt;setLastInput(INPUT_1);
			}
			break;
		case STATE_DISCONNECTED:
			// 接続が切れたら再度接続
			networkLogic-&gt;connect();
			break;
		case STATE_CONNECTING:
		case STATE_JOINING:
		case STATE_JOINED:
		case STATE_LEAVING:
		case STATE_DISCONNECTING:
		default:
			break;
	}

	while (!networkLogic-&gt;eventQueue.empty()) {
		std::array&lt;float, 3&gt; arr = networkLogic-&gt;eventQueue.front();
		networkLogic-&gt;eventQueue.pop();

		int playerNr = static_cast&lt;int&gt;(arr[0]);
		float x = arr[1];
		float y = arr[2];
		CCLOG("%d, %f, %f", playerNr, x, y);

		this-&gt;addParticle(playerNr, x, y);
	}
}</pre>

パーティクル処理(addParticle)はそのまま使えるので、コピーして使いましょう。

init()にタッチ処理の設定とupdate処理の設定を追加します。  
NetworkLogicの生成も少しだけ違っています。

<pre class="brush: cpp; gutter: true">// シングルタップリスナーを用意する
	auto listener = EventListenerTouchOneByOne::create();
	listener-&gt;setSwallowTouches(_swallowsTouches);

	// 各イベントの割り当て
	listener-&gt;onTouchBegan     = CC_CALLBACK_2(HelloWorld::onTouchBegan, this);
	listener-&gt;onTouchMoved     = CC_CALLBACK_2(HelloWorld::onTouchMoved, this);
	listener-&gt;onTouchEnded     = CC_CALLBACK_2(HelloWorld::onTouchEnded, this);
	listener-&gt;onTouchCancelled = CC_CALLBACK_2(HelloWorld::onTouchCancelled, this);

	// イベントディスパッチャにシングルタップ用リスナーを追加する
	_eventDispatcher-&gt;addEventListenerWithSceneGraphPriority(listener, this);

	// Photonネットワーククラスのインスタンスを作成
	networkLogic = new NetworkLogic();

	// 毎フレームでupdateを実行させる
	this-&gt;schedule(schedule_selector(HelloWorld::update));
	this-&gt;scheduleUpdate();</pre>

これでビルドできれば、photon realtimeを利用した、マルチプレイヤーのサンプルが動作すると思います。  
Androidでもビルドできると思いますので、やってみてください。

<p style="text-align: center;">
  <a href="http://shamaton.orz.hm/blog/images/posts/2015/03/2015_0329_photon_test_exec.png"><img src="http://shamaton.orz.hm/blog/images/posts/2015/03/2015_0329_photon_test_exec.png" alt="photon test exec" width="469" height="307" class="aligncenter  wp-image-87" /></a>
</p>

お疲れ様でした。

参考サイト：  
<a href="http://recruit.gmo.jp/engineer/jisedai/blog/cocos2d-x_photon/" target="_blank">cocos2d-xでPhotonを使ってみよう</a>