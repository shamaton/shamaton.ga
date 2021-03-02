---
title: iTermでいい感じにテキストを選択したい
author: しゃまとん
type: post
date: 2016-10-20T14:26:11+00:00
url: /posts/257
featured_image: /images/posts/2016/08/iterm_icon.png
categories:
  - Mac

---
[<img src="http://shamaton.orz.hm/blog/images/posts/2016/08/iterm.jpg" alt="iterm" width="400" height="156" class="aligncenter size-full wp-image-262" />][1]

お世話になっております。  
しゃまとんです。

Macではデフォルトでターミナルが入っていますが、自分は[iTerm][2]というターミナルアプリケーションを使って普段CUI操作しています。

そのiTerm使っている理由は色々あるのですが、1つにウインドウ上の文字の選択をカスタマイズできるという点です。

ためしにプリインストールされているターミナルで時間（コロンで句切られた文字）をダブルクリックしてみると、こんな感じで分が選択された状態になります。

[<img src="http://shamaton.orz.hm/blog/images/posts/2016/08/term.png" alt="term" width="170" height="22" class="aligncenter size-full wp-image-264" />][3]

おそらくこのテキストをクリックした時って、時間全体を選択してくれたほうが便利ですよね。この辺を便利さがiTermにはあります。

これをiTermでダブルクリックしてみると・・・全部選択された状態になります。  
ダブルクリックするだけなので仕事が捗りますね（多分）

[<img src="http://shamaton.orz.hm/blog/images/posts/2016/08/iterm_1.png" alt="iterm_1" width="170" height="18" class="aligncenter size-full wp-image-265" />][4]

さらにiTermでは、いい感じに選択してくれる設定を変更できるようになっています。  
Preference → General → Selection → Characters considered part of word for selection  
に記号が入ってると思います。そこに考慮した文字を入れてやればOKです。

[<img src="http://shamaton.orz.hm/blog/images/posts/2016/08/iterm_2.png" alt="iterm_2" width="778" height="516" class="aligncenter size-full wp-image-266" />][5]

例えば、デフォルトでは@がないので直前までが選択範囲になります

[<img src="http://shamaton.orz.hm/blog/images/posts/2016/08/iterm_3.png" alt="iterm_3" width="170" height="18" class="aligncenter size-full wp-image-267" />][6]

が、このように設定しておくと&#8230;

[<img src="http://shamaton.orz.hm/blog/images/posts/2016/08/iterm_4.png" alt="iterm_4" width="341" height="63" class="aligncenter size-full wp-image-268" />][7]

このように含めてくれるようになります。  
まとめて選択してほしいものを追加しておくとさらに仕事が捗りますね！（多分）

[<img src="http://shamaton.orz.hm/blog/images/posts/2016/08/iterm_5.png" alt="iterm_5" width="170" height="16" class="aligncenter size-full wp-image-269" />][8]

iTerm素敵ですね！  
以上です。

 [1]: http://shamaton.orz.hm/blog/images/posts/2016/08/iterm.jpg
 [2]: https://www.iterm2.com/
 [3]: http://shamaton.orz.hm/blog/images/posts/2016/08/term.png
 [4]: http://shamaton.orz.hm/blog/images/posts/2016/08/iterm_1.png
 [5]: http://shamaton.orz.hm/blog/images/posts/2016/08/iterm_2.png
 [6]: http://shamaton.orz.hm/blog/images/posts/2016/08/iterm_3.png
 [7]: http://shamaton.orz.hm/blog/images/posts/2016/08/iterm_4.png
 [8]: http://shamaton.orz.hm/blog/images/posts/2016/08/iterm_5.png