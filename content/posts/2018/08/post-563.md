---
title: '[gRPC] UnityとgolangでgRPCを使ってみる（iOSも）'
author: しゃまとん
date: 2018-08-09T23:36:14+00:00
url: /posts/563
featured_image: /images/posts/2018/08/grpc.svg
categories:
  - gRPC
  - unity

---
お世話になっております。  
しゃまとんです。

前回の記事でiOSで実行できなかったので、追加記事を作成しました。

{{< blogcard url="/posts/553" >}}

こちらで一応iOSでもビルドが通るようになりました。  
簡単ですが以下、手順です。

まずは、gRPCのライブラリ群（Plugins）をまるっと入れ替えしました。  
前回の記事にリンクした[issue][1]に書き込みをされているユーザーさんがUnity向けにビルドしたパッケージを置いてくれているのでそちらを使いました。

{{< blogcard url="https://github.com/jsmouret/grpc-unity-package/releases" >}}

ただ、こちらでもそのままXcodeに持っていても別の原因でビルドが通りません。libresolvというライブラリへの参照が必要なので、UnityのiOSビルド時に参照が追加されるようにしておきます。こちらのコードはEditorフォルダを作成した配置しておきましょう。

```csharp
using System.IO;
using UnityEngine;
using UnityEditor;
using UnityEditor.iOS.Xcode;
using UnityEditor.Callbacks;

public class XcodeSettings {

  [PostProcessBuildAttribute(0)]
  public static void OnPostprocessBuild(BuildTarget buildTarget, string pathToBuiltProject) {

    // Stop processing if target is NOT iOS
    if (buildTarget != BuildTarget.iOS)
      return;

    // Initialize PbxProject
    var projectPath = pathToBuiltProject + "/Unity-iPhone.xcodeproj/project.pbxproj";
    PBXProject proj = new PBXProject();
    proj.ReadFromFile(projectPath);
    string target = proj.TargetGuidByName("Unity-iPhone");

    // need libresolv for gRPC
    proj.AddFileToBuild(target, proj.AddFile("usr/lib/libresolv.9.dylib", "Frameworks/libresolv.9.dylib", PBXSourceTree.Sdk));

    // Apply settings
    File.WriteAllText(projectPath, proj.WriteToString());

  }
}
```

これでiOSビルドが通ります！  
これで実行してみた結果です。（しまった、こっちも文字が小さい。。）

{{< figure src="/images/posts/2018/08/grpc_ios.png.jpeg" class="center" width="200">}}

最初クライアントコードをそのままで確認していたのですが、なぜか返答が得られず困っていたので、
ボタンと追加してやってみたところ、返答がもらえました。  
最初のRPCではエラーっぽいんですけど、その後はうまくいくっていう。。。
ちょっと挙動がわからない部分があるので試してみる方は注意してやってみてください。

あと余談なんですが、アプリサイズが現状（2018/08時点）だとかなり膨らんでしまうっぽいです。
gRPCのライブラリサイズが大きいからだと思うのですが、この辺は今後に期待でしょうか。

以上です。

 [1]: https://github.com/grpc/grpc/issues/15013