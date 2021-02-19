---
title: '[GCP] Compute Engine(CentOS7)でdocker/gitを使えるようにする'
author: しゃまとん
type: post
date: 2017-12-21T15:33:45+00:00
url: /archives/432
featured_image: /wp-content/uploads/2017/08/modern-CentOS-logo.png
categories:
  - GCP
  - Linux

---
お世話になっております。  
しゃまとんです。

GCPが無料で試せるということで、Compute Engineを使ってみることにしました。  
個人的にdockerとgitが使えればいいかなということでやっていたのですが、普段割りと使っているCentOS6でdockerがイケてない感じだったので、CentOS7を使うことに。

忘れないようにセットアップのメモです。  
基本的にrootで作業しています。sudoで実行する場合はつけて実行してください

まずはタイムゾーンを日本にしておきます。

<pre class="lang:sh decode:true">timedatectl set-timezone Asia/Tokyo</pre>

gitをインストールします。デフォルトの状態でgitを入れると1.x系が入ってしまうので、repoを追加して、stableの最新バージョンを入れるようにします。

<pre class="lang:sh decode:true"># iusレポジトリを取得して展開
wget https://dl.iuscommunity.org/pub/ius/stable/CentOS/7/x86_64/ius-release-1.0-15.ius.centos7.noarch.rpm
rpm -Uvh ius-release-1.0-15.ius.centos7.noarch.rpm

# gitをインストール
yum install git2u --enablerepo=ius

# 確認
git version</pre>

dockerをインストールしていきます。こちらもrepoを追加してcommunity editionの最新版をいれるようにします。

<pre class="lang:sh decode:true"># yum-config-managerを使えるようにする
yum install yum-utils

# repo追加
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# 有効にする
sudo yum-config-manager --disable docker-ce-edge</pre>

リストを一応確認しておきます。

<pre class="lang:sh decode:true">yum list docker-ce.x86_64 --showduplicates

# 実行結果
Available Packages
docker-ce.x86_64 17.03.0.ce-1.el7.centos docker-ce-stable
docker-ce.x86_64 17.03.1.ce-1.el7.centos docker-ce-stable
docker-ce.x86_64 17.03.2.ce-1.el7.centos docker-ce-stable
docker-ce.x86_64 17.06.0.ce-1.el7.centos docker-ce-stable</pre>

インストールして、root以外の指定ユーザーで実行できるようにしておきます。

<pre class="lang:sh decode:true"># インストール
yum install docker-ce

# 起動と確認
systemctl status docker
systemctl start docker
systemctl status docker

# 確認
docker version

# 起動時に有効にする
systemctl enable docker.service

# 一般ユーザーでdockerを使えるようにする（/etc/groupを確認しておくよい）
gpasswd -a {ユーザー名} docker</pre>

gpasswdを実行した後、一旦ログインしなさないとdockerコマンド実行時にエラーがでることがあります。

<pre class="lang:default decode:true " title="error">Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.30/version: dial unix /var/run/docker.sock: connect: permission denied</pre>

最後にdocker-composeを使えるようにします。  
最新のバージョンは[こちら][1]で確認してください。

<pre class="lang:sh decode:true "># インストール
curl -L https://github.com/docker/compose/releases/download/1.15.0/docker-compose-`uname -s`-`uname -m` &gt; /usr/local/bin/docker-compose

# 実行権限を追加
chmod +x /usr/local/bin/docker-compose

# 確認
docker-compose version</pre>

dockerはdocker run hello-worldして動作確認してもよいかもですね。  
以上です。

■ 参考  
[CentOS7にdockerをインストール][2]  
[CentOS7にDocker Community Edition, Docker Composeをインストールする][3]  
[CentOS7でsudoなしでdockerを利用するちょっとした工夫][4]  
[CentOSのGitをアップデート(2系に)する][5]

 [1]: https://github.com/docker/compose/releases
 [2]: http://qiita.com/maimai-swap/items/00590b96888330aa54f1
 [3]: http://qiita.com/shinespark/items/367959950d9e1fd454ee
 [4]: http://qiita.com/tubone/items/9c1b3d807197b7162fd9
 [5]: http://www.cyamax.com/entry/2017/05/02/060000