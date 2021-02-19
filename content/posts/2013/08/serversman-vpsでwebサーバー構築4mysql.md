---
title: 'ServersMan VPSでWebサーバー構築#4(MySQL)'
author: しゃまとん
type: post
date: 2013-08-15T15:19:01+00:00
url: /archives/15
categories:
  - Web関連
  - サーバー構築

---
Webサーバーができたところで、次にWordPressをいざ導入…の前にWordPressにはMySQLというデータベースが必要なので、MySQLを使えるようにします。

ここでも基本的にスーパーユーザ(su)で作業を行なっていきます。

それでは手順です。

<!--more-->

  1. MySQLインストール  
    シンプルプランではMySQLがインストールされていないので、インストールします。</p> <pre class="brush: bash; gutter: true">yum -y install mysql-server</pre>

  2. 設定ファイル編集 <pre class="brush: bash; gutter: true">vim /etc/my.cnf</pre>
    
    行末に追加しましょう。
    
    <pre class="brush: bash; gutter: true">character-set-server = utf8 #文字コードをUTF-8にする</pre>

  3. MySQL起動 <pre class="brush: bash; gutter: true">/etc/init.d/mysqld start

MySQL データベースを初期化中:  Installing MySQL system tables...
OK
Filling help tables...
OK

...(省略)...

mysqld を起動中:                                           [  OK  ]</pre>
    
    サーバー再起動時に自動で立ち上がるようにしましょう。
    
    <pre class="brush: bash; gutter: true">chkconfig --level 3 mysqld on</pre>

  4. MySQL初期設定 <pre class="brush: bash; gutter: true">mysql_secure_installation　# MySQL初期設定</pre>
    
    下記の通りに設定していきましょう。
    
    <pre class="brush: bash; gutter: true">NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MySQL
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MySQL to secure it, we&#039;ll need the current
password for the root user.  If you&#039;ve just installed MySQL, and
you haven&#039;t set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 　# 何も入力せずENTER
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MySQL
root user without the proper authorisation.

Set root password? [Y/n] 　Y # yes
New password: 　# rootのパスワードを設定
Re-enter new password: 　# 再度入力
Password updated successfully!
Reloading privilege tables..
 ... Success!

By default, a MySQL installation has an anonymous user, allowing anyone
to log into MySQL without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] 　# 何も入力せずENTER
 ... Success!

Normally, root should only be allowed to connect from &#039;localhost&#039;.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] 　# 何も入力せずENTER
 ... Success!

By default, MySQL comes with a database named &#039;test&#039; that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] 　# 何も入力せずENTER
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] 　# 何も入力せずENTER
 ... Success!

Cleaning up...

All done!  If you&#039;ve completed all of the above steps, your MySQL
installation should now be secure.

Thanks for using MySQL!</pre>

これでインストールと設定が完了です。

次はいよいよWordPressです！

&nbsp;