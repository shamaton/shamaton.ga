---
title: '[Unity] 申請用のスクリーンショットをエディタで作る'
author: しゃまとん
type: post
date: 2018-07-14T02:52:06+00:00
url: /archives/478
featured_image: /wp-content/uploads/2017/12/ss2.png
categories:
  - unity
  - メモ

---
お世話になっております。  
しゃまとんです。

iOS/Androidのアプリ申請時に何かしらのスクリーンショットが必要なわけで、ことだまっぷのときにも色々な解像度の素材を用意しないといけないのか&#8230;と嫌な感じになっていました。

ところが、そんなことも無いようでiOSで5.5インチサイズと12.9インチのものがあれば良くなったようです。（昔はもっと面倒だった気が）

詳しくはこの辺をみるとよさそうです。  
<http://help.apple.com/itunes-connect/developer/#/dev4e413fcb8>  
<http://help.apple.com/itunes-connect/developer/#/devd274dd925>

Androidに関しては細かい指定がなかったので、iOSで作ったものをそのまま利用することにしました。

で、肝心のスクリーンショットはどうしたらいいのかですが、端末でキャプチャして転送してみたいなのも面倒だなということでエディタ上でキャプチャしてしまうことにしました。（広告とか、デバック表示とか消すもの楽ですし）

まず指定の解像度でキャプチャできるようにUnityエディタで設定をします。  
Gameビューの左上にあるAspectの部分を選択すると、一番下にプラスボタンがあるので押すと、どのような条件にするのか聞かれるので解像度を指定します。

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2017/12/ss1.png" alt="" width="224" height="359" class="aligncenter size-full wp-image-487" />][1]

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2017/12/ss2.png" alt="" width="228" height="136" class="aligncenter size-full wp-image-488" />][2]

次にコレをキャプチャするためのコードを組みます。  
メニューから選択してキャプチャする形式にしました。

<pre class="lang:c# decode:true ">using System.Collections;
using System.Collections.Generic;
using System.IO;
using UnityEditor;
using UnityEngine;

/////////////////////////////////////////////////////////////////////////////////////////////////
/// &lt;summary&gt;
/// スクリーンショット機能
/// &lt;/summary&gt;
/////////////////////////////////////////////////////////////////////////////////////////////////
public class Screenshot {

  private static string outputDir = "ScreenShots/";
  private static string fileName = "capture.png";

  /////////////////////////////////////////////////////////////////////////////////////////////////
  /// &lt;summary&gt;
  /// キャプチャ
  /// &lt;/summary&gt;
  /////////////////////////////////////////////////////////////////////////////////////////////////
  [MenuItem("Tools/Screen Shot")]
  static void Capture() {
    string path = getOutputPath();
    if (!Directory.Exists(path)) {
      Directory.CreateDirectory(path);
    }

    string fn = path + fileName;
    Application.CaptureScreenshot(fn);

    Debug.Log("captured .. " + fn);
  }

  /////////////////////////////////////////////////////////////////////////////////////////////////
  /// &lt;summary&gt;
  /// パスを取得
  /// &lt;/summary&gt;
  /// &lt;returns&gt;The output path.&lt;/returns&gt;
  /////////////////////////////////////////////////////////////////////////////////////////////////
  static string getOutputPath() {
    // Assetsは文字列に含めてはいけない
    string projectDir = Application.dataPath + "/../";
    string path       = Path.GetFullPath(projectDir); 
    path += outputDir;
    return path;
  }
}</pre>

エディタ上ではScaleが1倍表示されないケースになると思いますが、キャプチャを取得してみると指定の解像度でスクリーンショットが取得できています。

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2017/12/ss3.png" alt="" width="259" height="66" class="aligncenter size-full wp-image-489" />][3]

必要ならこれにキャプションとかいれて、申請に追加すればOKですね！！  
以上です。

■ 参考  
<a href="https://docs.unity3d.com/jp/540/ScriptReference/Application.CaptureScreenshot.html" target="_blank" rel="noopener">[UNITY | DOCUMENTATION] Application.CaptureScreenshot</a>  
<a href="http://magnaga.com/unity/2016/05/03/screen_shot/" target="_blank" rel="noopener">iOS申請用のゲームスクリーンショットを撮る方法【Unity】【iOS申請】</a>

 [1]: https://shamaton.orz.hm/blog/wp-content/uploads/2017/12/ss1.png
 [2]: https://shamaton.orz.hm/blog/wp-content/uploads/2017/12/ss2.png
 [3]: https://shamaton.orz.hm/blog/wp-content/uploads/2017/12/ss3.png