---
title: '[Unity] transformはcacheした方が早いけど、gameObjectはどうなんだろう'
author: しゃまとん
type: post
date: 2016-05-29T07:42:26+00:00
url: /archives/187
featured_image: /wp-content/uploads/2016/03/unity-logo.png
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

Unityでパフォーマンスのよいコードを書く上でコンポーネントを取得してしまって扱うのは常套手段ですよね。  
transformはgameObject.transfromで呼び出すと実は毎回GetComponentしていて遅いということで自分も取得して使うようにしています。

<blockquote class="twitter-tweet" data-lang="ja">
  <p dir="ltr" lang="ja">
    GetComponentについて。<br /> 同一GameObjectなら：RequireComponentとして、Startで取得<br /> 違うGameObjectなら：FindWithTagとAssertのコンボで<a href="https://twitter.com/hashtag/UnityTips?src=hash">#UnityTips</a> <a href="https://t.co/8tAeYBxmWn">pic.twitter.com/8tAeYBxmWn</a>
  </p>
  
  <p>
    — 伊藤周 (@warapuri) <a href="https://twitter.com/warapuri/status/730933842895208449">2016年5月13日</a>
  </p>
</blockquote>



そこで疑問があったのですが、gameObjectはどうなのかな〜もしかしてtransformみたいに遅かったりするのだろうか&#8230;.と思い、簡単ですが計測してみました。（他に誰か調べてたりしないかな&#8230;）

計測に利用したコードはこんな感じで、適当なゲームオブジェクトを作成してスクリプトをひっつけるだけです。

<pre class="brush: csharp; gutter: true">using UnityEngine;
using System.Collections;
using System.Diagnostics; 

public class CacheTest : MonoBehaviour {

  // Use this for initialization
  void Start () {
    Transform cacheTrans = transform;

    var sw = new Stopwatch();
    sw.Start();
    for (int i = 0; i &lt; 1000000; i++) {
      Transform t = cacheTrans;
    }
    sw.Stop();
    // キャッシュしたtransfrom
    UnityEngine.Debug.Log("cacheTransform : " + sw.ElapsedMilliseconds);

    sw.Reset();
    sw.Start();
    for (int i = 0; i &lt; 1000000; i++) {
      Transform t = transform;
    }
    sw.Stop();
    // キャッシュしてないtransform
    UnityEngine.Debug.Log("getTransform : " + sw.ElapsedMilliseconds);

    GameObject cacheObj = gameObject;

    sw.Reset();
    sw.Start();
    for (int i = 0; i &lt; 1000000; i++) {
      GameObject g = cacheObj;
    }
    sw.Stop();
    // キャッシュしたgameObject
    UnityEngine.Debug.Log("cacheGameObject : " + sw.ElapsedMilliseconds);

    sw.Reset();
    sw.Start();
    for (int i = 0; i &lt; 1000000; i++) {
      GameObject g = gameObject;
    }
    sw.Stop();
    // キャッシュしてないgameObject
    UnityEngine.Debug.Log("get GameObject : " + sw.ElapsedMilliseconds);
  }
}</pre>

で計測してみたら、transformと似たような結果になりました。

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2016/05/time_check.png" alt="time_check" width="225" height="130" class="size-full wp-image-189 aligncenter" />][1]

gameObjectも場合によってはキャッシュしたほうがよいかもですね。  
以上です。

■参考  
<a href="http://qiita.com/crow_ver6/items/3dc6ba29062d397bbf60" target="_blank">this.transformを計測してみた</a>

 [1]: https://shamaton.orz.hm/blog/wp-content/uploads/2016/05/time_check.png