---
title: '[Unity]PC用とかに解像度対応どうするか'
author: しゃまとん
date: 2016-04-23T11:45:12+00:00
url: /posts/171
featured_image: /images/posts/2016/03/unity-logo.png
categories:
  - unity
  - プログラミング関連

---
お世話になります。  
しゃまとんです。

いつも問題になる解像度についてなのですが、どの辺を設定しておけばうまいことやれるかなーという忘備録になります。

PC用にゲーム作る場合ってどうするのかな、ユーザーによってフルスクリーンだったり、
この解像度で遊びたいとかありそうだなーともやもやしているのですが、
やはり基本解像度を決めてあとは拡縮させるのがよさ気だなという結論に至りました。

※まだ何も作ってないのでアレですが。。

解像度の周りの設定として、2要素の設定が大事かなと思いました  
* Player Setting  
* Canvas

■Player Setting  
遊ぶときにどのような環境でプレーさせるかの設定。個人的には起動時フルスクリーンのチェックも外したい。  
* Default Screen Width / heightに基本解像度を指定  
* Resizable Windowのチェックを外す  
* Supported Aspect Ratioでアスペクト比を制御

この辺りを設定することで意図した解像度でゲームが表示されるでしょうか。

{{< figure src="/images/posts/2016/04/playersetting.png" >}}

■Canvas  
キャンバスはPlayer Settingとは違い全体設定ではないですが、UIの描画に大きく関わっているので追加するキャンバス毎に対応が必要かと思われます。Canvasに付いているCanvas Scalerで設定できます。  
* UI Scale Modeを[Scale With Screen Size]にする  
* Reference Resolutionを基本解像度にする  
* Screen Match ModeをExpandにする（場合によって）

この辺りを設定すればよさそうです。

{{< figure src="/images/posts/2016/04/canvas_scaler.png" >}}

それと、設定ではないけど見えてはいけない領域対策として何かタイリングして隠すものを用意しておくよさげかなーと思いました。

あと、検証がてらですがPixel Per Unitも注意しておかないと、意図した大きさで表示されなかったりしますね。

この辺は設定をいじってみて、実際の動作を確認すると理解が早いかなと思います。  
以上です。

■参考
{{< blogcard url="http://tsubakit1.hateblo.jp/entry/2014/12/11/223427" >}}