---
title: '[Docker] CentOS6でwingを使わず最新のgitにする'
author: しゃまとん
type: post
date: 2018-11-04T13:21:38+00:00
url: /posts/567
featured_image: /images/posts/2016/10/small_v-dark.png
is_comment_form_freeze:
  - on
comment_form_freeze_message:
  - ' '
categories:
  - docker
  - Linux

---
お世話になっております。  
しゃまとんです。

この前、Docker Hubに公開していたあるコンテナのリポジトリにissueが飛んできてたので確認してみると、
「このコンテナのgolangのバージョンあげてや〜」みたいなのだったので対応することにしました。

対象のコンテナがCentOS 6で作られていて、コンテナにはgitとgolangがインストールされているものでした。  
じゃあgolangのバージョンだけ上げておくか〜ということで対応して、コンテナをビルドすると途中でエラーに。

原因はgit2.x系をインストールする際に使っていたwingリポジトリが使えなくなっていたことでした。
うーんこれは仕方がないということでgitを別の方法で入れることに。  
(実は他のところで生きているかもしれません)

手法としては至って単純で、とりあえず標準のgitをyum installした後に、
最新のgitをcloneしてきてビルドしておきかえるというものです。

バージョンはgithubのstableの最新をビルドするようにしました。  
Dockerfileはこちらになります。

```dockerfile
From centos:6

MAINTAINER shamaton

# define
ARG go_ver="1.10.4"

# root work
RUN yum update -y
RUN yum install -y sudo
RUN useradd -m -d /home/docker -s /bin/bash docker && echo "docker:docker" | chpasswd
RUN echo "docker ALL=(ALL) NOPASSWD:ALL" &gt;&gt; /etc/sudoers

# install latest stable git
RUN yum -y install \
git \
gcc \
curl-devel \
expat-devel \
gettext-devel \
openssl-devel \
zlib-devel \
perl-ExtUtils-MakeMaker && \
git clone https://github.com/git/git.git && \
cd git/ && \
git checkout `git tag | sort -V | grep -v "\-rc" | tail -1` && \
yum -y remove git && \
make prefix=/usr all && \
make prefix=/usr install && \
cd / && \
rm -rf /git

# install golang
RUN curl -O https://storage.googleapis.com/golang/go${go_ver}.linux-amd64.tar.gz
RUN tar -C /usr/local -xzf go${go_ver}.linux-amd64.tar.gz && \
rm go${go_ver}.linux-amd64.tar.gz

# change user docker
WORKDIR /home/docker
USER docker

# user work
ENV PATH $PATH:/usr/local/go/bin
RUN mkdir -p ${HOME}/.packages
ENV GOPATH /home/docker/.packages
```

大事なのは、#install latest stable gitのところですね。

git及びビルド用パッケージをインストールして、  
↓  
gitのlatest stableをclone  
↓  
とりあえずインストールしたgitをアンイストール  
↓  
make / make install

として最新版を使えるようにしました。  
ちなみに以前はこんなふうでした。

```dockerfile
# update git
RUN cd /etc/yum.repos.d/ && \
curl -O http://wing-repo.net/wing/6/EL6.wing.repo && \
yum -y --enablerepo=wing install git
```

行数は増えました(見やすさのため)が、コマンド的には一行でまとめておきました。  
ただ、これはコンテナをビルドしたタイミングでバージョンに差異がでるので、気にする必要がある場合には、cloneするバージョンを指定するほうがいいかもですね。

以上です。