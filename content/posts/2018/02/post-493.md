---
title: '[Server] nginx + golang な環境をhttps化する'
author: しゃまとん
date: 2018-02-26T15:11:10+00:00
url: /posts/493
featured_image: /images/posts/2017/07/key_close.png
categories:
  - Linux
  - Web関連

---
お世話になっております。  
しゃまとんです。

前回の記事につづいてnginx + golangな環境をhttps化してみました。

{{< blogcard url="/posts/497" >}}

今回もLet's Encryptを使ってhttps化していきます。  
環境を作るにあたってこちらがとても参考になりました。

{{< blogcard url="https://qiita.com/HeRo/items/f9eb8d8a08d4d5b63ee9" >}}

まずはサーバーでletsentrypt(certbot)を取得して証明書の発行を行います。  
私の場合はroot直下で作業しました。

```shell
git clone https://github.com/certbot/certbot
cd certbot
./certbot-auto certonly --standalone -t
```

問題なければ`Congraturation!!`と表示されます。  
次にnginxのtls周りを設定します。nginx.confのtls部分を下記のようにしました。  
初期状態だとコメントアウトされているので外して証明書やproxy_passを設定します。

```text
# Settings for a TLS enabled server.
#
    server {
        listen       443 ssl;
        server_name  _;

        ssl_certificate "/etc/letsencrypt/live/example.com/fullchain.pem";
        ssl_certificate_key "/etc/letsencrypt/live/example.com/privkey.pem";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        proxy_pass http://127.0.0.1:9999;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

}
```

設定後は忘れずリロードか再起動を行いましょう。  
これでhttpsでのアクセスが可能になっているはずです。

```shell
service nginx restart
service nginx reload
```

次にnginxは80ポートをバインドするので、Let's Encrypt側を–webrootを使った形式に変更しておきます。
（これは最初からこちらでもよかったかも）

更新時の確認用パスとなるディレクトリを作成し、renewalしてみます。  
実行後に`Congraturation!!`とでればOKです。

```shell
mkdir /var/letsencrypt
./certbot-auto certonly --webroot -w /var/letsencrypt -d example.com --agree-tos --force-renewal -n
```

再度nginx側の設定を変更します。  
以降はhttpでのアクセスは全てhttpsにリダイレクトするようにしておきます。  
ただしletsentryptはhttpsを利用するため、先程のディレクトリを指定し対象のアドレスのみHTTPを許可しておきます。

```text
server {
        listen       80;
        server_name  hostname;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location ^~ /.well-known {
                root /var/letsencrypt;
        }

        location / {
        # Redirect all HTTP requests to HTTPS with a 301 Moved Permanently response.
        return 301 https://$host$request_uri;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

設定したら再度nginxをリロードまたは再起動し、renewを実行してみます。

```shell
service nginx restart
./certbot-auto renew --force-renewal
```

ログがこんな感じならOKでしょう。

```text
Saving debug log to /var/log/letsencrypt/letsencrypt.log

-------------------------------------------------------------------------------
Processing /etc/letsencrypt/renewal/exmaple.com.conf
-------------------------------------------------------------------------------
Plugins selected: Authenticator webroot, Installer None
Renewing an existing certificate
Performing the following challenges:
http-01 challenge for exmaple.com
Waiting for verification...
Cleaning up challenges

-------------------------------------------------------------------------------
new certificate deployed without reload, fullchain is
/etc/letsencrypt/live/exmaple.com/fullchain.pem
-------------------------------------------------------------------------------

-------------------------------------------------------------------------------

Congratulations, all renewals succeeded. The following certs have been renewed:
/etc/letsencrypt/live/exmaple.com/fullchain.pem (success)
-------------------------------------------------------------------------------
```

もしうまくいかない場合は上記の例だと/var/letsencrypt/.well-known配下にindex.htmlとか作ってみて、
curlでURLを叩いてみるといいかもしれません。  
というのも、renewは何回か失敗すると一定時間実行できなくなるようなので。。

あとはファイルがあるはずなのに403 Forbiddenになってしまうような場合は、
SELinuxが影響している場合もありますのでgetenforceして確かめてみるといいかもしれません。

{{< blogcard url="http://kakakazuma.hatenablog.com/entry/2015/04/24/235812">}}

必要ならcronも設定しておくとよいですね。

```shell
crontab -e
00 05 15 * * /root/certbot/certbot-auto renew
```

これで一応ですが、nginx + golangな環境にhttpsを導入することができました。  
以上です。

■ 参考

{{< blogcard url="/posts/426" >}}