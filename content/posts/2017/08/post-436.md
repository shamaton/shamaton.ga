---
title: '[Unity] TextMeshProを使ってマイ絵文字をテキストに入れる'
author: しゃまとん
date: 2017-08-21T14:21:10+00:00
url: /posts/436
featured_image: /images/posts/2017/08/textmeshpro_7.gif
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

少し前にTextMeshProが無料化されたということで知っておいたほうがいいアセットの1つみたいです。  
どうやらUnity標準であるuGUIのTextよりもテキスト表現がリッチにできるみたいです。

{{< youtube MX_pM8QKTAc >}}

Unity道場でも指南しているようなので、使い方を覚えておきたいものです。

{{<slideshare yIQTqlvyJ0Any8 >}}

そして、個人的に気になったのが、テキスト内に絵文字を入れられるところ。  
なんだか今時（？）の表現ができそうですね。  
TextMeshProの場合はマイ絵文字的な感じで、自分でテキスト内に表示した絵文字アセットを作成してやるのだそう。  
なので、スマホとかで入力できる絵文字がそのまま使えます！みたいなのではないですね。  
（Forumにそれっぽい話をしているけども）

とりあえず「TextMeshPro 絵文字」あたりでググったのですがあまり検索結果が出なかった。。。  
ので、マイ絵文字をテキストに表示できるところまでをやってみました。

まずは、[TextMeshPro][1]をゲットして展開します。

{{<figure src="/images/posts/2017/08/textmeshpro_1.png">}}

そして、マイ絵文字にしたい画像を用意しておきます。  
サンプルにもあるように、使いたいものは1つのアトラスにしておくとよいみたいです。  
今回は[ICOOON MONO][3]さんの画像をお借りしました。

{{<figure src="/images/posts/2017/08/textmeshpro_2.png">}}

現状だと公式にアトラスを作成してくれる機能はありません（多分）  
アトラスの作成方法はこちらで公開してくださっているものがあります。  
[【Unity】スプライトを1枚にまとめる簡易SpritePacker][5]

他の外部ツールで作成してもいいかもしれません。

アトラスが用意できたら、マテリアルを作成します。  
[右クリックメニュー] → [Create] → [TextMeshPro] → [Sprite Asset]

{{<figure src="/images/posts/2017/08/textmeshpro_3.gif">}}

上記を実行すると、同じフォルダにマテリアルが作られます。  
中を確認すると、画像ごとに情報を作成されていますね。  
設定値は色々ありますが、表示位置だったりスケールを調整できたりします。

{{<figure src="/images/posts/2017/08/textmeshpro_4.png">}}

作成されたマテリアルはResources/Sprite Assets配下に置く必要があるみたいなので、そちらへ移動させておきます。  
これで準備は調ったはずなので、テキストにマイ絵文字を表示させてみます。

適当にシーンを作成し、uGUIのTextと同じようにTextMeshProのTextを追加します。

{{<figure src="/images/posts/2017/08/textmeshpro_5.gif">}}

あとはタグルールにそって記載します。するとテキスト表記内に絵文字が表示されます。

```text
money<sprite="Emoji" index=0>money<sprite="Emoji" index=1>money
```

{{<figure src="/images/posts/2017/08/textmeshpro_6.gif">}}

これで今時なメッセージボックスが作成できそうですね！  
以上です。

 [1]: https://www.assetstore.unity3d.com/jp/#!/content/84126
 [3]: http://icooon-mono.com/
 [5]: http://caitsithware.com/wordpress/posts/263