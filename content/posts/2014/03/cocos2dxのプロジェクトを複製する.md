---
title: cocos2dxのプロジェクトを複製する
author: しゃまとん
type: post
date: 2014-03-28T10:57:08+00:00
url: /archives/48
categories:
  - cocos2dx
  - プログラミング関連

---
どうもお世話になります。

しゃまとんです。

Cocos2dxのプロジェクトは作成しっぱなしだと、MyAppとなっているので、自分が決めたプロジェクト名に変えたいと思い、色々調べた時の備忘録です。

<!--more-->

■iOS

こちらのサイトがわかりやすかったのでリンクだけ。  
ついでのこのタイミングでBundle Identiferも変えておく。あとで色々ややこしいだろうから…

[Xcode 5] プロジェクトの複製 コピーのやり方  
<http://webhoric.com/apple/mac/xcode-mac/xcode-project-copy>

Xcodeのプロジェクト名、Bundle Identifier、アプリ名の関係  
<http://d.hatena.ne.jp/paraches/20130211>

■Android

Androidは手動で色々と変更した。

**1. AndroidManifest.xml**  
package=&#8221;jp.co.mydomain&#8221;  
↓  
package=&#8221;com.shamaton.NewAppName&#8221;

<activity android:name=&#8221;.MyApp&#8221;  
↓  
<activity android:name=&#8221;.NewAppName&#8221;

**2. build.xml**  
<project name=&#8221;MyApp&#8221; default=&#8221;help&#8221;>  
↓  
<project name=&#8221;NewAppName&#8221; default=&#8221;help&#8221;>

**3. build_native.sh**  
APPNAME=&#8221;MyApp&#8221;  
APPNAME=&#8221;NewAppName&#8221;

**4. res/values/string.xml**  
<string name=&#8221;app_name&#8221;>MyApp</string>  
↓  
<string name=&#8221;app_name&#8221;>NewAppName</string>

**5. jp/co/mydomainパッケージからcom/shamaton/xxxxx(プロジェクト名)に変更**  
パッケージ内に置かれている、MyApp.javaをxxxxx.javaにする

このとき、Classes内のファイルでAndroid個別の処理をしている場合は、そちらも忘れないように変更する。  
#define CLASS_NAME &#8220;jp/co/mydomain/MyApp&#8221;  
↓  
#define CLASS_NAME &#8220;com/shamaton/xxxxx/NewAppName&#8221;

&nbsp;

これらを行って、Xcodeならプロジェクトファイルをクリック、eclipseなら既存のandroidプロジェクトの取り込みでビルドできるようになります。