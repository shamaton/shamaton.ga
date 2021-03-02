---
title: '[Server] awstatsをHTTPSに対応させる'
author: しゃまとん
date: 2017-12-15T14:20:23+00:00
url: /posts/430
featured_image: /images/posts/2017/07/awstats_logo.png
categories:
  - Web関連
  - サーバー構築

---
お世話になっております。  
しゃまとんです。

少し前にサイトをhttps化したのですが、その影響でアクセスログ解析がうまく動かなくなっていました。

awstatsを使っているのですが、こちらも合わせて対応する必要があるようです。  
使っている方は参考になると幸いです。

{{< blogcard url="https://awstats.sourceforge.io/">}}

原因としてはhttps化した事によりアクセスログの出力先が変わったことです。  
よって、その辺りの設定を変更することで修正できます。

まずは`/etc/httpd/conf.d/ssl.conf`を下記のように変更します。  
もともとログ出力していたところに戻す感じですね。

```text
ErrorLog logs/ssl_error_log
↓　こちらに変更する
ErrorLog logs/error_log

これを追加する
LogFormat "%h %l %u %t \"%!414r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\""

TransferLog logs/ssl_access_log
↓　こちらに変更する
TransferLog logs/access_log
```

次にawstatsの設定ファイル(awstats.conf)を下記のようにしておきます。  
多分ssl.confと同じ`/etc/httpd/conf.d`にあるかなと。

```text
UseHTTPSLinkForUrl=""
↓ 変更する　
UseHTTPSLinkForUrl="/"
```

関係ないかもですが、httpdをrestartまたはreloadしておきます。

```shell
service httpd restart
# もしくは
service httpd reload
```

自分のところでは変更直後に治っているかわからなかったので、
集計が走るまで（一日とか）待ってみて確認するのがいいかもしれません。

以上です。

■ 参考
{{< blogcard url="http://fedorasrv.com/bbshtml/webpatio/3373.shtml">}}
