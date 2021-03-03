---
title: '[Unity] Zenjectを触り始めてみた'
author: しゃまとん
date: 2017-04-24T16:15:40+00:00
url: /posts/402
featured_image: /images/posts/2017/04/zenject.png
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

Unity界隈で話題になっているZenjectについてご存知でしょうか。  
かく言う私もつい最近しったのですが、使えるととても便利そうなので覚えたほうがいいなぁというやつみたいです。

{{< blogcard url="http://u3d.as/7ER" >}}

どうやらこのアセットはポケモンGoで使われているそうで、依存性を注入してくれるフレームワークだそうです。
Dependency Injection（DI）などと言うそうですが、ん？知らんぞということになり、一旦さわってみることに。

とりあえず初手としては、簡単に注入するぞ！な感じで始めてみました。

まずは空のシーンを用意しておきます。Zenjectを使うためには、SceneContextとInstallerが必要なようです。
SceneContextはGameobjectを追加する要領で追加（右クリックメニュー等）できます。

{{< figure src="/images/posts/2017/04/zen_1.gif" >}}

InstallerはZenjectで用意されているMonoInstallerクラスを継承して定義するようです。今回は一旦下記のようにしました。
（他にも色々あるが）

```csharp
using System;  
using Zenject;  

public class InstallerSample : MonoInstaller<InstallerSample>  
{  
  public override void InstallBindings()  
  {  
  }  
}
```

Installerは空のGameobjectを作成してコンポーネントとして追加しておきます。
そしてSceneContextに登録しておきます。

{{< figure src="/images/posts/2017/04/zen_1.png" >}}

次に挙動を確認するためのクラスを用意します。今回はFugaクラスからHogeクラスを参照してみることにしました。
各クラスは下記のようにしておきます。

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Hoge : MonoBehaviour {
  public int A = 1;
  public int B = 2;
  public int C = 3;
}
```

```csharp
using System.Collections;
using UnityEngine;
using Zenject;

public class Fuga : MonoBehaviour {

  [Inject] Hoge hoge;

  // Use this for initialization
  void Awake () {

    if (hoge == null) {
      Debug.Log("is null");
    }
    else {
      Debug.Log("not null");
      Debug.Log(hoge.A.ToString() + hoge.B.ToString() + hoge.C.ToString());
    }
  }

  // Update is called once per frame
  void Update () {
  }
}
```


HogeクラスをつけるオブジェクトにはZenject Bindingもつけておきます。
このコンポーネントをつけておくとBindしてくれるようです。

{{< figure src="/images/posts/2017/04/zen_4.png" >}}

Fugaクラスも適当なオブジェクトにつけておきます。最終的にヒエラルキーはこんな感じになります。Fuga内には[Inject]としてHogeが定義されています。これがポイントで、参照等を設定しなくても、注入してくれるようになります。

{{< figure src="/images/posts/2017/04/zen_2.png" >}}

実行すると・・・

{{< figure src="/images/posts/2017/04/zen_3.png" >}}
こんな感じで参照が設定され、値を読むことができました。  
本当に初手な感じなので、引き続き覚えていきたいと思います！

以上です。

■参考

{{< blogcard url="http://yutakaseda3216.hatenablog.com/entry/2017/04/17/124612" >}}
