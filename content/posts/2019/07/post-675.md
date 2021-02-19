---
title: '[Docker] 32bit環境のコンテナを作る'
author: しゃまとん
type: post
date: 2019-07-14T10:23:09+00:00
url: /archives/675
featured_image: /wp-content/uploads/2016/10/small_v-dark.png
is_comment_form_freeze:
  - on
comment_form_freeze_message:
  - ' '
categories:
  - docker

---
 お世話になっております。  
しゃまとんです。  
  
先日、msgpackにissueが立っていたので確認したところ windows/386 on Windows 10でbuildエラーになってしまうということだったので修正しておくことに。 エラーになっている箇所を確認したところ、どうやらWindowsのOSは関係なく32bit環境だとエラーになってるぽいなーと思い、じゃあ32bit環境で試すかー。ってそんな環境はねぇ！って一瞬思ったんですが  
  
32bitの環境だったら、Linuxでもいいから用意できたらいいかも  
Dockerでいけないだろうかと思ったら、そんなコンテナあるんですねー。  
  
<https://hub.docker.com/r/i386/centos/>  
  
ということで32bitなコンテナのテスト環境をつくっていきます。  
（CentOS7でも良さそうだったのに、今回は6でやりました）  
  
上記のコンテナをベースにDockerfileを作りました。以前に似たようなことをしていたのですが、同様にgolang/gitを入れておきます。 ベースそのままだとyumがうまく動かないのでrepoファイルに対処を入れる必要がありました。 

<pre class="wp-block-preformatted">FROM i386/centos:6
 
LABEL maintainer="shamaton"
 
# define
ARG go_ver="1.12.7"
 
# repo setting
RUN sed -i 's/$basearch/i386/g' /etc/yum.repos.d/CentOS-*.repo
RUN yum clean all

# root work
RUN yum install -y sudo
RUN useradd -m -d /home/docker -s /bin/bash docker && echo "docker:docker" | chpasswd
RUN echo "docker ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
 
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
yum -y remove gcc \
curl-devel \
expat-devel \
gettext-devel \
openssl-devel \
zlib-devel \
perl-ExtUtils-MakeMaker && \
cd / && \
rm -rf /git
 
# install golang
RUN curl -O https://storage.googleapis.com/golang/go${go_ver}.linux-386.tar.gz
RUN tar -C /usr/local -xzf go${go_ver}.linux-386.tar.gz && \
rm go${go_ver}.linux-386.tar.gz
 
# change user docker
WORKDIR /home/docker
USER docker
 
# user work
ENV PATH $PATH:/usr/local/go/bin
RUN mkdir -p ${HOME}/go
ENV GOPATH /home/docker/go</pre> 次にbuildして立ち上げて見ましょう。 

<pre class="wp-block-code"><code>docker build ./ -t centos6-i386-go-git
docker run -it --rm --name test123 centos6-i386-go-git bash</code></pre> コンテナが起動したら、32bitでgolangとgitが使えるか確認しておきます 

<pre class="wp-block-code"><code>$ getconf LONG_BIT
32

$ git version
git version 2.22.0

$ go version
go version go1.12.7 linux/386</code></pre> ついでにissueになっている挙動も確認しておきます 

<pre class="wp-block-code"><code>$ go get -d -v github.com/shamaton/msgpack
 github.com/shamaton/msgpack (download)

$ cd go/src/github.com/shamaton/msgpack

$ go test -v .
github.com/shamaton/msgpack/internal/encoding
internal/encoding/byte.go:22:14: constant 4294967295 overflows int
internal/encoding/byte.go:36:14: constant 4294967295 overflows int
internal/encoding/encoding.go:110:15: constant 4294967295 overflows int
internal/encoding/encoding.go:158:15: constant 4294967295 overflows int
internal/encoding/encoding.go:195:15: constant 4294967295 overflows int
internal/encoding/map.go:267:14: constant 4294967295 overflows int
internal/encoding/slice.go:107:14: constant 4294967295 overflows int
internal/encoding/struct.go:94:14: constant 4294967295 overflows int
internal/encoding/struct.go:141:14: constant 4294967295 overflows int
internal/encoding/struct.go:197:16: constant 4294967295 overflows int
internal/encoding/struct.go:197:16: too many errors
FAIL    github.com/shamaton/msgpack [build failed] </code></pre> エラーでた！

  
ってことで、32bit環境になっているようです。（不具合を晒していく）  
  
さぁ、修正しよう。（した）  
以上です。  
  
あとから気づいたけど、こっちのほうが良かったかな&#8230; <https://hub.docker.com/r/i386/alpine/>