---
title: '[golang]Go1.5から1.6に更新した際のあれこれ（IntelliJとか）'
author: しゃまとん
date: 2016-07-01T16:22:09+00:00
url: /posts/217
featured_image: /images/posts/2016/06/IDEADEV.png
categories:
  - go
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

少し前（2016/02）にgo1.6がリリースされました。
で更新しなきゃと思っていたのですがやっていなかったのでやったというメモです。

ちなみに1.6の新機能に関してはコチラでまとめてくださっています。

{{< blogcard url="http://qiita.com/ksato9700/items/5505e506c20b6048c218" >}}

Go自体の更新（アップグレード）はとてもシンプルです。公式でも下記のように言われています。

> If you are upgrading from an older version of Go you must first [remove the existing version][1].

公式で配布されているパッケージを利用している再は/usr/local以下にインストールされているはずなので、下記ように実行します。
削除したくない場合はディレクトリ名を変更してもいいかと思います。

```shell
# 現在のバージョンを削除
rm -rf /usr/local/go

# 残しておきたい場合（例）
mv /usr/local/go /usr/local/go.1.5
```

コマンド実行後にダウンロードしたパッケージを実行するだけでアップグレードできます。
アップグレード後はおそらく管理者権限でインストールしているので、必要に応じてファイル権限を変更しておきます。

```shell
# your_name：ユーザー名をいれてください
chmod -R your_name:admin /usr/local/go
```

これで更新は完了ですが、外部のパッケージ（github.com）などをgo getしてた場合は
なくなっていると思うので再度go getが必要になるかもしれません。

**■ 1.6更新時のintelliJで対応したこと**

goの開発ではintelliJを使っているのですが、細かい設定を更新したのでこれも一応メモしておきます。
個人的に大きかったところとしてはvendor（外部パッケージ管理）の部分で、
GO15VENDOREXPERIMENTで提供されていた部分がデフォルトになったそうです。1.7でパラメータ自体が削除されるみたいです。

それではintelliJでGo関係の設定を変更していきます。

まずは既存のプロジェクトを開き、ツリーの一番したにあるExternal LibrariesからGoを右クリックし、
Open Library Settingsを選択します。 そしてNameのところを1.6にしておきます。

{{< figure src="/images/posts/2016/06/sdks.png" >}}

次にIntellij IDEA -> Preferences -> Languages & Frameworks -> Goを開き、Project Settingsを確認します。
ここではEnable vendoringが有効になっているかを確認しておきます。（デフォルトでは有効のはず）

{{< figure src="/images/posts/2016/06/projectsettings.png" >}}

もしDisableの状態でvendor配下の外部パッケージを参照しようとすると...

{{< figure src="/images/posts/2016/06/error.png" >}}

上記のようにエラーになります（赤色なだけだけど&#8230;）、正常な場合はコチラ

{{< figure src="/images/posts/2016/06/ok.png" >}}

Go Librariesは基本的に変更する必要なないと思いますが、何かあれば対応しておきましょう。

{{< figure src="/images/posts/2016/06/golibraries.png" >}}

なんか長くなってしまいましたが、vendoring対応してくれてよかったね！という感じですね！  
以上です。

■参考
{{< blogcard url="http://qiita.com/dorayaki_kun/items/6762a452010d42e38bd9" >}}
{{< blogcard url="http://junchang1031.hatenablog.com/entry/2016/03/12/175744" >}}
MacにGo言語の開発環境を構築する【IDE編】
{{< blogcard url="http://akirachiku.com/2016/03/01/go16-development.html" >}}


 [1]: https://golang.org/doc/install#uninstall