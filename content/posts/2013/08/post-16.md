---
title: 'ServersMan VPSでWebサーバー構築#5(WordPress)'
author: しゃまとん
date: 2013-08-17T15:55:35+00:00
url: /posts/16
categories:
  - Web関連
  - サーバー構築

---
さて、いよいよ最後のステップWordPressの導入です。  
WordPressは導入するだけで簡単にブログサイトを開設できるソフトウェアです。

様々なプラグインやテーマがあり、個性的なブログを作ることができます。

最後も同様にスーパーユーザで作業を行います。  
それでは以下、手順です。  
<!--more-->

1. データベース作成  
まずはWordPress用のデータベースを作成します。
```shell
mysql -u root -p # rootでログイン
```
パスワードを入力して、mysqlに入ったら下記の通りに実行します。
```shell
mysql> create database wordpress;　# wordpress用データベース作成
Query OK, 1 row affected (0.00 sec)

mysql> grant all privileges on wordpress.* to wordpress@localhost identified by '*******';　# wordpressユーザー作成、******にはパスワードを入れる

Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye<
```

  2. WordPressインストール  
    下記を順番に実行。wordpressをダウンロードして/var/wwwに置きます。
     
```shell
yum -y install php-mysql
wget http://ja.wordpress.org/latest-ja.zip
unzip latest-ja.zip
mv wordpress /var/www/
chown -R apache:apache /var/www/wordpress/
```

  3. WordPress設定  
    まずは設定ファイルをコピーします。
```shell
cp /var/www/wordpress/wp-config-sample.php /var/www/wordpress/wp-config.php
vim /var/www/wordpress/wp-config.php
```
    
下記のように設定します。  
秘密鍵は[こちら][1]にアクセスして生成し、出力されているものを貼り付けましょう。
    
```text
/** WordPress のためのデータベース名 */
define('DB_NAME', 'wordpress');

/** MySQL データベースのユーザー名 */
define('DB_USER', 'wordpress');

/** MySQL データベースのパスワード */
define('DB_PASSWORD', '******'); # wordpressユーザーのパスワード

/**#@+
 * 認証用ユニークキー
 *
 * それぞれを異なるユニーク (一意) な文字列に変更してください。
 * {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org の秘密鍵サービス} で自動生成することもできます。
 * 後でいつでも変更して、既存のすべての cookie を無効にできます。これにより、すべてのユーザーを強制的に再ログインさせることに
なります。
 *
 * @since 2.6.0
 */
# ここをまるっと上書きする
define('AUTH_KEY',         '****************************************************************');
define('SECURE_AUTH_KEY',  '****************************************************************');
define('LOGGED_IN_KEY',    '****************************************************************');
define('NONCE_KEY',        '****************************************************************');
define('AUTH_SALT',        '****************************************************************');
define('SECURE_AUTH_SALT', '****************************************************************');
define('LOGGED_IN_SALT',   '****************************************************************');
define('NONCE_SALT',       '****************************************************************');
/**#@-*/
```

4. httpd設定  
wordpress用のconfを作成して設定します。  
とりあえず、下記の設定でアクセスできるようになります。  
もし、トップでアクセスしたい場合は
   `/etc/httpd/conf/httpd.conf`の中のDocumentRootと<directory>を`/var/www/wordpress`
   に変更すればOKです。
```text
echo Alias /wordpress /var/www/wordpress > /etc/httpd/conf.d/wordpress.conf
```
   <pre class="brush: bash; gutter: true">echo Alias /wordpress /var/www/wordpress > /etc/httpd/conf.d/wordpress.conf</pre>

設定を反映させましょう。

<pre class="brush: bash; gutter: true">/etc/init.d/httpd reload</pre>

5. wordpress開設  
http://設定したアドレス/wordpressにアクセスしてみましょう。  
画面に表示されている通りにインストールボタンを押せば開設できます！  
これでひと通り完了です。お疲れ様でした！

 [1]: https://api.wordpress.org/secret-key/1.1/salt/