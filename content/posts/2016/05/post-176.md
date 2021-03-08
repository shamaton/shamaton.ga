---
title: '[Unity] DoTweenでuGUIにバウンドさせてみる'
author: しゃまとん
date: 2016-05-01T11:57:45+00:00
url: /posts/176
featured_image: /images/posts/2016/04/bound_text.gif
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

unityで何かしらのオブジェクトに対して、バウンドアクション（跳ねるような動作）をさせたいなーと思いまして、その際のメモです。

やはりtween系のアセットを利用してやるのが楽そうだなということで、今回もiTweenを使うかな−と思っていたのですが、
こんな情報を見まして。

{{< blogcard url="https://assetstore.unity.com/packages/tools/animation/itween-84?locale=ja-JP" >}}

> なるほど、iTweenに慣れてしまう前にDOTweenにするのが良いか RT <a href="https://twitter.com/takaoka_hide">@takaoka_hide</a>: DOTweenと他のTweenAssetのスピード比較表 <a href="https://t.co/24283nfP3v">pic.twitter.com/24283nfP3v</a>
> — ホウチ (@houchi_game) <a href="https://twitter.com/houchi_game/status/723354699450060803">2016年4月22日</a>


後々処理落ちが発生すると面倒そうだな&#8230;というのと、DoTweenが軽くてよさそうだなということで、
使い始めがてら利用して見ました。

{{< blogcard url="http://dotween.demigiant.com/index.php" >}}

DoTweenではtransformに対して、Tweenの関数が埋め込まれる形で利用できるように実装されていることで、
iTweenよりも簡潔に記載できるようです。callbackも簡単に設定できました。

とりあえず跳ねるっぽいコードです（callbackはログ出力のみです）。
利用する際はusingが必要です。テキスト等のUIに追加してください。

```csharp
using UnityEngine;

// これを追加する
using DG.Tweening;

public class BoundText : MonoBehaviour {
  // Use this for initialization
  void Start () {
  }

  // Update is called once per frame
  void Update () {

    // キー入力
    if (Input.GetKeyDown(KeyCode.H)) {
      // 跳ねるっぽく
      transform.DOLocalMoveY(-30f, 2f).SetEase(Ease.OutBounce);
      transform.DOLocalMoveX(30f, 2f).OnComplete(callback);
    }
  }

  // コールバック
  private void callback() {
    Debug.Log("do tween callback!!");
  }
}
```

動作はこんな感じになります。

{{< figure src="/images/posts/2016/04/bound_text.gif" >}}

iTweenよりも直感的だったので、今後はこちらを利用したいと思います。  
以上です。

■参考
{{< blogcard url="https://albatrus.com/entry/main/unity/7416" >}}
{{< blogcard url="http://tkhsken.hatenablog.com/entry/2016/04/06/220600" >}}
{{< blogcard url="http://esakun.hateblo.jp/entry/2015/08/26/090000" >}}