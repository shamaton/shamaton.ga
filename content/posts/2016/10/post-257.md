---
title: iTermでいい感じにテキストを選択したい
author: しゃまとん
date: 2016-10-20T14:26:11+00:00
url: /posts/257
featured_image: /images/posts/2016/08/iterm_icon.png
categories:
  - Mac

---

{{<figure src="/images/posts/2016/08/iterm.jpg" class="center">}}
お世話になっております。  
しゃまとんです。

Macではデフォルトでターミナルが入っていますが、
自分はiTermというターミナルアプリケーションを使って普段CUI操作しています。

{{< blogcard url="https://iterm2.com/" >}}

そのiTerm使っている理由は色々あるのですが、1つにウインドウ上の文字の選択をカスタマイズできるという点です。

ためしにプリインストールされているターミナルで時間（コロンで句切られた文字）をダブルクリックしてみると、
こんな感じで分が選択された状態になります。

{{< figure src="/images/posts/2016/08/term.png" >}}

おそらくこのテキストをクリックした時って、時間全体を選択してくれたほうが便利ですよね。この辺を便利さがiTermにはあります。

これをiTermでダブルクリックしてみると・・・全部選択された状態になります。  
ダブルクリックするだけなので仕事が捗りますね（多分）

{{< figure src="/images/posts/2016/08/iterm_1.png" >}}

さらにiTermでは、いい感じに選択してくれる設定を変更できるようになっています。  
Preference → General → Selection → Characters considered part of word for selection  
に記号が入ってると思います。そこに考慮した文字を入れてやればOKです。

{{< figure src="/images/posts/2016/08/iterm_2.png" >}}

例えば、デフォルトでは@がないので直前までが選択範囲になります

{{< figure src="/images/posts/2016/08/iterm_3.png" >}}

が、このように設定しておくと...

{{< figure src="/images/posts/2016/08/iterm_4.png" >}}

このように含めてくれるようになります。  
まとめて選択してほしいものを追加しておくとさらに仕事が捗りますね！（多分）

{{< figure src="/images/posts/2016/08/iterm_5.png" >}}

iTerm素敵ですね！  
以上です。
