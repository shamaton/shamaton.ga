---
title: '[Unity] GitHub for Unityを触ってみたかった…'
author: しゃまとん
type: post
date: 2017-07-02T06:40:20+00:00
url: /posts/419
featured_image: /images/posts/2016/12/GitHub-Mark-120px-plus.png
categories:
  - git
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

2017年3月頃にアナウンスされていて[GitHub for Unity][1]がついに公開されたようです！  
ということで、早速触ってみよーとした記事です。  
内容的には残念な結果に終わっています。

[GitHub、「GitHub for Unity」をオープンソース化 ～無償でダウンロード可能][2]

ここから下は残念な結果に終わるまでの流れを記載しています。

* * *

必要なUnityのバージョンは5.4以上を対象としているようです。  
私は今回5.6.0f3で試してみました。

Github for UnityはUnityのEditorExtensionとして配布しているようです。  
[こちら][3]から最新のunitypackageをダウンロードして、使いたいプロジェクトにimportしてみます。  
今回は新規のプロジェクトを作成してやってみました。

[<img src="https://shamaton.orz.hm/blog/images/posts/2017/07/cap01.png" alt="" width="305" height="363" class="aligncenter wp-image-420 size-full" />][4]

importが完了するとEditorフォルダが作成され、メニューにGithubが追加されます。

[<img src="https://shamaton.orz.hm/blog/images/posts/2017/07/cap02.png" alt="" width="630" height="23" class="aligncenter wp-image-421 size-full" />][5]

Sign inというメニューがあったので、ログイン情報いれてみたんですが何も起きず&#8230;だったので、Windowを表示してみます。  
※（2017/07/02）どうやらSign inのメニューはまだBugがあるようで機能しないみたいです

[<img src="https://shamaton.orz.hm/blog/images/posts/2017/07/cap03.png" alt="" width="285" height="454" class="aligncenter wp-image-423 size-full" />][6]

Windowを表示するとInitialize repositoryとあるのでクリックしてみたのですが、反応が帰ってこず&#8230;ログを確認しても

<pre class="lang:default decode:true">170702-14:43:13 TRACE [ 1] &lt;IApplicationManager&gt;               Running Repository Initialize</pre>

と表示されるだけでした。仕方がないということで、git initだけして再起動してみると、表示が切り替わったので、アカウント名とかリポジトリのURLを設定してみたのですが反応がない&#8230;

[<img src="https://shamaton.orz.hm/blog/images/posts/2017/07/cap04.png" alt="" width="304" height="556" class="aligncenter wp-image-424 size-full" />][7]

じゃあ、何かcommitしてみるかということで.gitignoreを作ってcommitして、unity再起動してみたのですが、historyとかブランチとか何も表示されず&#8230;

どうしたらいいの！

バージョンが古いってことなの！

ってことで、念のため5.6.2f1をインストールして見たけどダメでした&#8230;  
使ってみた所感を書きたかったのですが、αですしそんなに期待してはいけなかったですね。

こんな風につかえたよ！みたいな記事が他で出ることを期待しつつ、終わります。  
以上です。

 [1]: https://unity.github.com/
 [2]: http://forest.watch.impress.co.jp/docs/news/1067877.html
 [3]: https://github.com/github-for-unity/Unity/releases
 [4]: https://shamaton.orz.hm/blog/images/posts/2017/07/cap01.png
 [5]: https://shamaton.orz.hm/blog/images/posts/2017/07/cap02.png
 [6]: https://shamaton.orz.hm/blog/images/posts/2017/07/cap03.png
 [7]: https://shamaton.orz.hm/blog/images/posts/2017/07/cap04.png