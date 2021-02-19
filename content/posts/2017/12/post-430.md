---
title: '[Server] awstatsをHTTPSに対応させる'
author: しゃまとん
type: post
date: 2017-12-15T14:20:23+00:00
url: /archives/430
featured_image: /wp-content/uploads/2017/07/awstats_logo.png
categories:
  - Web関連
  - サーバー構築

---
お世話になっております。  
しゃまとんです。

少し前にサイトをhttps化したのですが、その影響でアクセスログ解析がうまく動かなくなっていました。

[awstats][1]を使っているのですが、こちらも合わせて対応する必要があるようです。  
使っている方は参考になると幸いです。

原因としてはhttps化した事によりアクセスログの出力先が変わったことです。  
よって、その辺りの設定を変更することで修正できます。

まずは/etc/httpd/conf.d/ssl.confを下記のように変更します。  
もともとログ出力していたところに戻す感じですね。

<pre class="lang:default decode:true" title="/etc/httpd/conf.d/ssl.conf">ErrorLog logs/ssl_error_log
↓　こちらに変更する
ErrorLog logs/error_log

これを追加する
LogFormat "%h %l %u %t \"%!414r\" %&gt;s %b \"%{Referer}i\" \"%{User-Agent}i\""

TransferLog logs/ssl_access_log
↓　こちらに変更する
TransferLog logs/access_log</pre>

次にawstatsの設定ファイルを下記のようにしておきます。  
多分ssl.confと同じ/etc/httpd/conf.dにあるかなと。

<pre class="lang:default decode:true " title="awstats.conf">UseHTTPSLinkForUrl=""
↓ 変更する　
UseHTTPSLinkForUrl="/"</pre>

関係ないかもですが、httpdをrestartまたはreloadしておきます。

<pre class="lang:default decode:true ">service httpd restart
もしくは
service httpd reload</pre>

自分のところでは変更直後に治っているかわからなかったので、集計が走るまで（一日とか）待ってみて確認するのがいいかもしれません。

以上です。

参考  
<a href="http://fedorasrv.com/bbshtml/webpatio/3373.shtml" target="_blank" rel="noopener">[掲示板] No.3373 AWStatsについて</a>

 [1]: https://awstats.sourceforge.io/