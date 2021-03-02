---
title: '[Electron]Electronの開発環境を作ってビルドしてみる'
author: しゃまとん
date: 2017-10-03T15:04:56+00:00
url: /posts/452
featured_image: /images/posts/2017/09/electlon_logo.png
is_comment_form_freeze:
  - on
comment_form_freeze_message:
  - ' '
categories:
  - Electron
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

Electronご存知でしょうか。

{{< blogcard url="https://electron.atom.io/">}}

ElectronとはWebの技術でデスクトップアプリケーションを作ることができるものだそうです。  
なにやら新しそう感ですね。  
いいところは、クロスプラットフォームなところでOSを気にせずアプリケーションが作れるという優れもの。
Slackもコレを使って作られているらしいです。  
ということで、とりあえずMacで環境つくってビルドするところまでやってみました。

どうやら開発にはnode.jsが必要なようでして、その辺をやっていきます。  
まずはnodebrewを入れてnodeを使えるようにします。

```shell
$ brew install nodebrew

# ...インストールメッセージが流れる...

==> Summary
 /usr/local/Cellar/nodebrew/0.9.7: 8 files, 38.2KB, built in 4 seconds
```

一応バージョンを確認しておいて使える一覧を表示してみます。

```shell
$ nodebrew -v
nodebrew 0.9.7

# 使えるバージョン一覧
$ nodebrew ls-remote

v8.0.0 v8.1.0 v8.1.1 v8.1.2 v8.1.3 v8.1.4 v8.2.0 v8.2.1
v8.3.0 v8.4.0 v8.5.0 v8.6.0
```

今回は特にバージョン指定をせず、最新のものを使います。  
インストール前にディレクトリを準備し、実行します。

```shell
# インストール事前準備
$ mkdir -p ~/.nodebrew/src

# 最新をインストール
$ nodebrew install-binary latest
Fetching: https://nodejs.org/dist/v8.6.0/node-v8.6.0-darwin-x64.tar.gz
######################################################################## 100.0%
Installed successfully
```

次に最新版を使うように設定します。

```shell
# ローカル環境確認
$ nodebrew list
v8.6.0

current: none

# 利用設定
$ nodebrew use latest
use v8.6.0

# 再確認
$ nodebrew list
v8.6.0

current: v8.6.0
```

最後にパスを通して、簡単に確認します（ターミナルを再起動等する）  
パスはbashrc等に追記します。

```shell
export PATH=$PATH:/Users/${username}/.nodebrew/current/bin
```

```shell
$ node -v
v8.6.0

$ npm -v
5.3.0
```

これでnodeを使う準備が整いました。  
さらにElectron関係を進めます。まずは必要なパッケージを入れます。  
2つだけでいいみたいです。

```shell
$ npm install -g electron
$ npm install -g electron-packager
```

多分、開発出来る状態になったので、[awesome-electron][2]から適当に選んで試してみます。  
今回は**upterm**にしました。  
githubの説明の通りにすすめて、`npm start`してみます。

{{< blogcard url="https://github.com/railsware/upterm.git" >}}

```shell
# cloneして移動
$ git clone https://github.com/railsware/upterm.git
$ cd upterm

# 起動
$ npm start
npm WARN deprecated strtok2@1.0.4: depricated, use strtok3 & music-metadata instead
npm WARN deprecated minimatch@2.0.10: Please update to minimatch 3.0.2 or higher to avoid a RegExp DoS issue
npm WARN deprecated safe-to-string-x@2.0.3: Moved to https://github.com/Xotic750/to-string-symbols-supported-x
...
npm notice created a lockfile as package-lock.json. You should commit this file.
added 1100 packages in 61.615s
```

コンソールにメッセージが流れたあと、アプリケーションが立ち上がります。

{{< figure src="/images/posts/2017/09/electron_upterm.png">}}

次にappを作成してみます。少し時間がかかります。  
環境によってはもしかすると署名等で失敗するかもしれません。

```shell
$ npm run pack
...ビルドメッセージ...
Building macOS zip
Building DMG
```

Building DMGでdmgが作成されています。distフォルダ出来ているので確認してみます。

{{< figure src="/images/posts/2017/09/electron_dist.png">}}

実行すると、appをインストールすることができます。

{{< figure src="/images/posts/2017/09/electron_upterm_dmg.png">}}

簡単ですが、環境準備してビルドするところまでをやってみました。  
JS詳しい人は色々すぐに作れそうで羨ましい！  
以上です。

■ 参考
{{< blogcard url="https://qiita.com/hal_99/items/6e464f3d45d531ff9336">}}

 [2]: https://github.com/sindresorhus/awesome-electron