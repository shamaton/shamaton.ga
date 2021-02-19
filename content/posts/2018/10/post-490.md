---
title: '[Unity] 利用規約をつくったときのこと'
author: しゃまとん
type: post
date: 2018-10-13T13:47:33+00:00
url: /archives/490
featured_image: /wp-content/uploads/2017/12/EULA.png
categories:
  - メモ
  - 開発

---
お世話になっております。  
しゃまとんです。

ユーザー入力ができるアプリを作ったときのお話です。

最初のバージョンを申請したところリジェクトになりました。  
理由としてUser Generated Contentに関するものでした。原文はこのような感じです。

<pre class="lang:default decode:true">Fixed Guideline 1.2 - Safety - User Generated Content
Your app enables users to post content anonymously but does not have the proper precautions in place.

Next Steps

To resolve this issue, please revise your app to implement all of the following precautions:

- Age rating must reflect 17+
- Require that users agree to terms (EULA) and these terms must make it clear that there is no tolerance for objectionable content or abusive users
- A method for filtering objectionable content
- A mechanism for users to flag objectionable content
- A mechanism for users to block abusive users
- A mechanism for users to immediately remove posts from the feed
- Developer must act on objectionable content reports within 24 hours by removing the content and ejecting the user who provided the offending content
- Developer must provide contact information in the app itself, giving users the ability to report inappropriate activity</pre>

まぁ、とりあえずGoogle翻訳を通しておくと&#8230;

<pre class="lang:default decode:true ">あなたのアプリでは、ユーザーは匿名でコンテンツを投稿できますが、適切な予防措置は講じていません。

次のステップ

この問題を解決するには、アプリを改訂して次の予防措置をすべて実施してください：

- 年齢層は17歳以上を反映する必要があります
- ユーザーが利用規約（EULA）に同意することを要求する。これらの条件は、好ましくないコンテンツや不正なユーザーに対しては許容範囲がないことを明確にしなければならない
- 好ましくないコンテンツをフィルタリングする方法
- ユーザーが不快なコンテンツにフラグを立てる仕組み
- ユーザーが不正なユーザーをブロックする仕組み
- ユーザーがフィードから投稿をすぐに削除する仕組み
- 開発者は、コンテンツを削除し、問題のコンテンツを提供したユーザーを取り除くことにより、好ましくないコンテンツレポートを24時間以内に実行する必要があります
- 開発者は、アプリケーション自体に連絡先情報を提供し、不適切な活動を報告する機能をユーザーに提供する必要があります</pre>

うむ、なるほど&#8230;と。たしかにその通りすぎて、突っぱねる返答など微塵もするできる感じじゃなかったので、納得してくれそうな感じで対応することに。

ほとんどの内容はコンテンツ内容の対応と申請時の年齢設定で改善出来そうだなーと思っていたんですが、1つややこしいのが。

利用規約（End User License Agreement）

です。難しい文章とか硬い感じのしっかりした文章考えないといけないよなー&#8230;とうなっていたんですが、こういうときは他のアプリを参考にするしかない！ということでコミュニケーションアプリといえば[LINEの利用規約][1]を参考にすることにしました。

（そもそもEULAとはという方は[こちら][2]）

自分にアプリに必要なので読まざるを得ないとは言え  
利用規約をこんなにしっかり読むことは今までなかったです。。  
読んだらわかりやすい内容になっていることがわかったので、参考に自分のアプリ用の利用規約を作成しました。

あとはアプリに報告や問い合わせができるように機能を追加して、再度申請しました。  
これで申請が通り、無事リリースすることができました。

このようなものを作らなかったら経験できなかったことかもしれません。  
他のアプリ時に役に立つかもしれないし、勉強になりました。

以上です。

 [1]: https://terms.line.me/line_terms/?lang=ja
 [2]: http://e-words.jp/w/EULA.html