---
title: '[cocos2dx]ver3.xへのUpdate時の注意'
author: しゃまとん
date: 2014-08-24T15:23:58+00:00
url: /posts/56
categories:
  - cocos2dx
  - Mac

---
お世話になります。

しゃまとんです。

最近、めっきり遠のいていましたが、cocos2d-xも知らぬ間に3.x系の安定版が出ているということで、
ダウンロードして、素のプロジェクトをビルドしてみることにしました。

他のサイト様を参考にさせていただいたのですが、つまずいたところがあったので、メモ書きしておきます。

versionは3.2への乗り換えでした。

<!--more-->

公式サイトからダウンロードして、プロジェクトを作成まではすんなりいけたのですが、ビルドを実行すると失敗していました。

エラーコードは

```shell
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/libtool failed with exit code 1

```

となっていました。

どうやら、こちらXcodeのバージョンが古いとエラーとなってしまうようでした。  
なので、同じような現状になった方はXcodeを更新するとよいかと思います。

以上です。

■ 参考  
{{< blogcard url="http://discuss.cocos2d-x.org/t/solved-cocos2d-x-3-2-ios-build-errors/16226" >}}