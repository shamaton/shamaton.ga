---
title: '[Unity] RaycastのlayerMaskの扱い方について'
author: しゃまとん
type: post
date: 2016-11-11T13:08:36+00:00
url: /archives/327
featured_image: /wp-content/uploads/2016/11/laycast_test.gif
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

UnityにはRaycastという機能があり、例えばあるオブジェクトの正面にrayを発射して  
別のオブジェクトが存在するかを取得することが出来ます。

開発でRaycastを使う必要があり、使い始めたのですが勘違いはハマりポイントらしき点があったので  
メモしておきたいと思います。

Raycastの使い方はこちらのサイトで分かりやすく説明されています。



おそらく基本的に使うのはPhysics.Raycastということで、使っていたのですが衝突が判定されなかった  
のですが、LayerMaskの使い方を勘違いしておりました。

<pre class="lang:c# decode:true" title="Physics.Raycast">public static bool Raycast(Vector3 origin, Vector3 direction, float maxDistance, int layerMask);</pre>

LayerMaskを使わなければ、Ignore RaycastにLayerを指定していないものをすべて検出されるのですが、  
決められたものを検出したい場合はLayerMaskを使うと思います。

LayerMaskには文字通りMaskらしきものを設定しないといけません。  
ということで、Layerの番号を取得して下記のようにしてやります。

<pre class="lang:c# decode:true" title="set mask">// レイヤーの管理番号を取得
int layerNo = LayerMask.NameToLayer("レイヤーの名前");
// マスクへの変換（ビットシフト）
int layerMask = 1 &lt;&lt; layerNo;</pre>

ちなみにLayerを見てみると31まであります。多分bit管理ですよね。

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2016/11/layer_setting.png" alt="layer_setting" width="165" height="361" class="aligncenter size-full wp-image-330" />][1]

簡単に確認してみました。cubeからsphereにrayを射出して当たったら色を変えます。



&nbsp;

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2016/11/laycast_test.gif" alt="laycast_test" width="344" height="204" class="aligncenter size-full wp-image-331" />][2]

最初、取得した番号をそのまま渡していたので検出がうまくいきませんでした。  
これで検出できるようになりました。よかったよかった。

サンプルにつかったのはこちらです。

[GitHub &#8211; shamaton/LaycastSample][3]

以上です。

 [1]: https://shamaton.orz.hm/blog/wp-content/uploads/2016/11/layer_setting.png
 [2]: https://shamaton.orz.hm/blog/wp-content/uploads/2016/11/laycast_test.gif
 [3]: https://github.com/shamaton/LaycastSample