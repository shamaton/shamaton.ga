---
title: '[Unity] UniRxを使ったマルチシーンの利用方法を考えたけど微妙だった'
author: しゃまとん
date: 2017-05-28T09:27:10+00:00
url: /posts/387
featured_image: /images/posts/2017/05/unirx.png
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

今回は、UniRxを使ったマルチシーンの利用方法について考えていてやってみたけど微妙になってしまった話です。

最近はZenjectがいいという話からちょっと触ってみたりしているのですが、
やっぱマルチシーンの初期化時に引数的なものを渡して何かしたいよねと思っていたのが発端でググるとneueccさんの記事がヒットするので、それを参考にもう少し扱いやすくできないかなぁと実装してみました。

{{< blogcard url="http://neue.cc/2015/12/03_521.html" >}}

詳しい処理の流れは上記記事を参考にしていただくとして、変更した点は

* GameObject.FindObjectOfTypeはジェネリック変数Tで行う  
→ シーンを複数同時に読んだときに引数がアレしそう  
* Argをobject asしないようにした  
→ 毎回 as がめんどくさげ

2点で理由は書いたとおりです。

で... Argに何でもつっこめて勝手に動作して最高だぜ！と思って使っていたのですが、
とりあえず使ってみるとコードの読みづらさや、あれ&#8230;ここ同期まちさせたい...というような状況になり、
うーーーーん後から見返したときにやばそうとなった次第です。

簡単なサンプルを含めたコードはこんな感じです。

SceneBase.cs
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

using UniRx;

/////////////////////////////////////////////////////////////////////////////////////////////////
///<summary>
/// シーンローダーと連携するシーン管理基底クラス
///</summary>
/////////////////////////////////////////////////////////////////////////////////////////////////
public abstract class SceneBase<T> : PresenterBase {
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
  ///<summary>
  /// Initialize前にコールする
  ///</summary>
  ///<returns>The async.</returns>
  /////////////////////////////////////////////////////////////////////////////////////////////////
  public virtual IObservable<Unit> PrepareAsync() {
    return Observable.Return(Unit.Default);
  }

  /////////////////////////////////////////////////////////////////////////////////////////////////
  ///<summary>
  /// 生成時処理
  ///</summary>
  /////////////////////////////////////////////////////////////////////////////////////////////////
  protected override void OnAwake() {
    // 完了時にフラグを立てておく
    this.InitializeAsObservable().Subscribe(_ => IsLoaded = true);
  }

  /////////////////////////////////////////////////////////////////////////////////////////////////
  ///<summary>
  /// シーン名を設定
  ///</summary>
  /////////////////////////////////////////////////////////////////////////////////////////////////
  public void SetSceneName(string sceneName) {
    loadedSceneName = sceneName;
  }

  /////////////////////////////////////////////////////////////////////////////////////////////////
  ///<summary>
  /// 初期化値を設定
  ///</summary>
  ///<param name="arg">Argument.</param>
  /////////////////////////////////////////////////////////////////////////////////////////////////
  public void SetArg(T arg) {
    Arg = arg;
  }
}
```

SceneBaseLoader.cs
```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

using UniRx;

/////////////////////////////////////////////////////////////////////////////////////////////////
///<summary>
/// SceneBaseクラスを利用したシーンのロード管理
///</summary>
/////////////////////////////////////////////////////////////////////////////////////////////////
public static class SceneBaseLoader {

  /////////////////////////////////////////////////////////////////////////////////////////////////
  ///<summary>
  /// シーンを非同期にロード
  ///</summary>
  /////////////////////////////////////////////////////////////////////////////////////////////////
  public static IObservable<Unit> LoadAsync<T, T_Arg>(string sceneName, T_Arg argument, LoadSceneMode mode = LoadSceneMode.Single)
    where T : SceneBase<T_Arg>
  {
    return Observable.FromCoroutine<Unit>(observer => initRoutine(SceneManager.LoadSceneAsync(sceneName, mode), observer))
      .SelectMany(_ =>
        {
          // シーンを取得
          T[] scenes = GameObject.FindObjectsOfType<T>();

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
            .Do(__ => {
              // Activeにして動かしはじめる
              loadedScene.gameObject.SetActive(true);
            });
        });
  }

  /////////////////////////////////////////////////////////////////////////////////////////////////
  ///<summary>
  /// 初期化ルーチン
  ///</summary>
  /////////////////////////////////////////////////////////////////////////////////////////////////
  static IEnumerator initRoutine(AsyncOperation operation, IObserver<Unit> observer) {

    // ロード待機
    if (!operation.isDone) yield return operation;

    // 初期化処理する
    observer.OnNext(Unit.Default);

    // 完了
    observer.OnCompleted();
  }
}
```

Scene1.cs
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

using UniRx;

public class Scene1 : MonoBehaviour {
    
  void Update() {
    if (Input.GetKeyDown(KeyCode.A)) {
      Scene2Arg arg = new Scene2Arg();
      arg.mesasge = "Scene2をロードしました！";
      SceneBaseLoader.LoadAsync<Scene2, Scene2Arg>("Scene2", arg, LoadSceneMode.Additive).Subscribe();
    }
  }
}
```

Scene2.cs
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

using UniRx;

public class Scene2 : SceneBase<Scene2Arg> {

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
}
```

挙動例はこれ。

{{< figure src="/images/posts/2017/05/multi_test.gif" >}}

もっと理解していたらきれいに使えるかもなんですが、今はちょっとなーという感じになってしまいました。
一旦はシンプルな使い方にとどめておこうかと思います。

以上です。
