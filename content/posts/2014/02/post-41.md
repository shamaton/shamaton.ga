---
title: cocos2dxでnendを実装するためのリンク集
author: しゃまとん
date: 2014-02-25T14:12:50+00:00
url: /posts/41
categories:
  - cocos2dx
  - プログラミング関連

---
どうもお世話になります。

しゃまとんです。

cocos2dxを使って、アプリを作成しているのですが、広告を実装してみたのでその時に参考になったリンク集です。

まずはこれを見て、その通りに実装する。

<!--more-->
  
{{< blogcard url="http://www.appbank.net/2013/08/29/iphone-news/647273.php" >}}

次に、こちらでAndroid側の対応をする

{{< blogcard url="http://kishi777.blog.fc2.com/blog-entry-836.html" >}}

多分このままでビルドしようとするとライブラリでエラーしてビルドできない。  
iPhoneはこちらを参考にする。

{{< blogcard url="http://albatrus.com/entry/main/cocos2d/5286" >}}

Androidはこの辺を参考に。

{{< blogcard url="http://inujirushi123.blog.fc2.com/blog-entry-73.html" >}}

やっていて、よくわからなかったが、eclipseを再起動したりするとビルド通りました。

これらの記事を作ってくださってる皆様に感謝です。

追記：Androidで画面最下部に置きたい場合は下記のようにする。

MyApp.java

```java
adView = new NendAdView(activity, 140270, "4d2662319ead5ff52dc13a863ab12bde6501b492");

final FrameLayout.LayoutParams adParams = new FrameLayout.LayoutParams(WC,WC);
activity.addContentView(adView, adParams);

adView.loadAd();
```