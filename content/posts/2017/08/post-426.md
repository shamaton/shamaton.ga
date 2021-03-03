---
title: '[Server] letsencryptを使ってhttps化しました'
author: しゃまとん
date: 2017-08-04T16:11:25+00:00
url: /posts/426
featured_image: /images/posts/2017/07/key_close.png
categories:
  - Linux
  - Web関連
  - サーバー構築

---
お世話になっております。  
しゃまとんです。

少し前から気になっていたホームページのHTTPS化ですが、なんとなくやる気持ちになったので対応してみました。

HTTPSにするにあたって証明書が必要になるのですが、
以前はオレオレ証明書だったりどこかの発行所で購入する必要があったのですが、letsencryptというサービスが開始されたことで誰でも無料でHTTPS化を正式に行えるようになりました。

{{< blogcard url="https://letsencrypt.org/ja/" >}}

ちなみに2018年からワイルドカードにも対応するそうです！！

ということでletsencryptを使って、HTTPS化した手順をまとめておきます。  
今回はCentOS6でapacheを利用しているWebサイトに対しての例になります。

まず、letsencryptを利用するにはpython2.7が必要なので、インストールします。  
rootにて作業しています。

```shell
$ wget http://www.python.org/ftp/python/2.7.11/Python-2.7.11.tgz
$ tar xvzf Python-2.7.11.tgz
$ cd Python-2.7.11
$ ./configure --with-threads --enable-shared --prefix=/usr/local
$ make
$ sudo make install

# 確認
$ python --version
Python 2.7.11
```

すんなりいけばバージョンが更新されますが、自分の場合はエラーが表示されてしまったので、
下記のサイトを参考に対処しました。ちなみにエラーはこれ。

```text
error while loading shared libraries: libpython2.7.so.1.0: cannot open shared object file: No such file or directory

```

次にletsencryptを設定していきます。今回は証明書の発行だけを行います。  
実行すると色々聞かれるの答えます。その後、確認が行われ成功すると証明書が作成されます。

```shell
# rootで実行
git clone https://github.com/letsencrypt/letsencrypt
cd letsencrypt
./letsencrypt-auto --help
./letsencrypt-auto certonly --webroot -w /var/www/ -d shamaton.orz.hm
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
README  cert.pem  chain.pem  fullchain.pem  privkey.pem
```

無事、証明書が発行されました！  
続いて、apacheの設定を行います。`/etc/httpd/conf.d/ssl.conf`を開き、各項目を設定しておきます。

```text
SSLCertificateFile /etc/letsencrypt/live/shamaton.orz.hm/cert.pem
SSLCertificateKeyFile /etc/letsencrypt/live/shamaton.orz.hm/privkey.pem
SSLCertificateChainFile /etc/letsencrypt/live/shamaton.orz.hm/chain.pem
```

iptablesが有効になっている場合、HTTPSをポートを解放しましょう。  
追加コマンドか、ファイルを直接編集します。（/etc/sysconfig/iptables）  
内容は使っているサーバによって読み替えてください。

```text
-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 443 -j ACCEPT
```

apacheとiptablesを再起動しておきます。

```shell
$ sudo apachectl configtest
$ sudo apachectl restart
$ sudo service iptables restart
$ sudo service iptables status
```

ここでHTTPSでアクセスできるか確認しておきます。  
一通りアクセスしてみて問題なさそうだったら、httpをhttpsにリダイレクトするようにしておきます。  
/etc/httpd/conf.dに新しくconfファイルを追加します。

```shell
$ sudo vi /etc/httpd/conf.d/rewrite.conf

<ifModule mod_rewrite.c>
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R,L]
</ifModule>

$ sudo apachectl restart
```

わざとhttpでアクセスしてhttpsにリダイレクトすることを確認します。  
以上でhttps化は完了です。

最後にcronで証明書を定期更新するようにしておきます。
letsencryptの証明書は3ヶ月ほどで切れてしまうようなので、cron等で設定しておくとよいです。  
更新されたかわかるようにメールアドレスを添えておきました。

```shell
# rootで実行
crontab -e

# 追加
MAILTO="example@email.com"
00 03 07 * * /root/letsencrypt/letsencrypt-auto renew --force-renew
```

ちなみに、renewに成功するとこんな感じです。

```shell
# rootで実行
root/letsencrypt/letsencrypt-auto renew --force-renew

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
  /etc/letsencrypt/live/shamaton.orz.hm/fullchain.pem (success)
```

あとで気づいたのですが、画像等がhttpのURLで参照していると「保護された通信」と表示されなくなってしまうようです。
この辺は一括設定かなにか対応が必要ですね。

以上です。

■ 参考
{{< blogcard url="http://qiita.com/sue71/items/100004b704b9ff129b09">}}
{{< blogcard url="https://weblabo.oscasierra.net/apache-httpd22-ssl-centos6/">}}
{{< blogcard url="https://14code.com/blog/20160209_1143">}}