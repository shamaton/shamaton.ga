---
title: '[Server] nginx + golangな環境で502 Bad Gatewayになった時のこと'
author: しゃまとん
type: post
date: 2018-02-17T11:52:29+00:00
url: /archives/497
featured_image: /wp-content/uploads/2017/12/Nginx_logo.png
categories:
  - go
  - Linux
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

Goを使って、Webアプリケーションを作る際に80ポートを直接設定できない（しづらい）ので何か公開する際にはnginxでリバースプロキシさせたりしています。

で、nginx側の設定を行ってアクセスしたところ502になってしまいました。

nginx側の設定（nginx.conf）はこんな感じ。ちなみにEchoを使っているためfcgi.Serveではありません。（golnag nginx辺りでググると、よく結果に出る）

fcgi.Serveを使っているならそのまま下記サイトの設定で行けそうです。  
<a href="http://umegusa.hatenablog.jp/entry/2015/02/22/025832" target="_blank" rel="noopener">Nginx + Golang でWebアプリケーション開発を試してみた</a>

<pre class="lang:default decode:true" title="nginx.conf">listen 80;
server_name hostname;

location / {
     proxy_pass http://127.0.0.1:9999;
}</pre>

本題に戻って、これですがGoのアプリケーションに繋いでるだけです。なんだろーなと思いエラーログを確認してみると、このようなものがポロポロと出ていました。

<pre class="lang:default decode:true">[error] 24168#0: *32 connect() failed (111: Connection refused) while connecting to upstream,
client: 118.238.205.46, server: hostname, request: "GET /favicon.ico HTTP/1.1", upstream: "http://127.0.0.1:9999/favicon.ico",
host: "example.com", referrer: "http://example.com/"</pre>

このエラーですが、どうやらselinuxの設定によって起こる可能性があるようです。  
そちらに関してもログを見てみました。

<pre class="lang:default decode:true ">type=SYSCALL msg=audit(1512955800.388:1527): arch=c000003e syscall=42 
success=no exit=-13 a0=4 a1=556160ba11f0 a2=10 a3=7ffef19c2120 items=0 
ppid=10654 pid=24168 auid=4294967295 uid=996 gid=993 euid=996 suid=996 
fsuid=996 egid=993 sgid=993 fsgid=993 tty=(none) ses=4294967295 comm="nginx" 
exe="/usr/sbin/nginx" subj=system_u:system_r:httpd_t:s0 key=(null)
type=AVC msg=audit(1512955800.405:1528): avc: denied { name_connect } for pid=24168 
comm="nginx" dest=9999 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:jboss_management_port_t:s0 
tclass=tcp_socket</pre>

何やらよくないログっぽいですね&#8230;  
これに対処するには指定のパラメータを変更すると良いようです。  
（selinuxを無効にしても同じ効果はあります）

<pre class="lang:default decode:true "># 確認
$ getenforce
Enforcing

# 変更
$ sudo setsebool -P httpd_can_network_connect 1</pre>

これで再度アクセスしてみると、502が解消されました。  
最初はnginxの設定が悪いと思って小一時間ハマっていました。  
同じような事象になっている方は試してみてくださいませ。

以上です。

■ 参考  
<a href="https://tech.mktime.com/entry/447" target="_blank" rel="noopener">nginx : Permission denied を解決</a>

&nbsp;