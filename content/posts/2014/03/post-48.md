---
title: cocos2dxのプロジェクトを複製する
author: しゃまとん
date: 2014-03-28T10:57:08+00:00
url: /posts/48
categories:
  - cocos2dx
  - プログラミング関連

---
どうもお世話になります。

しゃまとんです。

Cocos2dxのプロジェクトは作成しっぱなしだと、MyAppとなっているので、
自分が決めたプロジェクト名に変えたいと思い、色々調べた時の備忘録です。

<!--more-->

■iOS

こちらのサイトがわかりやすかったのでリンクだけ。  
ついでのこのタイミングでBundle Identiferも変えておく。あとで色々ややこしいだろうから…

{{< blogcard url="http://webhoric.com/apple/mac/xcode-mac/xcode-project-copy" >}}

Xcodeのプロジェクト名、Bundle Identifier、アプリ名の関係  
{{< blogcard url="http://d.hatena.ne.jp/paraches/20130211" >}}


■Android

Androidは手動で色々と変更した。

**1. AndroidManifest.xml**  
`package="jp.co.mydomain"`  
↓  
`package="com.shamaton.NewAppName"`

`<activity android:name=".MyApp"`  
↓  
`<activity android:name=".NewAppName"`

**2. build.xml**  
`<project name="MyApp" default="help">`  
↓  
`<project name="NewAppName" default="help">`

**3. build_native.sh**  
`APPNAME="MyApp"`  
`APPNAME="NewAppName"`

**4. res/values/string.xml**  
`<string name="app_name">MyApp</string>`  
↓  
`<string name="app_name">NewAppName</string>`

**5. jp/co/mydomainパッケージからcom/shamaton/xxxxx(プロジェクト名)に変更**  
パッケージ内に置かれている、MyApp.javaをxxxxx.javaにする

このとき、Classes内のファイルでAndroid個別の処理をしている場合は、そちらも忘れないように変更する。  
`#define CLASS_NAME "jp/co/mydomain/MyApp"`  
↓  
`#define CLASS_NAME "com/shamaton/xxxxx/NewAppName"`

これらを行って、Xcodeならプロジェクトファイルをクリック、
eclipseなら既存のandroidプロジェクトの取り込みでビルドできるようになります。