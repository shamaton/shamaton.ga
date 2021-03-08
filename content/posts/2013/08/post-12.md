---
title: 'ServersMan VPSでWebサーバー構築#1(CentOS:初期設定)'
author: しゃまとん
date: 2013-08-11T14:51:05+00:00
url: /posts/12
categories:
  - Web関連
  - サーバー構築

---
お世話になっております。  
しゃまとんです。

VPSという存在を最近知り、気になっていたので使ってみることにしました。  
さくらなどVPSのサービスは色々とありますが、なるべく費用がかからないようにしたいと思い、ServersManにしました。  
契約したサービスはEntryプランなので、設定するためにCUIを使って構築していきます。  

{{< blogcard url="http://dream.jp/vps/" >}}
※今なら１ヶ月無料キャンペーンやってるそうです！(2013/08/01〜2013/09/30)

自分への備忘録も兼ねてWordPressによるボームページ公開までの手順を書いていこうと思います。
OSはCentOS 64bitにしました。  
※加筆修正は必要があれば行なっていきます。  
<!--more-->

1 .. 一般ユーザーの作成
rootで直接ログインできる状態で今後作業を行うのは危険なので、一般ユーザーを作成します。  
例として、hogeというユーザーを作成します。

```shell
adduser hoge
passwd hoge # このあとパスワード入力する、なにも出力されないが入力されている。
```

2 .. sudoの設定  
sudoは指定した一般ユーザに対して特定のrootコマンドを付与することで代理のrootユーザとして
サーバーを管理させる事ができるようになります。
毎回rootのログインをするのが手間なときに活用することができます。

```shell
rpm -qa | grep sudo
yum install sudo # インストールされてなければ
su -
visudo
```
    
下記の該当箇所にhogeユーザーを追加すれば、sudoを実行できるようになります。  
こちらのサイトが参考になりました。  

CentOS:sudoを設定する
{{< blogcard url="http://d.hatena.ne.jp/a__z/20071011" >}}
    
```text
# Allow root to run any commands anywhere
root         ALL=(ALL)       ALL
hoge        ALL=(ALL)       NOPASSWD:ALL
```
  
3 .. vimをインストールする  
私の環境では、もらった状態でvimはインストール済みでした。
もし入っていなければインストールしましょう。

```shell
yum -y install vim-enhanced
```

4 .. sshのポートを確認する。  
serversmanのデフォルト設定はsshのポート:22を使用していないようですが、22はセキュリティ上使わないように設定しましょう。
ポート番号の確認は/etc/ssh/sshd_configで確認できます。  
rootでログインしてない場合は`sudo vim /etc/ssh/sshd_config`で確認してもよいです。

```text
Port 10022 # ここが22の場合、別のポートに変更する。
```
    
もし変更した場合はsshdの再起動を行いましょう。

```shell
/etc/init.d/sshd restart
```

5 .. ssh接続を鍵認証に変更する。  
接続を行なっている、自分のパソコンで鍵作成を行います。
macならばターミナルが入っていますが、winの場合はTera Termなどを入れるとよいでしょう。

{{< blogcard url="http://sourceforge.jp/projects/ttssh2/" >}}

```shell
ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/hoge/.ssh/id_rsa): # ファイルの出来る場所
Enter passphrase (empty for no passphrase): # パスを入力
Enter same passphrase again: # パスをもう一度入力
```
    
id_rsa.pubというファイルが出来上がるので、サーバー上の.sshディレクトリにauthorized\_keysというファイル名に変更して格納しましょう。
ターム上からコピペでも、ファイル転送でも良いです。.sshディレクトリがなければhogeユーザーのホームディレクトリに作成しましょう。  
ファイルのパーミッション変更も忘れずに。
    
```shell
cd
mkdir .ssh
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```
    
ここまで設定を行うと、鍵認証による接続が行えるはずです。  
続いて、sshの設定も行いましょう。

```text
#LoginGraceTime 2m
PermitRootLogin no # rootでのログイン禁止

…省略…

# To disable tunneled clear text passwords, change to no here!
#PasswordAuthentication no
PermitEmptyPasswords no # 空パスワードの禁止
PasswordAuthentication no # パスワード認証しない
```
    
設定できたらsshdの再起動も忘れずに行いましょう。この際、誤ってログインができなくならないよう、
接続されているタームを1つ以上用意しておきましょう。

6 .. ファイアウォールの構築  
まずは設定ファイルを作成します。

```shell
vim /etc/sysconfig/iptables
```
    
設定するファイルはこちらのサイトの設定ファイルを使わせて頂きました。
sshのポートは人によって違うかもしれないので注意してください。

{{< blogcard url="http://weble.org/2011/05/16/sakura-vps-and-centos" >}}
    
```text
*filter
:INPUT   ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT  ACCEPT [0:0]
:RH-Firewall-1-INPUT - [0:0]

-A INPUT -j RH-Firewall-1-INPUT
-A FORWARD -j RH-Firewall-1-INPUT
-A RH-Firewall-1-INPUT -i lo -j ACCEPT
-A RH-Firewall-1-INPUT -p icmp --icmp-type any -j ACCEPT
-A RH-Firewall-1-INPUT -p 50 -j ACCEPT
-A RH-Firewall-1-INPUT -p 51 -j ACCEPT
-A RH-Firewall-1-INPUT -p udp --dport 5353 -d 224.0.0.251 -j ACCEPT
-A RH-Firewall-1-INPUT -p udp -m udp --dport 631 -j ACCEPT
-A RH-Firewall-1-INPUT -p tcp -m tcp --dport 631 -j ACCEPT
-A RH-Firewall-1-INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# SSH, HTTP, FTP1, FTP2, MySQL
-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 10022 -j ACCEPT
-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 80    -j ACCEPT
-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 20    -j ACCEPT
-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 21    -j ACCEPT
-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 3306  -j ACCEPT

-A RH-Firewall-1-INPUT -j REJECT --reject-with icmp-host-prohibited

COMMIT
```
        
設定できたら、再起動しましょう。

```shell
/etc/init.d/iptables restart
```        
    
7 .. yumのアップデート  
いわゆるOSのアップデートです。最後に忘れずに行いましょう。

```shell
yum update # rootなら
sudo yum update # hoge(一般ユーザーなら)
```
    
参考にさせていただいたサイトでは、他にも確認や設定を行なっていますが、serversmanの環境では不要っぽいものもあるので、
ここまで設定できれば初期設定はとりあえずOKだと思います。
    
次はApache(Webサーバー)のインストールと設定を行なっていきます。
