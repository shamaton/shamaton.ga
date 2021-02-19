---
title: GithubクローンのGitBucketをCentOS6に入れる
author: しゃまとん
type: post
date: 2016-02-20T03:41:24+00:00
url: /archives/154
featured_image: /wp-content/uploads/2016/02/gitbucket.png
categories:
  - Web関連
  - サーバー構築

---
お世話になっております。  
しゃまとんです。

ひょんなことから、プライベートなgitホスティングサービスを触ってみる機会があったのですが、以前に「gitlabがあるけどインストールがややこしい」という噂を聞いていて、ちょっとめんどそう&#8230;かと思いきや、割りと簡単にセットアップできたので、それのメモです。

今回はgithubクローンなgitbucketを触ってみました。  
gitlabに関しては公式サイトの手順でそのままいけて、yumまで使えるようになっていてサクッとできすぎる感じになってました！

他の方々が構築手順を載せてくださっていますが、自分は  
・CentOS 6  
・GITBUCKET_HOMEを/home/gitbucket/.gitbucketにする  
・80ポートで確認できる  
という条件で構築されるようにしています。

以下、手順です。  
CentOSは<a href="http://isoredirect.centos.org/centos/6/isos/x86_64/" target="_blank">この辺</a>から取得してインストールしてください。  
ユーザーはgitbucketを作成し、sudoの実行権限を与えておくとよいです。

gitbucketはjava(tomcat)で動作するので、openjdkとhttpd(apache)をyumでインストールします。wgetはファイル取得に使います。

<pre class="brush: bash; gutter: true">sudo yum install wget httpd java-1.8.0-openjdk-devel java-1.8.0-openjdk</pre>

続いて、tomcatを取得します。<a href="http://tomcat.apache.org/" target="_blank">サイト</a>からversion8の最新版を取得してください。  
取得したら、解凍しリネームしておきます。

<pre class="brush: bash; gutter: true">wget http://ftp.kddilabs.jp/infosystems/apache/tomcat/tomcat-8/v8.0.32/bin/apache-tomcat-8.0.32.tar.gz
tar zvxf apache-tomcat-8.0.32.tar.gz
mv apache-tomcat-8.0.32 tomcat8</pre>

次にgitbucketを取得します。こちらも<a href="https://github.com/gitbucket/gitbucket/releases" target="_blank">サイト</a>で最新版を取得してください。  
取得したら先ほどリネームしたtomcatの下に移動させます。

<pre class="brush: bash; gutter: true">wget https://github.com/gitbucket/gitbucket/releases/download/3.11/gitbucket.war
mv gitbucket.war ./tomcat8/webapps/gitbucket.war</pre>

これで必要なファイルが揃ったので、80ポートで受け付けるようにhttpdとtomcatの設定ファイルを少しだけ修正します。

<pre class="brush: bash; gutter: true">vi ./tomcat8/conf/server.xml</pre>

<pre class="brush: text; gutter: true">&lt;!-- 元設定
    &lt;Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" /&gt;
    --&gt;

    &lt;!-- こちらに変更 --&gt;
    &lt;Connector port="8009" protocol="AJP/1.3" redirectPort="8443" /&gt;</pre>

<pre class="brush: bash; gutter: true">sudo vi /etc/httpd/conf.d/gitbucket.conf</pre>

<pre class="brush: text; gutter: true">&lt;VirtualHost *:80&gt;
  DocumentRoot /home/gitbucket/tomcat8/webapps/gitbucket
  #ServerName yourhost.domain
  ErrorLog logs/gitbucket-error_log
  CustomLog logs/gitbucket-access_log common

    &lt;Proxy *&gt;
      Order deny,allow
      Allow from all
    &lt;/Proxy&gt;

  ProxyPass /assets !

  ProxyPass /gitbucket ajp://localhost:8009/gitbucket
  ProxyPassReverse /gitbucket ajp://localhost:8009/gitbucket
  ProxyPreserveHost on

&lt;/VirtualHost&gt;</pre>

※ServerNameは今回設定していません。

環境によってはiptablesが効いていると思うので、止めておくかポートを設定しておく必要があります。  
今回の場合は止めておきました。

<pre class="brush: bash; gutter: true">sudo service iptables stop</pre>

これで、tomcatを起動してhttpdを再起動したら、http://IPアドレスなど/gitbucket/でアクセスすることができます。（最初は表示されるまで数秒かかるっぽい）

<pre class="brush: bash; gutter: true">~/tomcat8/bin/startup.sh
sudo service httpd restart</pre>

githubと同じような間隔で使えるっぽいです！  
user : root, password : rootでログインできます。

[<img src="http://shamaton.orz.hm/blog/wp-content/uploads/2016/02/gitbuket_home.png" alt="gitbuket_home" width="970" height="508" class="alignleft size-full wp-image-157" />][1]

環境変数はこんな感じです。

[<img src="http://shamaton.orz.hm/blog/wp-content/uploads/2016/02/gitbuket_home2.png" alt="gitbuket_home2" width="1012" height="444" class="alignleft size-full wp-image-158" />][2]

以上です。

■ 参考  
<a href="http://blacknd.com/linux-server/centos7-gitbucket-jenkins-auto-deploy/" target="_blank">CentOS 7でGitBucketを動かしてJenkinsで自動デプロイ</a>  
<a href="http://qiita.com/YN0314/items/d205dfed2e968bf8f408" target="_blank">CentOSにGitBucketを構築してみた。</a>

 [1]: http://shamaton.orz.hm/blog/wp-content/uploads/2016/02/gitbuket_home.png
 [2]: http://shamaton.orz.hm/blog/wp-content/uploads/2016/02/gitbuket_home2.png