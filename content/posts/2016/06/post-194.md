---
title: '[Unity] DoTweenで擬似的に追従させてみる'
author: しゃまとん
date: 2016-06-09T13:43:54+00:00
url: /posts/194
featured_image: /images/posts/2016/06/dotween_chase.gif
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

以前にDoTweenで跳ねるような演出を記事にしたのですが、
はねた後に続けて何かを追従する（追いかける）ような仕組みを実装してみました。

前回の記事

{{< blogcard url="/posts/176" >}}

追従なので、対象のtransform.positionにDoMoveすればよさ気なのですが、
それだと対象物が動き回っている場合、設定した場所までしか移動せず中途半端な感じになってしまいます。

そこでDoTweenのサンプルにもあったのですが、
ChangeEndValueという機能を使えば呼び出し時に設定したpositionを更新してTweenさせることができるようです。

値を更新するにはDoMove実行時に返り値を取得しておき、それに対してChangeEndValueを呼び出す形になります。

サンプルはこんな感じ。  
はねた後から、対象との距離を確認して一定値に達していない場合は更新するようにしています。

```csharp
using UnityEngine;
using System.Collections;

using DG.Tweening;

public class TextAction : MonoBehaviour {

  private Transform myTrans;

  public Transform target;

  private Tweener tweener;

  // 起動時
  void Start() {
    myTrans = transform;

    StartCoroutine(start());
  }

  private IEnumerator start() {
    // 待つ
    yield return new WaitForSeconds(1);

    Sequence seq = DOTween.Sequence();
    // バウンド
    seq.Append(myTrans.DOLocalMoveY(-0.1f, 1f).SetEase(Ease.OutBounce).OnComplete(callback).SetRelative());
    seq.Join(myTrans.DOLocalMoveX(0.3f, 1f).SetRelative());
  }

  // 更新
  void Update () {
    // 標的を適当に移動
    Vector3 pos = target.position;
    pos.x -= 0.005f;
    target.position = pos;

    // バウンド終了後から監視する
    if (tweener != null) {
      // ある程度近づいたら消す
      if (Vector3.Distance(myTrans.position, target.position) < 0.05f) {
        // 消す
        gameObject.SetActive(false);
      }
      // 終点を更新
      tweener.ChangeEndValue(target.position, 0.2f, true);
    }
  }

  // バウンド終了後に呼ばれる
  private void callback() {
    tweener = myTrans.DOMove(target.position, 0.5f);
  }
}
```

実行するとこんな感じです。  
赤が対象（target）オブジェクト、白がTextActionを追加したオブジェクトです。

{{< figure src="/images/posts/2016/06/dotween_chase.gif" class="center" >}}

これですが、あくまで擬似的なものなので障害物等は考慮されません。色々考慮したい場合はNavMeshを使うのがいいみたいです（まだ使ったこと無い）  
演出としてある程度使えるかもーと思います。

以上です。