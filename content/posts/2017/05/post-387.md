---
title: '[Unity] UniRxを使ったマルチシーンの利用方法を考えたけど微妙だった'
author: しゃまとん
type: post
date: 2017-05-28T09:27:10+00:00
url: /archives/387
featured_image: /wp-content/uploads/2017/05/unirx.png
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

今回は、UniRxを使ったマルチシーンの利用方法について考えていてやってみたけど微妙になってしまった話です。

最近はZenjectがいいという話からちょっと触ってみたりしているのですが、やっぱマルチシーンの初期化時に引数的なものを渡して何かしたいよねと思っていたのが発端でググるとneueccさんの記事がヒットするので、それを参考にもう少し扱いやすくできないかなぁと実装してみました。

<a href="http://neue.cc/2015/12/03_521.html" target="_blank" rel="noopener noreferrer">Unity 5.3のMulti Scene EditingをUniRxによるシーンナビゲーションで統合する</a>

詳しい処理の流れは上記記事を参考にしていただくとして、変更した点は

・GameObject.FindObjectOfTypeはジェネリック変数Tで行う  
→　シーンを複数同時に読んだときに引数がアレしそう  
・Argをobject asしないようにした  
→　毎回 as がめんどくさげ

2点で理由は書いたとおりです。

で&#8230;. Argに何でもつっこめて勝手に動作して最高だぜ！と思って使っていたのですが、とりあえず使ってみるとコードの読みづらさや、あれ&#8230;ここ同期まちさせたい&#8230;というような状況になり、うーーーーん後から見返したときにやばそうとなった次第です。

簡単なサンプルを含めたコードはこんな感じです。

<pre class="lang:c# decode:true" title="SceneBase.cs">using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

using UniRx;

/////////////////////////////////////////////////////////////////////////////////////////////////
/// &lt;summary&gt;
/// シーンローダーと連携するシーン管理基底クラス
/// &lt;/summary&gt;
/////////////////////////////////////////////////////////////////////////////////////////////////
public abstract class SceneBase&lt;T&gt; : PresenterBase {
  // 初期化引数
  public T Arg { get; private set; }

  // ロードしたシーン名
  private string loadedSceneName;
  public string LoadedSceneName {
    get {
      return loadedSceneName;
    }
  }

  // ロード済みフラグ
  public bool IsLoaded { get; private set; }

  /////////////////////////////////////////////////////////////////////////////////////////////////
  /// &lt;summary&gt;
  /// Initialize前にコールする
  /// &lt;/summary&gt;
  /// &lt;returns&gt;The async.&lt;/returns&gt;
  /////////////////////////////////////////////////////////////////////////////////////////////////
  public virtual IObservable&lt;Unit&gt; PrepareAsync() {
    return Observable.Return(Unit.Default);
  }

  /////////////////////////////////////////////////////////////////////////////////////////////////
  /// &lt;summary&gt;
  /// 生成時処理
  /// &lt;/summary&gt;
  /////////////////////////////////////////////////////////////////////////////////////////////////
  protected override void OnAwake() {
    // 完了時にフラグを立てておく
    this.InitializeAsObservable().Subscribe(_ =&gt; IsLoaded = true);
  }

  /////////////////////////////////////////////////////////////////////////////////////////////////
  /// &lt;summary&gt;
  /// シーン名を設定
  /// &lt;/summary&gt;
  /////////////////////////////////////////////////////////////////////////////////////////////////
  public void SetSceneName(string sceneName) {
    loadedSceneName = sceneName;
  }

  /////////////////////////////////////////////////////////////////////////////////////////////////
  /// &lt;summary&gt;
  /// 初期化値を設定
  /// &lt;/summary&gt;
  /// &lt;param name="arg"&gt;Argument.&lt;/param&gt;
  /////////////////////////////////////////////////////////////////////////////////////////////////
  public void SetArg(T arg) {
    Arg = arg;
  }
}
</pre>

<pre class="lang:c# decode:true" title="SceneBaseLoader.cs">using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

