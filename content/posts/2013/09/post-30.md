---
title: cocos2d-html5の開発する環境を作る@win
author: しゃまとん
type: post
date: 2013-09-11T00:39:04+00:00
url: /posts/30
categories:
  - Windows
  - メモ

---
お世話になります。  
しゃまとんです。

ちょっと前にcocos2d-xの開発環境をwindowsで作成をしてみました。  
それは[こちら][1]。

&nbsp;

で、今回はブラウザ上でcocos2dを使ったアプリが公開できるcocos2d-html5があるというこで、まずはHello worldまでやってみました。  
お手軽に出力までいけました。

<!--more-->

■cocos2dのページで一式を取得  
下記ページからcocos2d-html5うんたらかんたらとある場所のダウンロードリンクから取得。  
http://cocos2d-x.googlecode.com/files/cocos2d-x-3.0alpha0-pre.zip  
↑2013.09時点ではコレ  
取得したら、任意の場所に解凍します。

■サンプル実行  
解凍したディレクトリからサンプルのある場所まで、以下の通りにアクセス。  
Cocos2d-html5-v2.1.5\Cocos2d-html5-v2.1.5\HelloHTML5World  
そのディレクトリの中にいかにもなファイルindex.htmlがあります。

実行の際、ローカルファイルにある状態で実行するにはchromeでは動かないようなので、firefox or safariが必要となります。  
<a title="firefox" href="http://www.mozilla.jp/firefox/" target="_blank" rel="noopener">firefox</a>  
<a title="safari" href="http://www.apple.com/jp/safari/" target="_blank" rel="noopener">safari</a>  
ブラウザをインストールし、起動したら、index.htmlをドラッグ&ドロップしてみましょう。

以下のような画面がでればサンプル実行完了です。

[<img src="https://shamaton.orz.hm/blog/images/posts/2013/09/hello_html5.jpg" alt="hello_html5" width="509" height="287" />][2]

■ここまで来たら  
次は開発していきたい。  
現在は英語のチュートリアルしかないので、ここを読んでいくしかないか・・・  
<a href="http://www.cocos2d-x.org/projects/cocos2d-x/wiki/Cocos2d-html5" target="_blank" rel="noopener">http://www.cocos2d-x.org/projects/cocos2d-x/wiki/Cocos2d-html5</a>

 [1]: http://shamaton.orz.hm/blog/posts/24 "Cocos2dx@WinでサンプルのJSゲームをビルドする#1"
 [2]: https://shamaton.orz.hm/blog/images/posts/2013/09/hello_html5.jpg