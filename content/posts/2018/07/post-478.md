---
title: '[Unity] 申請用のスクリーンショットをエディタで作る'
author: しゃまとん
date: 2018-07-14T02:52:06+00:00
url: /posts/478
featured_image: /images/posts/2017/12/ss2.png
categories:
  - unity
  - メモ

---
お世話になっております。  
しゃまとんです。

iOS/Androidのアプリ申請時に何かしらのスクリーンショットが必要なわけで、
ことだまっぷのときにも色々な解像度の素材を用意しないといけないのか...と嫌な感じになっていました。

ところが、そんなことも無いようでiOSで5.5インチサイズと12.9インチのものがあれば良くなったようです。
（昔はもっと面倒だった気が）

詳しくはこの辺をみるとよさそうです。

<http://help.apple.com/itunes-connect/developer/#/dev4e413fcb8>  
<http://help.apple.com/itunes-connect/developer/#/devd274dd925>

Androidに関しては細かい指定がなかったので、iOSで作ったものをそのまま利用することにしました。

で、肝心のスクリーンショットはどうしたらいいのかですが、
端末でキャプチャして転送してみたいなのも面倒だなということでエディタ上でキャプチャしてしまうことにしました。
（広告とか、デバック表示とか消すもの楽ですし）

まず指定の解像度でキャプチャできるようにUnityエディタで設定をします。  
Gameビューの左上にあるAspectの部分を選択すると、一番下にプラスボタンがあるので押すと、
どのような条件にするのか聞かれるので解像度を指定します。

{{< figure src="/images/posts/2017/12/ss1.png" >}}
{{< figure src="/images/posts/2017/12/ss2.png" >}}

次にコレをキャプチャするためのコードを組みます。  
メニューから選択してキャプチャする形式にしました。

```csharp
using System.Collections;
using System.Collections.Generic;
using System.IO;
using UnityEditor;
using UnityEngine;

/////////////////////////////////////////////////////////////////////////////////////////////////
/// <summary>
/// スクリーンショット機能
/// <summary>
/////////////////////////////////////////////////////////////////////////////////////////////////
public class Screenshot {

  private static string outputDir = "ScreenShots/";
  private static string fileName = "capture.png";

  /////////////////////////////////////////////////////////////////////////////////////////////////
  /// <summary>
  /// キャプチャ
  /// <summary>
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
  /// <summary>
  /// パスを取得
  /// <summary>
  /////////////////////////////////////////////////////////////////////////////////////////////////
  static string getOutputPath() {
    // Assetsは文字列に含めてはいけない
    string projectDir = Application.dataPath + "/../";
    string path       = Path.GetFullPath(projectDir); 
    path += outputDir;
    return path;
  }
}
```

エディタ上ではScaleが1倍表示されないケースになると思いますが、
キャプチャを取得してみると指定の解像度でスクリーンショットが取得できています。

{{< figure src="/images/posts/2017/12/ss3.png" >}}

必要ならこれにキャプションとかいれて、申請に追加すればOKですね！！  
以上です。

■ 参考
{{< blogcard url="https://docs.unity3d.com/jp/540/ScriptReference/Application.CaptureScreenshot.html" >}}
{{< blogcard url="http://magnaga.com/unity/2016/05/03/screen_shot/" >}}
