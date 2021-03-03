---
title: '[Mac] pkgでインストールしたjenkinsのportを変更する'
author: しゃまとん
date: 2017-02-23T15:36:57+00:00
url: /posts/276
featured_image: /images/posts/2016/08/headshot.png
categories:
  - Mac
  - メモ

---
お世話になっております。  
しゃまとんです。

macのでjenkins導入方法は色々あるようですが、公式のサイトからpkgを取得してインストールした場合、
ポートを設定しているファイルがバイナリっぽくなっていて編集が出来ないっぽいです（できるかもしれないけど怖い）

調べてみるとやり方があるみたいのだったのでメモしておきます。  
実はとても簡単でコマンドを1つ実行すればOKです。（defaultsコマンドはplistを操作するコマンドです）

portを`58080`に変更します。
```shell
sudo defaults write /Library/Preferences/org.jenkins-ci httpPort 58080
```

上記を実行して再起動します。

```shell
sudo launchctl unload /Library/LaunchDaemons/org.jenkins-ci.plist
sudo launchctl load /Library/LaunchDaemons/org.jenkins-ci.plist
```

58080にアクセスしてみると、表示されました！

{{< figure src="/images/posts/2016/08/port58080.png" >}}

デフォルトでは8080を使っているので、他のサービスとバッティングしてしまう場合は変更しておくとよいですね。

以上です。

■ 参考
{{< blogcard url="http://qiita.com/t_n/items/22e6c5fd9f2ced3d5fc4" >}}
{{< blogcard url="https://three1415.wordpress.com/2014/12/29/changing-jenkins-port-on-mac-os-x/" >}}
