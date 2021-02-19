---
title: '[Unity] 常に使えるグローバルなコルーチンを用意する'
author: しゃまとん
type: post
date: 2018-06-11T22:27:15+00:00
url: /archives/448
featured_image: /wp-content/uploads/2017/12/test.gif
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
ただSpriteを差し替える対象のImageのゲームオブジェクトはfalseになっていてStartCoroutineできない。。。といった状況になるかもしれません。

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2017/12/a.png" alt="" width="495" height="29" class="aligncenter size-full wp-image-484" />][1]

基本的には自分の管理下でコルーチン制御するほうがいいと思うのですが、こういう場合にグローバルなコルーチンを作っておくことでそちらに処理を移譲することができます。

コードはこんな感じです。  
ただ実行したいIEnumratorをもらって実行するだけです。

<pre class="lang:c# decode:true">using System.Collections;
using System.Collections.Generic;
using UnityEngine;

/////////////////////////////////////////////////////////////////////////////////////////////////
/// &lt;summary&gt;
/// Coroutine for inactive or static class.
/// &lt;/summary&gt;
/////////////////////////////////////////////////////////////////////////////////////////////////
public class GlobalCoroutine : MonoBehaviour {

  // singleton
  private static GlobalCoroutine instance;

  /////////////////////////////////////////////////////////////////////////////////////////////////
  /// &lt;summary&gt;
  /// Run the specified routine.
  /// &lt;/summary&gt;
  /// &lt;param name="routine"&gt;Routine.&lt;/param&gt;
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
  /// &lt;summary&gt;
  /// execute routine
  /// &lt;/summary&gt;
  /// &lt;param name="src"&gt;Source.&lt;/param&gt;
  /////////////////////////////////////////////////////////////////////////////////////////////////
  private IEnumerator routine(IEnumerator src) {
    yield return StartCoroutine(src);
  }
}</pre>

このクラスはシングルトンで一度生成したら以降はずっと使えるようにしました。なるべくnewしたくないので。  
呼び出し側の例はこういう感じで。

<pre class="lang:c# decode:true">private void sample() {
    GlobalCoroutine.Run(routine());
  }

  private IEnumerator routine() {
    Debug.Log("1");
    yield return null;
    Debug.Log("2");
    yield return null;
    Debug.Log("3");
    yield return null;
  }</pre>

今回はサンプルで汎用的なダイアログを作って、シングルトンで使いまわすの想定して実装してみました。（ここでは処理は抜粋しています）  
staticメソッド内でプレハブからインスタンス生成するのですが、StartCoroutineは使えないため処理を移譲しています。

<pre class="lang:c# decode:true">/////////////////////////////////////////////////////////////////////////////////////////////////
  /// &lt;summary&gt;
  /// Create this instance.
  /// &lt;/summary&gt;
  /////////////////////////////////////////////////////////////////////////////////////////////////
  public static Coroutine Create() {
    if (instance != null) {
      return null;
    }
    return GlobalCoroutine.Run(create());
  }

  /////////////////////////////////////////////////////////////////////////////////////////////////
  /// &lt;summary&gt;
  /// Create this instance.
  /// &lt;/summary&gt;
  /////////////////////////////////////////////////////////////////////////////////////////////////
  private static IEnumerator create() {

    var req = Resources.LoadAsync(AssetName);
    while (!req.isDone) {
      yield return null;
    }

    GameObject prefab = req.asset as GameObject;
    Instantiate(prefab);
    yield return null;
  }</pre>

今回のサンプルは下記URLに置いておきました。  
<a href="https://github.com/shamaton/GlobalCoroutine" target="_blank" rel="noopener">https://github.com/shamaton/GlobalCoroutine</a>  
確認動作させたものはこちらです。

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2017/12/test.gif" alt="" width="512" height="297" class="aligncenter size-full wp-image-483" />][2]

どうにもならんときに使えるかもです。  
以上です。

■ 参考  
<a href="https://qiita.com/naoK/items/55fb18bd348cfaa92708" target="_blank" rel="noopener">コルーチンを呼び出し元とは別のオブジェクトで動作させる方法</a>

 [1]: https://shamaton.orz.hm/blog/wp-content/uploads/2017/12/a.png
 [2]: https://shamaton.orz.hm/blog/wp-content/uploads/2017/12/test.gif