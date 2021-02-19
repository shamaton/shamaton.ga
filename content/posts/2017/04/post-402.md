---
title: '[Unity] Zenjectを触り始めてみた'
author: しゃまとん
type: post
date: 2017-04-24T16:15:40+00:00
url: /archives/402
featured_image: /wp-content/uploads/2017/04/zenject.png
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

Unity界隈で話題になっている[Zenject][1]についてご存知でしょうか。  
かく言う私もつい最近しったのですが、使えるととても便利そうなので覚えたほうがいいなぁというやつみたいです。

どうやらこのアセットはポケモンGoで使われているそうで、依存性を注入してくれるフレームワークだそうです。Dependency Injection（DI）などと言うそうですが、ん？知らんぞということになり、一旦さわってみることに。

<p style="text-align: center;">
</p>

とりあえず初手としては、簡単に注入するぞ！な感じで始めてみました。

まずは空のシーンを用意しておきます。Zenjectを使うためには、SceneContextとInstallerが必要なようです。SceneContextはGameobjectを追加する要領で追加（右クリックメニュー等）できます。

[<img src="http://shamaton.orz.hm/blog/wp-content/uploads/2017/04/zen_1.gif" alt="" width="484" height="325" class="aligncenter size-full wp-image-408" />][2]

InstallerはZenjectで用意されているMonoInstallerクラスを継承して定義するようです。今回は一旦下記のようにしました。（他にも色々あるが）

<pre class="lang:c# decode:true ">using System;  
using Zenject;  

public class InstallerSample : MonoInstaller&lt;InstallerSample&gt;  
{  
  public override void InstallBindings()  
  {  
  }  
}</pre>

Installerは空のGameobjectを作成してコンポーネントとして追加しておきます。そしてSceneContextに登録しておきます。

[<img src="http://shamaton.orz.hm/blog/wp-content/uploads/2017/04/zen_1.png" alt="" width="269" height="275" class="aligncenter size-full wp-image-404" />][3]

次に挙動を確認するためのクラスを用意します。今回はFugaクラスからHogeクラスを参照してみることにしました。各クラスは下記のようにしておきます。

<pre class="lang:default decode:true " title="Hoge.cs">using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Hoge : MonoBehaviour {
  public int A = 1;
  public int B = 2;
  public int C = 3;
}</pre>

<pre class="lang:default decode:true" title="Fuga.cs">using System.Collections;
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
}</pre>

HogeクラスをつけるオブジェクトにはZenject Bindingもつけておきます。このコンポーネントをつけておくとBindしてくれるようです。

[<img src="http://shamaton.orz.hm/blog/wp-content/uploads/2017/04/zen_4.png" alt="" width="272" height="390" class="aligncenter size-full wp-image-407" />][4]

Fugaクラスも適当なオブジェクトにつけておきます。最終的にヒエラルキーはこんな感じになります。Fuga内には[Inject]としてHogeが定義されています。これがポイントで、参照等を設定しなくても、注入してくれるようになります。

[<img src="http://shamaton.orz.hm/blog/wp-content/uploads/2017/04/zen_2.png" alt="" width="160" height="106" class="aligncenter size-full wp-image-405" />][5]

実行すると・・・

[<img src="http://shamaton.orz.hm/blog/wp-content/uploads/2017/04/zen_3.png" alt="" width="419" height="111" class="aligncenter size-full wp-image-406" />][6]

こんな感じで参照が設定され、値を読むことができました。  
本当に初手な感じなので、引き続き覚えていきたいと思います！

以上です。

■参考  
[【Unity】Zenjectで禅の心を手に入れる][7]

 [1]: http://u3d.as/7ER
 [2]: http://shamaton.orz.hm/blog/wp-content/uploads/2017/04/zen_1.gif
 [3]: http://shamaton.orz.hm/blog/wp-content/uploads/2017/04/zen_1.png
 [4]: http://shamaton.orz.hm/blog/wp-content/uploads/2017/04/zen_4.png
 [5]: http://shamaton.orz.hm/blog/wp-content/uploads/2017/04/zen_2.png
 [6]: http://shamaton.orz.hm/blog/wp-content/uploads/2017/04/zen_3.png
 [7]: http://yutakaseda3216.hatenablog.com/entry/2017/04/17/124612