---
title: '[Unity] transformはcacheした方が早いけど、gameObjectはどうなんだろう'
author: しゃまとん
date: 2016-05-29T07:42:26+00:00
url: /posts/187
featured_image: /images/posts/2016/03/unity-logo.png
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

Unityでパフォーマンスのよいコードを書く上でコンポーネントを取得してしまって扱うのは常套手段ですよね。  
transformはgameObject.transfromで呼び出すと実は毎回GetComponentしていて遅いということで自分も取得して使うようにしています。

{{< tweet 730933842895208449 >}}

そこで疑問があったのですが、gameObjectはどうなのかな〜もしかしてtransformみたいに遅かったりするのだろうか...と思い、
簡単ですが計測してみました。（他に誰か調べてたりしないかな...）

計測に利用したコードはこんな感じで、適当なゲームオブジェクトを作成してスクリプトをひっつけるだけです。

```csharp
using UnityEngine;
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
}
```

で計測してみたら、transformと似たような結果になりました。

{{< figure src="/images/posts/2016/05/time_check.png" >}}

gameObjectも場合によってはキャッシュしたほうがよいかもですね。  
以上です。

■参考  
{{< blogcard url="http://qiita.com/crow_ver6/items/3dc6ba29062d397bbf60" >}}