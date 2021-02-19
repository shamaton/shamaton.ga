---
title: '[Server] letsencryptを使ってhttps化しました'
author: しゃまとん
type: post
date: 2017-08-04T16:11:25+00:00
url: /archives/426
featured_image: /wp-content/uploads/2017/07/key_close.png
categories:
  - Linux
  - Web関連
  - サーバー構築

---
お世話になっております。  
しゃまとんです。

少し前から気になっていたホームページのHTTPS化ですが、なんとなくやる気持ちになったので対応してみました。

HTTPSにするにあたって証明書が必要になるのですが、以前はオレオレ証明書だったりどこかの発行所で購入する必要があったのですが、[letsencrypt][1]というサービスが開始されたことで誰でも無料でHTTPS化を正式に行えるようになりました。

ちなみに2018年からワイルドカードにも対応するそうです！！

ということでletsencryptを使って、HTTPS化した手順をまとめておきます。  
今回はCentOS6でapacheを利用しているWebサイトに対しての例になります。

まず、letsencryptを利用するにはpython2.7が必要なので、インストールします。  
rootにて作業しています。

<pre class="lang:sh decode:true">$ wget http://www.python.org/ftp/python/2.7.11/Python-2.7.11.tgz
$ tar xvzf Python-2.7.11.tgz
$ cd Python-2.7.11
$ ./configure --with-threads --enable-shared --prefix=/usr/local
$ make
$ sudo make install

# 確認
$ python --version
Python 2.7.11</pre>

すんなりいけばバージョンが更新されますが、自分の場合はエラーが表示されてしまったので、下記のサイトを参考に対処しました。ちなみにエラーはこれ。

<pre class="lang:default decode:true ">error while loading shared libraries: libpython2.7.so.1.0: cannot open shared object file: No such file or directory</pre>



次にletsencryptを設定していきます。今回は証明書の発行だけを行います。  
実行すると色々聞かれるの答えます。その後、確認が行われ成功すると証明書が作成されます。

<pre class="lang:default decode:true"># git clone https://github.com/letsencrypt/letsencrypt
# cd letsencrypt
# ./letsencrypt-auto --help
# ./letsencrypt-auto certonly --webroot -w /var/www/ -d shamaton.orz.hm
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): example@email.com
# メールアドレスを入力

-------------------------------------------------------------------------------
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.1.1-August-1-2016.pdf. You must agree
in order to register with the ACME server at
https://acme-v01.api.letsencrypt.org/directory
-------------------------------------------------------------------------------
(A)gree/(C)ancel: A
# Aを入力

-------------------------------------------------------------------------------
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about EFF and
our work to encrypt the web, protect its users and defend digital rights.
-------------------------------------------------------------------------------
(Y)es/(N)o: N
# Nを入力
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for shamaton.orz.hm
Using the webroot path /var/www for all unmatched domains.
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at
   /etc/letsencrypt/live/shamaton.orz.hm/fullchain.pem. Your cert will
   expire on 2017-10-08. To obtain a new or tweaked version of this
   certificate in the future, simply run letsencrypt-auto again. To
   non-interactively renew *all* of your certificates, run
   "letsencrypt-auto renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

# ls /etc/letsencrypt/live/shamaton.orz.hm/
README  cert.pem  chain.pem  fullchain.pem  privkey.pem</pre>

無事、証明書が発行されました！  
続いて、apacheの設定を行います。/etc/httpd/conf.d/ssl.confを開き、各項目を設定しておきます。

<pre class="lang:default decode:true" title="ssl.conf">SSLCertificateFile /etc/letsencrypt/live/shamaton.orz.hm/cert.pem
SSLCertificateKeyFile /etc/letsencrypt/live/shamaton.orz.hm/privkey.pem
SSLCertificateChainFile /etc/letsencrypt/live/shamaton.orz.hm/chain.pem</pre>

iptablesが有効になっている場合、HTTPSをポートを解放しましょう。  
追加コマンドか、ファイルを直接編集します。（/etc/sysconfig/iptables）  
内容は使っているサーバによって読み替えてください。

<pre class="lang:default decode:true">-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 443 -j ACCEPT</pre>

apacheとiptablesを再起動しておきます。

<pre class="lang:sh decode:true">$ sudo apachectl configtest
$ sudo apachectl restart
$ sudo service iptables restart
$ sudo service iptables status</pre>

ここでHTTPSでアクセスできるか確認しておきます。  
一通りアクセスしてみて問題なさそうだったら、httpをhttpsにリダイレクトするようにしておきます。  
/etc/httpd/conf.dに新しくconfファイルを追加します。

<pre class="lang:sh decode:true">$ sudo vi /etc/httpd/conf.d/rewrite.conf

&lt;ifModule mod_rewrite.c&gt;
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R,L]
&lt;/ifModule&gt;

$ sudo apachectl restart</pre>

わざとhttpでアクセスしてhttpsにリダイレクトすることを確認します。  
以上でhttps化は完了です。

最後にcronで証明書を定期更新するようにしておきます。letsencryptの証明書は3ヶ月ほどで切れてしまうようなので、cron等で設定しておくとよいです。  
更新されたかわかるようにメールアドレスを添えておきました。

<pre class="lang:default decode:true"># crontab -e
MAILTO="example@email.com"
00 03 07 * * /root/letsencrypt/letsencrypt-auto renew --force-renew</pre>

ちなみに、renewに成功するとこんな感じです。

<pre class="lang:default decode:true"># /root/letsencrypt/letsencrypt-auto renew --force-renew
Saving debug log to /var/log/letsencrypt/letsencrypt.log

-------------------------------------------------------------------------------
Processing /etc/letsencrypt/renewal/shamaton.orz.hm.conf
-------------------------------------------------------------------------------
Renewing an existing certificate
Performing the following challenges:
http-01 challenge for shamaton.orz.hm
Waiting for verification...
Cleaning up challenges

-------------------------------------------------------------------------------
new certificate deployed without reload, fullchain is
/etc/letsencrypt/live/shamaton.orz.hm/fullchain.pem
-------------------------------------------------------------------------------

Congratulations, all renewals succeeded. The following certs have been renewed:
  /etc/letsencrypt/live/shamaton.orz.hm/fullchain.pem (success)</pre>

あとで気づいたのですが、画像等がhttpのURLで参照していると「保護された通信」と表示されなくなってしまうようです。この辺は一括設定かなにか対応が必要ですね。

以上です。

■ 参考  
<a href="http://inatim.com/centos-python2-7-11/" target="_blank" rel="noopener">Linux(CentOS)にPython 2.7.11をインストール</a>  
<a href="http://qiita.com/sue71/items/100004b704b9ff129b09" target="_blank" rel="noopener">apacheでhttpへのアクセスをhttpsへ自動リダイレクトする</a>  
<a href="https://weblabo.oscasierra.net/apache-httpd22-ssl-centos6/" target="_blank" rel="noopener">Apache httpd 2.2 に HTTPS (SSL/TLS) の設定をする (CentOS 6)</a>  
<a href="https://14code.com/blog/20160209_1143" target="_blank" rel="noopener">Let’s Encryptで証明書を発行してWordPressをSSL化する</a>

 [1]: https://letsencrypt.jp/