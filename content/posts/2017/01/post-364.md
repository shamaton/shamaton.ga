---
title: '[Golang] SideCIを設定してコードレビューしてもらう'
author: しゃまとん
date: 2017-01-28T14:44:10+00:00
url: /posts/364
featured_image: /images/posts/2017/01/sideci_icon.png
categories:
  - go
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

SideCIというサービスをご存知でしょうか。

{{< blogcard url="https://sider.review/ja" >}}

SideCIはGitHubと連携し、自動的に解析ツールを利用したコードレビューを行うサービスで、  
公開しているリポジトリだと無料で全機能を使うことができるようです。

無料だし、これは使うしか無い！ということで試しにレビューしてもらいました。

とりあえずSideCIのホームページにアクセスして、「Githubで無料会員登録」をします。  
すると、連携の認証に進みます。

{{< figure src="/images/posts/2017/01/recognize.png" >}}

問題なければ、Authorize applicationするとパスワードを聞かれるので対応します。
その後SideCIの管理ページに飛ばされ、解析対象を選択する画面になります。言語のところにgopherがいますね。

{{< figure src="/images/posts/2017/01/select.png" >}}

選択すると、（おそらく）デフォルトで設定された解析が始まります。解析が終わると、
goの場合は[go_vet][4]と[golint][5]の結果が表示されています。golintに関しての表示を見てみるとこんな感じです。
zeroformatter対応しないとダメですね (；・∀・)

{{< figure src="/images/posts/2017/01/detail.png" >}}

上記はSideCIの管理画面からですが、GithubでPull Requestしたときにも同様に解析をおこなってくれます。リポジトリに対して特に何かしたわけではないのですが、勝手に解析してくれるようになっていました。

{{< figure src="/images/posts/2017/01/run_github.png" >}}

結果↓

{{< figure src="/images/posts/2017/01/result.png" >}}

このように簡単な手順だけで無料で解析してくれるようになりました。積極的に利用することでプロダクトの品質を高めてくれそうですね！

SideCIは日本語版も[提供開始した][9]そうです。英語苦手だけど、コードレビューされたい方は利用してみてはいかがでしょうか。  
以上です。

{{< blogcard url="http://gamebiz.jp/?p=176423" >}}


 [4]: http://golang-jp.org/pkg/code.google.com/p/go.tools/cmd/vet/
 [5]: https://github.com/golang/lint