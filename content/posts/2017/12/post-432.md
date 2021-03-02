---
title: '[GCP] Compute Engine(CentOS7)でdocker/gitを使えるようにする'
author: しゃまとん
date: 2017-12-21T15:33:45+00:00
url: /posts/432
featured_image: /images/posts/2017/08/modern-CentOS-logo.png
categories:
  - GCP
  - Linux

---
お世話になっております。  
しゃまとんです。

GCPが無料で試せるということで、Compute Engineを使ってみることにしました。  
個人的にdockerとgitが使えればいいかなということでやっていたのですが、
普段割りと使っているCentOS6でdockerがイケてない感じだったので、CentOS7を使うことに。

{{< blogcard url="https://cloud.google.com/compute?hl=ja" >}}

忘れないようにセットアップのメモです。  
基本的にrootで作業しています。`sudo`で実行する場合はつけて実行してください

まずはタイムゾーンを日本にしておきます。

```shell
timedatectl set-timezone Asia/Tokyo
```

gitをインストールします。デフォルトの状態でgitを入れると1.x系が入ってしまうので、
repoを追加して、stableの最新バージョンを入れるようにします。

```shell
# iusレポジトリを取得して展開
wget https://dl.iuscommunity.org/pub/ius/stable/CentOS/7/x86_64/ius-release-1.0-15.ius.centos7.noarch.rpm
rpm -Uvh ius-release-1.0-15.ius.centos7.noarch.rpm

# gitをインストール
yum install git2u --enablerepo=ius

# 確認
git version
```

dockerをインストールしていきます。こちらもrepoを追加してcommunity editionの最新版をいれるようにします。

```shell
# yum-config-managerを使えるようにする
yum install yum-utils

# repo追加
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# 有効にする
sudo yum-config-manager --disable docker-ce-edge
```

リストを一応確認しておきます。

```shell
yum list docker-ce.x86_64 --showduplicates

# 実行結果
Available Packages
docker-ce.x86_64 17.03.0.ce-1.el7.centos docker-ce-stable
docker-ce.x86_64 17.03.1.ce-1.el7.centos docker-ce-stable
docker-ce.x86_64 17.03.2.ce-1.el7.centos docker-ce-stable
docker-ce.x86_64 17.06.0.ce-1.el7.centos docker-ce-stable
```

インストールして、root以外の指定ユーザーで実行できるようにしておきます。

```shell
# インストール
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
gpasswd -a {ユーザー名} docker
```

`gpasswd`を実行した後、一旦ログインし直さないとdockerコマンド実行時にエラーがでることがあります。

```text
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http:///var/run/docker.sock/v1.30/version: dial unix /var/run/docker.sock: connect: permission denied
```

最後にdocker-composeを使えるようにします。  
最新のバージョンはリンク先で確認してください。

{{< blogcard url="https://github.com/docker/compose/releases" >}}

```shell
# インストール
curl -L https://github.com/docker/compose/releases/download/1.15.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# 実行権限を追加
chmod +x /usr/local/bin/docker-compose

# 確認
docker-compose version
```

dockerは`docker run hello-world`して動作確認してもよいかもですね。  
以上です。

■ 参考
{{< blogcard url="http://qiita.com/maimai-swap/items/00590b96888330aa54f1" >}}
{{< blogcard url="http://qiita.com/shinespark/items/367959950d9e1fd454ee" >}}
{{< blogcard url="http://qiita.com/tubone/items/9c1b3d807197b7162fd9" >}}
{{< blogcard url="http://www.cyamax.com/entry/2017/05/02/060000" >}}
