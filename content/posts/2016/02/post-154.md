---
title: GithubクローンのGitBucketをCentOS6に入れる
author: しゃまとん
date: 2016-02-20T03:41:24+00:00
url: /posts/154
featured_image: /images/posts/2016/02/gitbucket.png
categories:
  - Web関連
  - サーバー構築

---
お世話になっております。  
しゃまとんです。

ひょんなことから、プライベートなgitホスティングサービスを触ってみる機会があったのですが、
以前に「gitlabがあるけどインストールがややこしい」という噂を聞いていて、ちょっとめんどそう&#8230;かと思いきや、
割りと簡単にセットアップできたので、それのメモです。

今回はgithubクローンなgitbucketを触ってみました。  
gitlabに関しては公式サイトの手順でそのままいけて、yumまで使えるようになっていてサクッとできすぎる感じになってました！

他の方々が構築手順を載せてくださっていますが、自分は  
* CentOS 6  
* GITBUCKET_HOMEを/home/gitbucket/.gitbucketにする  
* 80ポートで確認できる  

という条件で構築されるようにしています。

以下、手順です。  
CentOSは[この辺](https://www.centos.org/download/)から取得してインストールしてください。  
ユーザーはgitbucketを作成し、sudoの実行権限を与えておくとよいです。


gitbucketはjava(tomcat)で動作するので、openjdkとhttpd(apache)をyumでインストールします。wgetはファイル取得に使います。

```shell
sudo yum install wget httpd java-1.8.0-openjdk-devel java-1.8.0-openjdk
```

続いて、tomcatを取得します。サイトからversion8の最新版を取得してください。  
取得したら、解凍しリネームしておきます。

{{< blogcard url="http://tomcat.apache.org/" >}}

```shell
wget http://ftp.kddilabs.jp/infosystems/apache/tomcat/tomcat-8/v8.0.32/bin/apache-tomcat-8.0.32.tar.gz
tar zvxf apache-tomcat-8.0.32.tar.gz
mv apache-tomcat-8.0.32 tomcat8
```

次にgitbucketを取得します。こちらもサイトで最新版を取得してください。  
取得したら先ほどリネームしたtomcatの下に移動させます。

{{< blogcard url="https://github.com/gitbucket/gitbucket/releases" >}}

```shell
wget https://github.com/gitbucket/gitbucket/releases/download/3.11/gitbucket.war
mv gitbucket.war ./tomcat8/webapps/gitbucket.war
```

これで必要なファイルが揃ったので、80ポートで受け付けるようにhttpdとtomcatの設定ファイルを少しだけ修正します。

```shell
vi ./tomcat8/conf/server.xml
```

```text
<!-- 元設定
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
    -->

    <!-- こちらに変更 -->
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
```

```shell
sudo vi /etc/httpd/conf.d/gitbucket.conf
```

```text
<VirtualHost *:80>
  DocumentRoot /home/gitbucket/tomcat8/webapps/gitbucket
  #ServerName yourhost.domain
  ErrorLog logs/gitbucket-error_log
  CustomLog logs/gitbucket-access_log common

    <Proxy *>
      Order deny,allow
      Allow from all
    </Proxy>

  ProxyPass /assets !

  ProxyPass /gitbucket ajp://localhost:8009/gitbucket
  ProxyPassReverse /gitbucket ajp://localhost:8009/gitbucket
  ProxyPreserveHost on

</VirtualHost>
```

※ServerNameは今回設定していません。

環境によってはiptablesが効いていると思うので、止めておくかポートを設定しておく必要があります。  
今回の場合は止めておきました。

```shell
sudo service iptables stop
```

これで、tomcatを起動してhttpdを再起動したら、http://IPアドレスなど/gitbucket/でアクセスすることができます。
（最初は表示されるまで数秒かかるっぽい）

```shell
~/tomcat8/bin/startup.sh
sudo service httpd restart
```

githubと同じような間隔で使えるっぽいです！  
user : root, password : rootでログインできます。

{{< figure src="/images/posts/2016/02/gitbuket_home.png" >}}

環境変数はこんな感じです。

{{< figure src="/images/posts/2016/02/gitbuket_home2.png" >}}

以上です。

■ 参考  
{{< blogcard url="http://blacknd.com/linux-server/centos7-gitbucket-jenkins-auto-deploy/" >}}
{{< blogcard url="http://qiita.com/YN0314/items/d205dfed2e968bf8f408" >}}
