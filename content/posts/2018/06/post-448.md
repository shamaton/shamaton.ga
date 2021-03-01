---
title: '[Unity] 常に使えるグローバルなコルーチンを用意する'
author: しゃまとん
date: 2018-06-24T22:27:15+00:00
url: /posts/448
featured_image: /images/posts/2017/12/test.gif
is_comment_form_freeze:
  - on
comment_form_freeze_message:
  - ' '
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

Unityにはコルーチンという機能がありますが、GameObjectがInActiveな場合使うことができません。  
自分自身がアクティブでも親がInActiveであれば、同様の状態になります。

例えば、ある画像を表示前にロードして切り替えておきたい&#8230;みたいなことがあるとします。  
ただSpriteを差し替える対象のImageのゲームオブジェクトはfalseになっていてStartCoroutineできない。。。
といった状況になるかもしれません。

{{< figure src="/images/posts/2017/12/a.png" >}}

基本的には自分の管理下でコルーチン制御するほうがいいと思うのですが、
こういう場合にグローバルなコルーチンを作っておくことでそちらに処理を移譲することができます。

コードはこんな感じです。  
ただ実行したい`IEnumrator`をもらって実行するだけです。

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

/////////////////////////////////////////////////////////////////////////////////////////////////
/// <summary>
/// Coroutine for inactive or static class.
/// <summary>
/////////////////////////////////////////////////////////////////////////////////////////////////
public class GlobalCoroutine : MonoBehaviour {

  // singleton
  private static GlobalCoroutine instance;

  /////////////////////////////////////////////////////////////////////////////////////////////////
  /// <summary>
  /// Run the specified routine.
  /// <summary>
  /////////////////////////////////////////////////////////////////////////////////////////////////
  public static Coroutine Run(IEnumerator routine) {
    // check and create GameObject.
    if (instance == null) {
      GameObject obj = new GameObject();
      obj.name = "GlobalCoroutine";
      instance = obj.AddComponent&lt;GlobalCoroutine&gt;();
      DontDestroyOnLoad(obj);
    }

    return instance.StartCoroutine(instance.routine(routine));
  }

  /////////////////////////////////////////////////////////////////////////////////////////////////
  /// <summary>
  /// execute routine
  /// <summary>
  /////////////////////////////////////////////////////////////////////////////////////////////////
  private IEnumerator routine(IEnumerator src) {
    yield return StartCoroutine(src);
  }
}
```

このクラスはシングルトンで一度生成したら以降はずっと使えるようにしました。なるべくnewしたくないので。  
呼び出し側の例はこういう感じで。

```csharp
private void sample() {
  GlobalCoroutine.Run(routine());
}

private IEnumerator routine() {
  Debug.Log("1");
  yield return null;
  Debug.Log("2");
  yield return null;
  Debug.Log("3");
  yield return null;
}
```

今回はサンプルで汎用的なダイアログを作って、シングルトンで使いまわすの想定して実装してみました。
（ここでは処理は抜粋しています）  
staticメソッド内でプレハブからインスタンス生成するのですが、StartCoroutineは使えないため処理を移譲しています。

```csharp
  /////////////////////////////////////////////////////////////////////////////////////////////////
  /// <summary>
  /// Create this instance.
  /// <summary>
  /////////////////////////////////////////////////////////////////////////////////////////////////
  public static Coroutine Create() {
    if (instance != null) {
      return null;
    }
    return GlobalCoroutine.Run(create());
  }

  /////////////////////////////////////////////////////////////////////////////////////////////////
  /// <summary>
  /// Create this instance.
  /// <summary>
  /////////////////////////////////////////////////////////////////////////////////////////////////
  private static IEnumerator create() {

    var req = Resources.LoadAsync(AssetName);
    while (!req.isDone) {
      yield return null;
    }

    GameObject prefab = req.asset as GameObject;
    Instantiate(prefab);
    yield return null;
  }
```

今回のサンプルは下記URLに置いておきました。  
<a href="https://github.com/shamaton/GlobalCoroutine" target="_blank" rel="noopener">https://github.com/shamaton/GlobalCoroutine</a>  
確認動作させたものはこちらです。

{{< figure src="/images/posts/2017/12/test.gif" >}}

どうにもならんときに使えるかもです。  
以上です。

■ 参考  
{{< blogcard url="https://qiita.com/naoK/items/55fb18bd348cfaa92708" >}}
