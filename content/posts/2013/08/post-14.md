---
title: 'ServersMan VPSでWebサーバー構築#3(Apache)'
author: しゃまとん
date: 2013-08-14T15:44:47+00:00
url: /posts/14
categories:
  - Web関連
  - サーバー構築

---
ドメインの取得もできたので、次にWebサーバーを構築していきます。

Webサーバーでよく知られているApache(httpd)を導入します。  
作業は初期設定同様、基本的にスーパーユーザ(su)で行います。

<!--more-->

1 .. インストール  
まずはインストールされているか確認。ServerMansは最初から入ってたかも。。。
なのでまずはインストールされているか確認。

```shell
rpm -qa | grep httpd
```

インストールされていなければ、下記コマンドを実行する

```shell
yum -y install httpd
```

合わせてphpもインストールしておきましょう。

```shell
yum -y install php php-mbstring
```

2 .. httpd.confの設定  
インストールができたら、設定ファイルを編集します。

```shell
vim /etc/httpd/conf/httpd.conf
```

ファイル内、該当箇所を編集します。

```text
ServerTokens Prod　# エラーページ等でOS名を表示しない

ServerName shamaton.orz.hm:80　# 取得したドメインにする

<Directory "/var/www/html">

...(省略)...

# http://httpd.apache.org/docs-2.0/mod/core.html#options
# for more information.
#
    Options Includes ExecCGI FollowSymLinks　# CGI,SSIの許可

#
# AllowOverride controls what directives may be placed in .htaccess files.
# It can be "All", "None", or any combination of the keywords:
#   Options FileInfo AuthConfig Limit
#
    AllowOverride All　# .htaccessを許可する

...(省略)...

ServerSignature Off　＃ エラーページでサーバー情報を表示しない

#AddDefaultCharset UTF-8　# 文字化けするのでコメントアウト

AddHandler cgi-script .cgi .pl　＃ CGIスクリプトに.plを追加する

<Directory "/var/www/icons">
    Options MultiViews　＃ iconsのファイル一覧を表示しない
    AllowOverride None
    Order allow,deny
    Allow from all
</Directory>
```

3 .. 不要ページ削除  
これらのページは使わないので削除します。
```shell
rm -f /etc/httpd/conf.d/welcome.conf
rm -f /var/www/error/noindex.html
```

4 .. ドキュメントルートの所有者を変更  
ホームページを置く場所の所有者を自分にします。
```shell
chown hoge. /var/www/html/   # 所有者をユーザー:hogeにする
ls-al # 所有者が変更されているか確認
```

5 .. httpd起動  
Webサーバーを起動しましょう。すでにしてるならrestartでOK。

```shell
/etc/init.d/httpd start  # apache(httpd)起動
/etc/init.d/httpd restart  # 再起動
```

テストページを作成して、[こちら][1]でアクセスできるかチェックしましょう。  
WebsiteTestのラジオボタンにチェックを入れ、サーバーのアドレスを入力しましょう。

```shell
echo test >> /var/www/html/index.html
```
    
結果のStatusにOKと表示されれば、外部からのアクセスが可能です。
    
* 後始末  
最後にテストページを削除して完了です。  
`rm -f /var/www/html/index.html`

* その他のテスト…  
試してみたい、もしくは必要ならば行いましょう。  
下記サイトが参考になります。  
  {{< blogcard url="http://centossrv.com/apache.shtml" >}}

 [1]: http://www.websitepulse.com/help/tools.php