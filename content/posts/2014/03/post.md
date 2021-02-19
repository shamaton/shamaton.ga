---
title: cocos2dxで作ったアプリのアイコンを変更するには
author: しゃまとん
type: post
date: 2014-03-25T01:18:03+00:00
featured_image: /wp-content/uploads/2015/08/Icon-cocos.png
is_comment_form_freeze:
  - on
comment_form_freeze_message:
  - ' '
categories:
  - cocos2dx
  - プログラミング関連

---
どうもお世話になります。

しゃまとんです。

cocos2dxでアプリの作成をしているのですが、アイコンをそろそろ変えようと思い、調べた時のメモです。

<!--more-->

  
プロジェクトを作成した際に、デフォルトのアイコンを用意してくれるので、そこを上書きします。

アイコンの変更はOSごとのフォルダ内にアイコンが設定されているので、個別に変更する必要があります。  
また、OSによって必要なサイズか違うようなので、リサイズなどして用意しましょう。

&nbsp;

■iOS

下記フォルダに格納されています。  
(プロジェクト名)/proj.ios

デフォルトで設定されているアイコンサイズを用意して、新しいアイコンで上書きします。  
ファイル名に付いている数字のサイズでよいです。  
Icon-144.png  
Icon-114.png  
Icon-72.png  
Icon-57.png

■Android

Androidではアイコンを指定サイズのフォルダで上書きします。  
(プロジェクト名)/proj.android

drawable-ldpi/icon.png (36×36)  
drawable-mdpi/icon.png (48×48)  
drawable-hdpi/icon.png (72×72)

これでインストールしてみるとアイコンが新しいものに変更されていると思います。

■参考サイト

【xcode5】「アイコンの変更」と「光沢を削除する」を行う  
<http://albatrus.com/main/ios/5083>

解像度の違いとランチャーアイコンのサイズ &#8211; Android  
<http://blog.livedoor.jp/elss/archives/23375717.html>