---
title: '[Unity] TextMeshProを使ってマイ絵文字をテキストに入れる'
author: しゃまとん
type: post
date: 2017-08-21T14:21:10+00:00
url: /archives/436
featured_image: /wp-content/uploads/2017/08/textmeshpro_7.gif
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

少し前にTextMeshProが無料化されたということで知っておいたほうがいいアセットの1つみたいです。  
どうやらUnity標準であるuGUIのTextよりもテキスト表現がリッチにできるみたいです。



Unity道場でも指南しているようなので、使い方を覚えておきたいものです。



そして、個人的に気になったのが、テキスト内に絵文字を入れられるところ。  
なんだか今時（？）の表現ができそうですね。  
TextMeshProの場合はマイ絵文字的な感じで、自分でテキスト内に表示した絵文字アセットを作成してやるのだそう。  
なので、スマホとかで入力できる絵文字がそのまま使えます！みたいなのではないですね。  
（Forumにそれっぽい話をしているけども）

とりあえず「TextMeshPro 絵文字」あたりでググったのですがあまり検索結果が出なかった。。。  
ので、マイ絵文字をテキストに表示できるところまでをやってみました。

まずは、[TextMeshPro][1]をゲットして展開します。

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2017/08/textmeshpro_1.png" alt="" width="479" height="451" class="aligncenter size-full wp-image-438" />][2]

そして、マイ絵文字にしたい画像を用意しておきます。  
サンプルにもあるように、使いたいものは1つのアトラスにしておくとよいみたいです。  
今回は[ICOOON MONO][3]さんの画像をお借りしました。

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2017/08/textmeshpro_2.png" alt="" width="279" height="180" class="aligncenter size-full wp-image-439" />][4]

現状だと公式にアトラスを作成してくれる機能はありません（多分）  
アトラスの作成方法はこちらで公開してくださっているものがあります。  
[【Unity】スプライトを1枚にまとめる簡易SpritePacker][5]

他の外部ツールで作成してもいいかもしれません。

アトラスが用意できたら、マテリアルを作成します。  
[右クリックメニュー] → [Create] → [TextMeshPro] → [Sprite Asset]

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2017/08/textmeshpro_3.gif" alt="" width="475" height="425" class="aligncenter size-full wp-image-440" />][6]

上記を実行すると、同じフォルダにマテリアルが作られます。  
中を確認すると、画像ごとに情報を作成されていますね。  
設定値は色々ありますが、表示位置だったりスケールを調整できたりします。

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2017/08/textmeshpro_4.png" alt="" width="244" height="361" class="aligncenter size-full wp-image-442" />][7]

作成されたマテリアルはResources/Sprite Assets配下に置く必要があるみたいなので、そちらへ移動させておきます。  
これで準備は調ったはずなので、テキストにマイ絵文字を表示させてみます。

適当にシーンを作成し、uGUIのTextと同じようにTextMeshProのTextを追加します。

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2017/08/textmeshpro_5.gif" alt="" width="388" height="453" class="aligncenter size-full wp-image-443" />][8]

あとはタグルールにそって記載します。するとテキスト表記内に絵文字が表示されます。

<pre class="lang:default decode:true ">money&lt;sprite="Emoji" index=0&gt;money&lt;sprite="Emoji" index=1&gt;money</pre>

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2017/08/textmeshpro_6.gif" alt="" width="817" height="157" class="aligncenter size-full wp-image-444" />][9]

これで今時なメッセージボックスが作成できそうですね！  
以上です。

 [1]: https://www.assetstore.unity3d.com/jp/#!/content/84126
 [2]: https://shamaton.orz.hm/blog/wp-content/uploads/2017/08/textmeshpro_1.png
 [3]: http://icooon-mono.com/
 [4]: https://shamaton.orz.hm/blog/wp-content/uploads/2017/08/textmeshpro_2.png
 [5]: http://caitsithware.com/wordpress/archives/263
 [6]: https://shamaton.orz.hm/blog/wp-content/uploads/2017/08/textmeshpro_3.gif
 [7]: https://shamaton.orz.hm/blog/wp-content/uploads/2017/08/textmeshpro_4.png
 [8]: https://shamaton.orz.hm/blog/wp-content/uploads/2017/08/textmeshpro_5.gif
 [9]: https://shamaton.orz.hm/blog/wp-content/uploads/2017/08/textmeshpro_6.gif