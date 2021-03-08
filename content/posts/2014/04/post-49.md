---
title: cocos2dxのリリースビルド設定
author: しゃまとん
date: 2014-04-01T15:38:54+00:00
url: /posts/49
featured_image: /images/posts/2015/08/Icon-cocos.png
categories:
  - cocos2dx
  - プログラミング関連

---
お世話になります。

しゃまとんです。

アプリを公開する前にリリースビルドして問題ないか確認する必要があります。  
毎回、検索してしまうので、参考にさせて頂いているリンクとメモをしておきます。

<!--more-->

iOSとAndroidで設定が別々です。

■iOS

{{< blogcard url="https://albatrus.com/entry/main/ios/5173" >}}

■Android

**Android側の切り替えに関して、別記事を追加しています。**  
{{< blogcard url="/posts/89" >}}
**[こちら][1]を参考にしてください。**

~~Androidは、jni/Application.mk内で下記の記載を変更するといいです。  
 -DCOCOS2D_DEBUG=1  
 ↓  
 -DCOCOS2D_DEBUG=0~~

以上です。
