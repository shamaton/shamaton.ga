---
title: '[Unity]パッケージ重複でAndroidのBuildに失敗した'
author: しゃまとん
date: 2016-03-27T03:05:18+00:00
url: /posts/163
featured_image: /images/posts/2016/03/unity-logo.png
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

Unityのモバイル開発で外部のサービスを組み込む場合、iOSやAndroid用のパッケージを利用することになるのですが、
下記のようなエラーが出ることがあります。

```text
Error building Player: CommandInvokationFailure: Unable to convert classes into dex format. See the Console for details. 
/Library/Java/JavaVirtualMachines/jdk1.7.0_65.jdk/Contents/Home/bin/java -Xmx2048M -Dcom.android.sdkmanager.toolsdir="/Users/mukaidaichi/Developer/Android/sdk/tools" -Dfile.encoding=UTF8 -jar "/Applications/Unity/Unity.app/Contents/BuildTargetTools/AndroidPlayer/sdktools.jar"
```

このようなメッセージがでていて更に下記のようなメッセージが続く場合

```text
UNEXPECTED TOP-LEVEL EXCEPTION: 
java.lang.IllegalArgumentException: already added:
```

何かのパッケージが複数同梱されている状態になっています。  
already addedの後に続くメッセージにパッケージ名に似た文字列が表示されていると思います。

今回のメモでは、android-support-v4が外部のunitypackageを利用した際に別のディレクトリに複数配置されていました。

対処としては、どちらかを削除することで解決できました。  
以上です。