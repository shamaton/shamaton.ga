---
title: Macでcompletionするアレコレ
author: しゃまとん
date: 2017-04-05T15:10:31+00:00
url: /posts/333
featured_image: /images/posts/2016/11/completion.gif
categories:
  - Linux
  - Mac
  - メモ

---
お世話になっております。  
しゃまとんです。

皆さんcompletion使ってますでしょうか。  
gitやdockerを使うときにcompletionを設定しているとコマンドを補完してくれて作業が捗りますよね。

大体gitやdockerをインストールした時に設定したりするのですが、  
いつもcompletion毎に調べたりしてめんどくさいなぁと思ったので個人的な趣味でまとめました。  
bashかzshで分けておきました。

* git, dockerが入ってる前提です  
* 各OSでzshのディレクトリ構成が違うかもしれません

■ Bash

{{< gist shamaton 278938bde469ac59a5a7adff017d5592 >}}


最後に現在のシェルに反映します。

<pre class="lang:sh decode:true">source ~/.bashrc</pre>

■ Zsh

{{< gist shamaton 277d73debd08dee9bbcb271f3228d926 >}}

次にzshrcにfpathを追記します。  
autoloadよりも前に記載しておきます。

```shell
#[.zshrc]
fpath=(${HOME}/.zsh_completions $fpath)

autoload -Uz compinit -i
compinit -u
```

最後に反映させます

```shell
source ~/.zshrc
```

試しにやってみると...

{{< figure src="/images/posts/2016/11/completion.gif" >}}

このように補完してくれるようになりました。  
どっちも便利なので是非活用しましょう！  
以上です。

■ appendix  
かんたんに検証したい方向けにCentOS on Docker用コマンド

```shell
# コンテナ起動
docker run --name=tmp -it centos bash

# コンテナにgit/docker周りをインストール
echo "include_only=.jp" >> /etc/yum/pluginconf.d/fastestmirror.conf && \
yum -y update && \
yum -y install git docker zsh && \
curl -L https://github.com/docker/machine/releases/download/v0.8.2/docker-machine-`uname -s`-`uname -m` >/usr/local/bin/docker-machine && \
chmod +x /usr/local/bin/docker-machine && \
curl -L "https://github.com/docker/compose/releases/download/1.8.1/docker-compose-$(uname -s)-$(uname -m)" > /usr/local/bin/docker-compose && \
chmod +x /usr/local/bin/docker-compose

# いろいろ試す...

# 後始末
docker rm tmp
docker rmi centos:latest
```
