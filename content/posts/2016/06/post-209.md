---
title: CentOS7に.NET Coreを入れてhello worldしてみる
author: しゃまとん
type: post
date: 2016-06-24T15:07:44+00:00
url: /archives/209
featured_image: /wp-content/uploads/2016/06/microsoft.png
categories:
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

UnityでC#を触り始めて、ふと「LinuxでもC#で開発できるのだろうか？」と思いました。  
たしかC#ってMicrosoft的な何かじゃなかったっけ・・・と思いつつググると。

.NET Coreなるものが存在するそうで、Windows / Mac / Linuxな環境でC#の開発や実行が可能だそうです。  
（monoとかいろいろあるんだけれど、まぁそれはあれで）

[.NET Core &#8211; CentOS][1]

紹介ページをみると、なんとも簡単に環境を用意できそうだったのでhello worldするまでやってみました。  
サイトではCentOS7.1向けの手順となっていました（2016/06時点）。ちなみにPreview版でした。

とりあえず手元にあるCentOS6.7で試みたのですが、途中で何度もつまづきhello worldできなかったので、何もなければCentOS7でやるのがいいと思います。

ちなみにCentOS7でもちょっと余分な手順が必要でした。（preview版）だからでしょうか・・・以下、手順となります。

まず最初に記載されているコマンドを実行します。インストールしたての状態で実行すると何かしらのエラーが出ます。自分の時はこんな感じでした。

<pre class="brush: bash; gutter: true">[user@centos7 ~]$ curl -sSL https://raw.githubusercontent.com/dotnet/cli/rel/1.0.0-preview1/scripts/obtain/dotnet-install.sh | bash /dev/stdin --version 1.0.0-preview1-002702 --install-dir ~/dotnet
dotnet_install: Error: Unable to locate libunwind. Install libunwind to continue
dotnet_install: Error: Unable to locate libicu. Install libicu to continue
dotnet_install: Error: Unable to locate gettext. Install gettext to continue</pre>

何やら足らないので、yum install で追加していきます。

<pre class="brush: bash; gutter: true">sudo yum install libunwind libicu gettext</pre>

これで再度実行してみます。

<pre class="brush: bash; gutter: true">[user@centos7 ~]$ curl -sSL https://raw.githubusercontent.com/dotnet/cli/rel/1.0.0-preview1/scripts/obtain/dotnet-install.sh | bash /dev/stdin --version 1.0.0-preview1-002702 --install-dir ~/dotnet
dotnet-install: Downloading https://dotnetcli.blob.core.windows.net/dotnet/beta/Binaries/1.0.0-preview1-002702/dotnet-dev-centos-x64.1.0.0-preview1-002702.tar.gz
dotnet-install: Extracting zip
dotnet-install: Adding to current process PATH: /home/game/dotnet. Note: This change will be visible only when sourcing script.
dotnet-install: Installation finished successfuly.</pre>

インストールに成功したようです。  
続いて手順通りにリンクを貼ります。

<pre class="brush: bash; gutter: true">sudo ln -s ~/dotnet/dotnet /usr/local/bin</pre>

手順通りに空のディレクトリを作成し、cdしたのち新規作成っぽいコマンドを実行します。

<pre class="brush: bash; gutter: true">[user@centos7 ~]$ mkdir hwapp
[user@centos7 ~]$ cd hwapp/
[user@centos7 hwapp]$ dotnet new
Created new C# project in /home/user/hwapp.

[user@centos7 hwapp]$ ls
Program.cs project.json</pre>

さらに手順通りに実行してみます。  
restoreを実行すると、必要なモジュールをInstallするようですね。nugetして必要なものを解決してくれるようです。

<pre class="brush: bash; gutter: true">[user@centos7 hwapp]$ dotnet restore
log : Restoring packages for /home/game/hwapp/project.json...

...（中略）...

Installed:
113 package(s) to /home/game/hwapp/project.json</pre>

最後に実行です。

<pre class="brush: bash; gutter: true">[user@centos7 hwapp]$ dotnet run
Project hwapp (.NETCoreApp,Version=v1.0) will be compiled because expected outputs are missing
Compiling hwapp for .NETCoreApp,Version=v1.0

Compilation succeeded.
0 Warning(s)
0 Error(s)

Time elapsed 00:00:05.1812463
Hello World!</pre>

こんにちは。

これからC#が様々なOSで開発に利用されていくのでしょうか。  
Unity使っている人にはとっつきやすくていいかも（？）ですね〜

以上です。

■ 参考  
LinuxでもC#プログラミング(導入編)

■ メモ  
No package libmpc-devel available.といわれたら  
→ [epelを追加する  
][2] 

ldconfig -p | grep hogehoge でビルドしたライブラリが見つからない場合[  
/usr/local/libを認識させる][3]

 [1]: https://www.microsoft.com/net/core#centos
 [2]: http://www.kakiro-web.com/linux/centos6-epel-install.html
 [3]: http://blog.goo.ne.jp/asakurah/e/7f9cdf8cbada8841c388ece8cd421432