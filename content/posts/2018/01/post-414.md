---
title: '[Server] Let’s Encryptのrenewに失敗した'
author: しゃまとん
type: post
date: 2018-01-07T13:50:12+00:00
url: /archives/414
featured_image: /wp-content/uploads/2017/07/key_close.png
categories:
  - Web関連
  - サーバー構築

---
お世話になっております。  
しゃまとんです。

以前にLet&#8217;sEncryptを使ってホームページをHTTPSに対応させた記事をあげていました。

<https://shamaton.orz.hm/blog/archives/426>

記事の中でも自動で更新されるように設定していたのですが、ある日失敗していることに気づきました。

エラー時のログです。

<pre class="lang:default decode:true">/root/.local/share/letsencrypt/lib/python2.6/site-packages/cryptography/__init__.py:26: DeprecationWarning: Python 2.6 is no longer supported by the Python core team, please upgrade your Python. A future version of cryptography will drop support for Python 2.6
DeprecationWarning
Saving debug log to /var/log/letsencrypt/letsencrypt.log

-------------------------------------------------------------------------------
Processing /etc/letsencrypt/renewal/shamaton.orz.hm.conf
-------------------------------------------------------------------------------
Renewing an existing certificate
Performing the following challenges:
http-01 challenge for shamaton.orz.hm
Waiting for verification...
Cleaning up challenges
Attempting to renew cert from /etc/letsencrypt/renewal/shamaton.orz.hm.conf produced an unexpected error: Failed authorization procedure. shamaton.orz.hm (http-01): urn:acme:error:connection :: The server could not connect to the client to verify the domain :: Could not connect to shamaton.orz.hm. Skipping.

All renewal attempts failed. The following certs could not be renewed:
/etc/letsencrypt/live/shamaton.orz.hm/fullchain.pem (failure)
1 renew failure(s), 0 parse failure(s)

IMPORTANT NOTES:
- The following errors were reported by the server:

Domain: shamaton.orz.hm
Type: connection
Detail: Could not connect to shamaton.orz.hm

To fix these errors, please make sure that your domain name was
entered correctly and the DNS A record(s) for that domain
contain(s) the right IP address. Additionally, please check that
your computer has a publicly routable IP address and that no
firewalls are preventing the server from communicating with the
client. If you're using the webroot plugin, you should also verify
that you are serving files from the webroot path you provided.</pre>

どうやらアクセスが出来てないようでした。  
エラーに書いてあることを読んでいたらhttp-01（80ポート）で失敗しているみたいでした。  
（HTTPSで更新を行う場合はtls-sni-01を使うみたい）

でポートを確認したら80ポートを無効にしていたので、開けて試してみることに。

<pre class="lang:default decode:true">iptables -I INPUT 5 -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT</pre>

下記のように更新に成功したログが表示されました。

<pre class="lang:default decode:true ">/root/.local/share/letsencrypt/lib/python2.6/site-packages/cryptography/__init__.py:26: DeprecationWarning: Python 2.6 is no longer supported by the Python core team, please upgrade your Python. A future version of cryptography will drop support for Python 2.6
DeprecationWarning

# Firewall configuration written by system-config-firewall
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

対処としては一時敵に80ポート開けるか、443でchallengeするように変えるかですかね。。  
とりあえず原因がわかってよかったです。

以上です。

■ 参考  
<a href="https://community.letsencrypt.org/t/lets-encrypt-and-firewall-rules/18641/2" target="_blank" rel="noopener">Let’s Encrypt and Firewall rules</a>  
<a href="https://community.home-assistant.io/t/error-in-renewing-letsencrypt/13030/24" target="_blank" rel="noopener">Error in renewing Letsencrypt …?</a>