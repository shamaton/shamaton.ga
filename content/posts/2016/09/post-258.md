---
title: sshでローカル環境だけにパスワード認証を有効にする
author: しゃまとん
date: 2016-09-17T02:25:33+00:00
url: /posts/258
featured_image: /images/posts/2016/08/ssh_eye_catch.png
is_comment_form_freeze:
  - on
categories:
  - Linux
  - Mac
  - Web関連

---
お世話になっております。  
しゃまとんです。

環境構築のテストをしていて、ssh前の環境から鍵情報をもってきてgit操作したいなーと思い調査していた際のメモです。  
やりたかったのは

```text
自分PC　→ Server(user:shamaton) → Server(user:hoge)
```

の流れでログインし、最終的にhogeになった状態でgit操作したい、でした。
はじめは`sudo su hoge`とかでなんとか出来ないかなーと思っていたのですが、どうやら無理みたいでした。

{{< blogcard url="https://ngyuki.hatenablog.com/entry/20120508/p1" >}}

ということでssh localhost hogeという手段をとり、鍵をそのまま使えるようにしようと思いました。  
ただ、hogeに関しては外部からログインできないようにしたくどうしようかというところ、調べてみるとsshのMatchキーワードを設定すると実現できました。

以下、設定方法です。

一応、バックアップしておきます

```shell
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.org
sudo vi /etc/ssh/sshd_config
```

デフォルトのパスワード認証は無効にしておきます

```text
PasswordAuthentication no
```

そしてローカル環境だけパスワード認証できるようにします

```text
Match Address 127.0.0.1
PasswordAuthentication yes
```
<pre class="brush: text; gutter: true"></pre>

Matchのキーワードには User, Group, Host, Addressが設定できるようです。

```text
Match User hoge, fuga Host localhost
PasswordAuthentication yes
```

とすると、hogeとfugaはローカルからのパスワード認証を許可することができます。

設定後は再起動しておきましょう。

```shell
sudo /etc/rc.d/init.d/sshd restart
```

この際に編集していたシェルは万が一のために残しておいて、別のシェルから確認作業しましょう。

簡単に設定できました。細かいユーザー分けもできそうですね。  
以上です。