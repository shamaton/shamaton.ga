---
title: '[gRPC] UnityとgolangでgRPCを使ってみる（androidまで）'
author: しゃまとん
type: post
date: 2018-08-09T13:57:14+00:00
url: /posts/553
featured_image: /images/posts/2018/08/grpc.svg
categories:
  - go
  - gRPC
  - unity

---
お世話になっております。  
しゃまとんです。

前の記事でまずはgolangでgRPCを使って動作させてみました。

{{< blogcard url="/posts/542" >}}

今回はclient側にUnityを使ってみたいと思います。  
実行環境はこんな感じでした。

Unity2018.1.1f1（OSはMacです）  
Go 1.10.3

まずUnityでgRPCを使うには.NETのバージョンを上げておく必要があります。  
Menu:Edit -> Project Settings -> Player -> Configuration として図のように設定しておきます。

{{< figure src="/images/posts/2018/08/grpc_config.png" >}}

次にgRPC関連の必要なパッケージを用意しましょう。  
本当に最近（2018/08時点）ですが、公式でビルド済みのライブラリを用意してくれるようなりました！

まだexperimentalなので、ご注意くださいね  
{{< blogcard url="https://github.com/grpc/grpc/issues/15013" >}}

ということでこちら（<https://packages.grpc.io/>）にアクセスし、
Daily Builds of master Branchの項目の最新のBuild IDのリンクをクリックします。

{{< figure src="/images/posts/2018/08/grpc_download.png" >}}

するとパッケージのリストがあるので、Grpc.Toolsとgrpc_unity_packageをダウンロードしましょう。

{{< figure src="/images/posts/2018/08/grpc_download2.png" >}}

ダウンロードしたら、Unityで新規プロジェクトを作成し、とりあえず移動させておきます。

Grpc.Toolsはnupkgになっていますが、拡張子をzipに変更することで簡単に解凍することができます。
ただGUI上で解凍すると、フォルダ構成がうまく復元できないことがあったので、コマンドで実行するほうがよいかもしれません。

```shell script
# できればフォルダを用意しておくとよい
$ mkdir Grpc.Tools
 
# zipに変更して解凍
$ mv Grpc.Tools.1.15.0-dev.nupkg Grpc.Tools.1.15.0-dev.zip
$ unzip Grpc.Tools.1.15.0-dev.zip -d Grpc.Tools
Archive:  Grpc.Tools.1.15.0-dev.zip
  inflating: Grpc.Tools/_rels/.rels
  inflating: Grpc.Tools/Grpc.Tools.nuspec
  inflating: Grpc.Tools/tools/windows_x86/protoc.exe
  inflating: Grpc.Tools/tools/windows_x86/grpc_csharp_plugin.exe
  ...
  inflating: Grpc.Tools/tools/macosx_x64/protoc
  inflating: Grpc.Tools/tools/macosx_x64/grpc_csharp_plugin
  inflating: Grpc.Tools/[Content_Types].xml
  inflating: Grpc.Tools/package/services/metadata/core-properties/8799bc730c0846ad904b28d32702ee35.psmdcp

# 対象OSのtoolに実行権限を付与
$ chmod +x Grpc.Tools/tools/macosx_x64/*
```

grpc_unity_packageはUnityで必要になるので、Assets配下にフォルダを作成して解凍します。
こちらはPluginsにすべて含まれた状態になるので、Assetsを対象にして解凍します。

```shell script
$ unzip grpc_unity_package.1.15.0-dev.zip -d Assets
Archive:  grpc_unity_package.1.15.0-dev.zip
warning:  grpc_unity_package.1.15.0-dev.zip appears to use backslashes as path separators
  inflating: Assets/Plugins/Google.Protobuf.meta
  inflating: Assets/Plugins/Grpc.Core.meta
  inflating: Assets/Plugins/System.Interactive.Async.meta
  ...
  inflating: Assets/Plugins/System.Interactive.Async/lib/net45/System.Interactive.Async.dll
  inflating: Assets/Plugins/System.Interactive.Async/lib/net45/System.Interactive.Async.dll.meta
  inflating: Assets/Plugins/System.Interactive.Async/lib/net45/System.Interactive.Async.xml.meta
```


これでgolangと同じ準備ができた（はず）ので、protoファイルを作成して実装を進めていきます。基本的に前回のものと一緒ですが、namespaceだけ分けておくため、追記して何処かに配置しておきましょう。（本当は共通のものを参照しているとよい）

今回はGrpcSample（プロジェクト名）/protosに配置しました。

```proto
syntax = "proto3";

# これを追記
option csharp_namespace = "Pj.Grpc.Sample";

package helloworld;
```

protoをC#に変換します。出力時に指定のフォルダがないとエラーになってしまうので、
事前に作成しておく必要があります。そして生成コマンドがgolangより長い。

```shell script
# 出力先を作成
$ mkdir -p Assets/Pj.Grpc

# コード生成
$ protoc -I protos --csharp_out Assets/Pj.Grpc \
  --grpc_out Assets/Pj.Grpc protos/*.proto \
  --plugin=protoc-gen-grpc=Grpc.Tools/tools/macosx_x64/grpc_csharp_plugin

# 確認
$ ls Assets/Pj.Grpc
Helloworld.cs     HelloworldGrpc.cs
```

それでは生成したコードを利用して、クライアントを書きます。
適当なシーンを作成しUI.Textを参照させておいてください。

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using Grpc.Core;
using Pj.Grpc.Sample;

public class Sample : MonoBehaviour {

  public string ip = "127.0.0.1";
  public string port = "9999";

  public Text text;

  private void Start() {
    text.text = "wait reply...";
    Say();
  }

  private void Say() {
    Channel channel = new Channel(ip + ":" + port, ChannelCredentials.Insecure);

    var client = new Greeter.GreeterClient(channel);
    string user = "editor";
#if UNITY_ANDROID && !UNITY_EDITOR
    user = "android";
#elif UNITY_IOS && !UNITY_EDITOR
    user = "ios";
#endif

    var reply = client.SayHello(new HelloRequest { Name = user });
    Debug.Log("reply: " + reply.Message);
    text.text = "reply: " + reply.Message;

    channel.ShutdownAsync().Wait();
  }
}
```

それではserver.goを実行した状態で、Unityの再生を開始すると&#8230;  
replyがきます！


{{< figure src="/images/posts/2018/08/grpc_unity_editor.gif" >}}

一応サーバーはこうなります。

```shell script
$ go run server.go
2018/08/09 22:35:34 call from editor
```

最後に端末で動作確認してみたいと思います。
Project SettingsのIdentificationを初期値から適当に使わなさそうな名前に変更しておきましょう。

{{< figure src="/images/posts/2018/08/grpc_id.png" >}}

あとはSwitch PlatformしてAndroid / iOSビルドするだけ！と思っていたのですが、iOSではビルドエラーになってしまうようでした。。。Androidでの実行結果だけ残しておきます。

{{< figure src="/images/posts/2018/08/grpc_android.png" >}}

文字が小さくてすいません。。  
以上です。

追記：iOSもできました！  

{{< blogcard url="/posts/563" >}}

■ 参考  
{{< blogcard url="https://qiita.com/jhorikawa_err/items/f75539ffe6cd7a360f65" >}}