using UniRx;

/////////////////////////////////////////////////////////////////////////////////////////////////
/// &lt;summary&gt;
/// SceneBaseクラスを利用したシーンのロード管理
/// &lt;/summary&gt;
/////////////////////////////////////////////////////////////////////////////////////////////////
public static class SceneBaseLoader {

  /////////////////////////////////////////////////////////////////////////////////////////////////
  /// &lt;summary&gt;
  /// シーンを非同期にロード
  /// &lt;/summary&gt;
  /////////////////////////////////////////////////////////////////////////////////////////////////
  public static IObservable&lt;Unit&gt; LoadAsync&lt;T, T_Arg&gt;(string sceneName, T_Arg argument, LoadSceneMode mode = LoadSceneMode.Single)
    where T : SceneBase&lt;T_Arg&gt;
  {
    return Observable.FromCoroutine&lt;Unit&gt;(observer =&gt; initRoutine(SceneManager.LoadSceneAsync(sceneName, mode), observer))
      .SelectMany(_ =&gt;
        {
          // シーンを取得
          T[] scenes = GameObject.FindObjectsOfType&lt;T&gt;();

          // 同じシーンを重複して読まないようにする
          if (scenes.Length != 1) {
            Debug.LogErrorFormat(typeof(T).Name + " component must set only one in scene !!");
          }

          T loadedScene = scenes[0];

          // ロード済み
          if (loadedScene.IsLoaded) {
            Debug.LogErrorFormat(sceneName + " has alreay been loaded !!");
          }

          // ロード済みにして
          loadedScene.SetArg(argument);
          loadedScene.SetSceneName(sceneName);

          // 一旦非Activeにして止める
          loadedScene.gameObject.SetActive(false); 

          // PrepareAsyncが完了するまで待つ
          return loadedScene.PrepareAsync() 
            .Do(__ =&gt; {
              // Activeにして動かしはじめる
              loadedScene.gameObject.SetActive(true);
            });
        });
  }

  /////////////////////////////////////////////////////////////////////////////////////////////////
  /// &lt;summary&gt;
  /// 初期化ルーチン
  /// &lt;/summary&gt;
  /////////////////////////////////////////////////////////////////////////////////////////////////
  static IEnumerator initRoutine(AsyncOperation operation, IObserver&lt;Unit&gt; observer) {

    // ロード待機
    if (!operation.isDone) yield return operation;

    // 初期化処理する
    observer.OnNext(Unit.Default);

    // 完了
    observer.OnCompleted();
  }
}</pre>

<pre class="lang:c# decode:true " title="Scene1.cs">using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

using UniRx;

public class Scene1 : MonoBehaviour {
    
  void Update() {
    if (Input.GetKeyDown(KeyCode.A)) {
      Scene2Arg arg = new Scene2Arg();
      arg.mesasge = "Scene2をロードしました！";
      SceneBaseLoader.LoadAsync&lt;Scene2, Scene2Arg&gt;("Scene2", arg, LoadSceneMode.Additive).Subscribe();
    }
  }
}
</pre>

<pre class="lang:c# decode:true " title="Scene2.cs">using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

using UniRx;

public class Scene2 : SceneBase&lt;Scene2Arg&gt; {

  [SerializeField] private Text text;


  protected override IPresenter[] Children {
    get { return EmptyChildren; }
  }

  protected override void BeforeInitialize()
  {
    // 何もしない
  }

  protected override void Initialize()
  {
    if (Arg != null) {
      text.text = Arg.mesasge;
    }
  }
}

public class Scene2Arg {
  public string mesasge = "";
}</pre>

挙動例はこれ。

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2017/05/multi_test.gif" alt="" width="598" height="410" class="aligncenter size-full wp-image-411" />][1]

もっと理解していたらきれいに使えるかもなんですが、今はちょっとなーという感じになってしまいました。一旦はシンプルな使い方にとどめておこうかと思います。

以上です。

 [1]: https://shamaton.orz.hm/blog/wp-content/uploads/2017/05/multi_test.gif