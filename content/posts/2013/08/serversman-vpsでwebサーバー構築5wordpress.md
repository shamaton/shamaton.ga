---
title: 'ServersMan VPSでWebサーバー構築#5(WordPress)'
author: しゃまとん
type: post
date: 2013-08-17T15:55:35+00:00
url: /archives/16
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
    まずはWordPress用のデータベースを作成します。</p> <pre class="brush: bash; gutter: true">mysql -u root -p # rootでログイン</pre>
    
    パスワードを入力して、mysqlに入ったら下記の通りに実行します。
    
    <pre class="brush: bash; gutter: true">mysql&gt; create database wordpress;　# wordpress用データベース作成
Query OK, 1 row affected (0.00 sec)

mysql&gt; grant all privileges on wordpress.* to wordpress@localhost identified by &#039;*******&#039;;　# wordpressユーザー作成、******にはパスワードを入れる

Query OK, 0 rows affected (0.00 sec)

mysql&gt; exit
Bye</pre>

  2. WordPressインストール  
    下記を順番に実行。wordpressをダウンロードして/var/wwwに置きます。</p> <pre class="brush: bash; gutter: true">yum -y install php-mysql
wget http://ja.wordpress.org/latest-ja.zip
unzip latest-ja.zip
mv wordpress /var/www/
chown -R apache:apache /var/www/wordpress/</pre>

  3. WordPress設定  
    まずは設定ファイルをコピーします。</p> <pre class="brush: actionscript3; gutter: true">cp /var/www/wordpress/wp-config-sample.php /var/www/wordpress/wp-config.php
vim /var/www/wordpress/wp-config.php</pre>
    
    下記のように設定します。  
    秘密鍵は[こちら][1]にアクセスして生成し、出力されているものを貼り付けましょう。
    
    <pre class="brush: bash; gutter: true">/** WordPress のためのデータベース名 */
define(&#039;DB_NAME&#039;, &#039;wordpress&#039;);

/** MySQL データベースのユーザー名 */
define(&#039;DB_USER&#039;, &#039;wordpress&#039;);

/** MySQL データベースのパスワード */
define(&#039;DB_PASSWORD&#039;, &#039;******&#039;); # wordpressユーザーのパスワード

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
define(&#039;AUTH_KEY&#039;,         &#039;****************************************************************&#039;);
define(&#039;SECURE_AUTH_KEY&#039;,  &#039;****************************************************************&#039;);
define(&#039;LOGGED_IN_KEY&#039;,    &#039;****************************************************************&#039;);
define(&#039;NONCE_KEY&#039;,        &#039;****************************************************************&#039;);
define(&#039;AUTH_SALT&#039;,        &#039;****************************************************************&#039;);
define(&#039;SECURE_AUTH_SALT&#039;, &#039;****************************************************************&#039;);
define(&#039;LOGGED_IN_SALT&#039;,   &#039;****************************************************************&#039;);
define(&#039;NONCE_SALT&#039;,       &#039;****************************************************************&#039;);
/**#@-*/</pre>

  4. httpd設定  
    wordpress用のconfを作成して設定します。  
    とりあえず、下記の設定でアクセスできるようになります。  
    もし、トップでアクセスしたい場合は/etc/httpd/conf/httpd.confの中のDocumentRootと<directory>を/var/www/wordpressに変更すればOKです。</p> <pre class="brush: bash; gutter: true">echo Alias /wordpress /var/www/wordpress &gt; /etc/httpd/conf.d/wordpress.conf</pre>
    
    設定を反映させましょう。
    
    <pre class="brush: bash; gutter: true">/etc/init.d/httpd reload</pre>

  5. wordpress開設  
    http://設定したアドレス/wordpressにアクセスしてみましょう。  
    画面に表示されている通りにインストールボタンを押せば開設できます！  
    これでひと通り完了です。お疲れ様でした！

 [1]: https://api.wordpress.org/secret-key/1.1/salt/