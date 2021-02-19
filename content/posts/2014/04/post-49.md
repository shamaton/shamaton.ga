---
title: cocos2dxのリリースビルド設定
author: しゃまとん
type: post
date: 2014-04-01T15:38:54+00:00
url: /archives/49
featured_image: /wp-content/uploads/2015/08/Icon-cocos.png
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

【Xcode】Appleにアプリを提出する時の注意点（デバックの違い）  
<http://albatrus.com/main/ios/5173>

■Android

**Android側の切り替えに関して、別記事を追加しています。**  
**[こちら][1]を参考にしてください。**

<del>Androidは、jni/Application.mk内で下記の記載を変更するといいです。</del>  
 <del>-DCOCOS2D_DEBUG=1</del>  
 <del>↓</del>  
 <del>-DCOCOS2D_DEBUG=0</del>

以上です。

 [1]: http://shamaton.orz.hm/blog/archives/89 "[cocos2dx]Androidのdebug,releaseビルドの切り替え"