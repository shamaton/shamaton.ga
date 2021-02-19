---
title: Cocos2d-xのAndroidアプリ開発環境をWindowsに作成する
author: しゃまとん
type: post
date: 2013-08-20T15:43:17+00:00
url: /archives/21
categories:
  - メモ

---
開発環境の構築はよくmacの文献では見るけど、Winはあまり見られない。  
Macでの開発が主流なのかな…?

ということで、かなり自分用だと思うけどメモ。  
eclipseを使用して、ビルド環境を作成する。

<!--more-->

下記サイトを参考にしつつ、構築してみた。  
Cocos2dxとは何か？についても説明してくれている。  
[Cocos2dxでiOS／Androidの2Dゲーム開発を始めるには][1]

&nbsp;

■各種インストール  
Androidでの開発に各種必要なライブラリを取得する。  
現在(2013/08)はandroid sdkでの配布ではなくADB bundleという、eclipseをAndroidアプリ開発用にチューニングしてあるものを使用する。  
また、jdk(java development kit)のバージョンは6を推奨しているため、どこかからバージョン6を取得する。  
※現在、バージョン6の取得にはユーザー登録が必要となっている。  
jdk(x86が32bit版)  
[http://www.oracle.com/technetwork/java/javase/downloads/index.html  
][2] adb bundle(32bitと64bitを確認する)  
[http://developer.android.com/sdk/index.html  
][3] Android NDK  
[http://developer.android.com/tools/sdk/ndk/index.html  
][4] cocos2d-x  
[http://www.cocos2d-x.org/  
][5] cygwin(プロジェクト作成時に必要)  
<http://www.cygwin.com/>

jdkはインストール。cygwinもsetup.exeをそのまま実行すればOK。  
他3つは解凍するだけなので、自分はadb-bundleをcドライブ直下に置き、その中にNDKとCocos2d-xを置いた。

■create-android-project.batの修正  
そのままだと使えないので、インストールした状況に応じて適宜変更する。  
batを適当なテキストエディタで開き、該当箇所を修正する。  
・set _CYGBIN=c:\cygwin\bin  
・set _ANDROIDTOOLS=c:\adt-bundle-windows\sdk\tools  
・set _NDKROOT=c:\adt-bundle-windows\android-ndk-r8e

■アイコンの追加  
プロジェクト作成時にアイコンが入ってないので、/res/drawable-xxx/に格納する。  
ココを参照。  
[Android @drawable/iconでハマる][6]

ここまで設定できたら後は下のページ通りにプロジェクトをインポートしたら、ビルドできるはず。  
Eclipseへのインポートから参照  
<http://www.atmarkit.co.jp/ait/articles/1302/25/news017_2.html>

&nbsp;

 [1]: http://www.atmarkit.co.jp/ait/articles/1302/25/news017.html
 [2]: http://www.oracle.com/technetwork/java/javase/downloads/index.html
 [3]: http://developer.android.com/sdk/index.html
 [4]: http://developer.android.com/tools/sdk/ndk/index.html
 [5]: http://www.cocos2d-x.org/
 [6]: http://blog.livedoor.jp/saladismaindish/archives/3988308.html