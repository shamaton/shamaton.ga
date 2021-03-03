---
title: '[Unity] GitHub for Unityを触ってみたかった…'
author: しゃまとん
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

2017年3月頃にアナウンスされていて「GitHub for Unity」がついに公開されたようです！  
ということで、早速触ってみよーとした記事です。  
内容的には残念な結果に終わっています。

{{< blogcard url="https://unity.github.com/">}}

{{< blogcard url="http://forest.watch.impress.co.jp/docs/news/1067877.html">}}

ここから下は残念な結果に終わるまでの流れを記載しています。

* * *

必要なUnityのバージョンは5.4以上を対象としているようです。  
私は今回5.6.0f3で試してみました。

Github for UnityはUnityのEditorExtensionとして配布しているようです。  
[こちら][3]から最新のunitypackageをダウンロードして、使いたいプロジェクトにimportしてみます。  
今回は新規のプロジェクトを作成してやってみました。

{{< figure src="/images/posts/2017/07/cap01.png">}}
importが完了するとEditorフォルダが作成され、メニューにGithubが追加されます。

{{< figure src="/images/posts/2017/07/cap02.png">}}

Sign inというメニューがあったので、ログイン情報いれてみたんですが何も起きず...だったので、Windowを表示してみます。  
※（2017/07/02）どうやらSign inのメニューはまだBugがあるようで機能しないみたいです

{{< figure src="/images/posts/2017/07/cap03.png">}}

Windowを表示するとInitialize repositoryとあるのでクリックしてみたのですが、
反応が帰ってこず...ログを確認しても

```text
170702-14:43:13 TRACE [ 1] <IApplicationManager>               Running Repository Initialize
```

と表示されるだけでした。仕方がないということで、git initだけして再起動してみると、
表示が切り替わったので、アカウント名とかリポジトリのURLを設定してみたのですが反応がない...

{{< figure src="/images/posts/2017/07/cap04.png">}}

じゃあ、何かcommitしてみるかということで.gitignoreを作ってcommitして、
unity再起動してみたのですが、historyとかブランチとか何も表示されず&#8230;

どうしたらいいの！

バージョンが古いってことなの！

ってことで、念のため5.6.2f1をインストールして見たけどダメでした&#8230;  
使ってみた所感を書きたかったのですが、αですしそんなに期待してはいけなかったですね。

こんな風につかえたよ！みたいな記事が他で出ることを期待しつつ、終わります。  
以上です。

 [3]: https://github.com/github-for-unity/Unity/releases