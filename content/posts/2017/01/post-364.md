---
title: '[Golang] SideCIを設定してコードレビューしてもらう'
author: しゃまとん
type: post
date: 2017-01-28T14:44:10+00:00
url: /posts/364
featured_image: /images/posts/2017/01/sideci_icon.png
categories:
  - go
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

[SideCI][1]というサービスをご存知でしょうか。  
SideCIはGitHubと連携し、自動的に解析ツールを利用したコードレビューを行うサービスで、  
公開しているリポジトリだと無料で全機能を使うことができるようです。

無料だし、これは使うしか無い！ということで試しにレビューしてもらいました。  
※対応言語は[こちら][1]から確認してくださいね

とりあえずSideCIのホームページにアクセスして、「Githubで無料会員登録」をします。  
すると、連携の認証に進みます。

[<img src="https://shamaton.orz.hm/blog/images/posts/2017/01/recognize.png" alt="" width="500" height="303" class="aligncenter size-full wp-image-367" />][2]

問題なければ、Authorize applicationするとパスワードを聞かれるので対応します。その後SideCIの管理ページに飛ばされ、解析対象を選択する画面になります。言語のところにgopherがいますね。

[<img src="https://shamaton.orz.hm/blog/images/posts/2017/01/select.png" alt="" width="557" height="302" class="aligncenter size-full wp-image-368" />][3]

選択すると、（おそらく）デフォルトで設定された解析が始まります。解析が終わると、goの場合は[go_vet][4]と[golint][5]の結果が表示されています。golintに関しての表示を見てみるとこんな感じです。zeroformatter対応しないとダメですね (；・∀・)

[<img src="https://shamaton.orz.hm/blog/images/posts/2017/01/detail.png" alt="" width="648" height="286" class="aligncenter size-full wp-image-369" />][6]

上記はSideCIの管理画面からですが、GithubでPull Requestしたときにも同様に解析をおこなってくれます。リポジトリに対して特に何かしたわけではないのですが、勝手に解析してくれるようになっていました。

[<img src="https://shamaton.orz.hm/blog/images/posts/2017/01/run_github.png" alt="" width="394" height="310" class="aligncenter size-full wp-image-370" />][7]

<p style="text-align: center;">
  ↓
</p>

[<img src="https://shamaton.orz.hm/blog/images/posts/2017/01/result.png" alt="" width="391" height="196" class="aligncenter size-full wp-image-371" />][8]

このように簡単な手順だけで無料で解析してくれるようになりました。積極的に利用することでプロダクトの品質を高めてくれそうですね！

SideCIは日本語版も[提供開始した][9]そうです。英語苦手だけど、コードレビューされたい方は利用してみてはいかがでしょうか。  
以上です。

 [1]: https://sideci.com/ja
 [2]: https://shamaton.orz.hm/blog/images/posts/2017/01/recognize.png
 [3]: https://shamaton.orz.hm/blog/images/posts/2017/01/select.png
 [4]: http://golang-jp.org/pkg/code.google.com/p/go.tools/cmd/vet/
 [5]: https://github.com/golang/lint
 [6]: https://shamaton.orz.hm/blog/images/posts/2017/01/detail.png
 [7]: https//shamaton.orz.hm/blog/images/posts/2017/01/run_github.png
 [8]: https://shamaton.orz.hm/blog/images/posts/2017/01/result.png
 [9]: http://gamebiz.jp/?p=176423