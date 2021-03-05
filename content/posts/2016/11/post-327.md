---
title: '[Unity] RaycastのlayerMaskの扱い方について'
author: しゃまとん
date: 2016-11-11T13:08:36+00:00
url: /posts/327
featured_image: /images/posts/2016/11/laycast_test.gif
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

{{< blogcard url="http://megumisoft.hatenablog.com/entry/2015/08/13/172136">}}

おそらく基本的に使うのはPhysics.Raycastということで、使っていたのですが衝突が判定されなかった  
のですが、LayerMaskの使い方を勘違いしておりました。

```csharp
public static bool Raycast(Vector3 origin, Vector3 direction, float maxDistance, int layerMask);
```

LayerMaskを使わなければ、Ignore RaycastにLayerを指定していないものをすべて検出されるのですが、  
決められたものを検出したい場合はLayerMaskを使うと思います。

LayerMaskには文字通りMaskらしきものを設定しないといけません。  
ということで、Layerの番号を取得して下記のようにしてやります。

```csharp
// レイヤーの管理番号を取得
int layerNo = LayerMask.NameToLayer("レイヤーの名前");
// マスクへの変換（ビットシフト）
int layerMask = 1 << layerNo;
```

ちなみにLayerを見てみると31まであります。多分bit管理ですよね。

{{< figure src="/images/posts/2016/11/layer_setting.png">}}

簡単に確認してみました。cubeからsphereにrayを射出して当たったら色を変えます。

{{< figure src="/images/posts/2016/11/laycast_test.gif">}}

最初、取得した番号をそのまま渡していたので検出がうまくいきませんでした。  
これで検出できるようになりました。よかったよかった。

サンプルにつかったのはこちらです。

{{< blogcard url="https://github.com/shamaton/LaycastSample" >}}

以上です。