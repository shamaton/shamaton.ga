---
title: XcodeをVer5.1にしたらCocos2dxをビルドできない
author: しゃまとん
date: 2014-03-13T02:51:14+00:00
url: /posts/44
categories:
  - cocos2dx
  - Mac

---
お世話になっております。

しゃまとんです。

先日、使っているmacを更新した時にXcodeが、updateされました。  
特に気にせず更新したら、cocos2dxのビルドができなくなりました。

<!--more-->

cocos2dx内のソースでエラー(Semantic issue)が出るようになります。  
version2.2.0を使用されている方はご注意ください。

間違ってXcodeをupdateしてしまった、もしくはinstallしたらversion5.1のときは5.0系をインストールするようにしたほうがよいです。
(※apple審査でリジェクトされる…かも？)

戻す手順は簡単です。

  1. Finderからアプリケーションを開く
  2. Xcode.appをゴミ箱へ
  3. [apple developer][1]にアクセス
  4. Xcodeで検索して、Xcode 5.0.2をダウンロード
  5. クリックしてinstall

これで、元通りビルドできるようになります。


 [1]: https://developer.apple.com/downloads/index.action