---
title: windowsのキーボードをmacで使う
author: しゃまとん
type: post
date: 2015-01-06T04:43:28+00:00
url: /posts/61
featured_image: /images/posts/2015/01/keyboard_photo.jpg
categories:
  - Mac
  - メモ

---
MacでSierraをお使いの方は使えないのでこちらを参照してください〜

<https://shamaton.orz.hm/blog/posts/446>

お世話になっております。

しゃまとんです。

ちょっと前からmacを使いはじめました。  
その前まではwindowsPCを使っていて、<a href=" http://www.diatec.co.jp/products/det.php?prod_c=760" target="_blank" rel="noopener">filicoのmajestouch</a>を使用していたのですが、macでも使えたらいいなーと思いつつ、公式ではサポートしていないとのこと。

でも使いたい！とのことで、調べておりましたら、カスタマイズしている方もちらほらいたみたいなので、参考にしながら、設定してみました。

キーボード環境としては  
mac : JIS配列(スペースの横に英数・かながあるやつ)  
使いたいキーボード : <a title=" " href=" http://www.diatec.co.jp/products/det.php?prod_c=760" target="_blank" rel="noopener">flico majestouch(ちなみに茶軸)</a>  
となります。

<!--more-->

やりたいこととしては  
・macのJIS配列にキーアサインする  
これだけです。

キーアサインの変更に伴いインストールが必要なアプリケーションがあるので、インストールしてください。  
<a href="https://pqrs.org/osx/karabiner/seil.html.ja" target="_blank" rel="noopener">Seil</a>(旧PCKeyboardHack)  
<a href="https://pqrs.org/osx/karabiner/index.html.ja" target="_blank" rel="noopener">Karabiner</a>(旧KeyRemap4MacBook)  
高機能キーボードカスタマイズツールです。

では、順番に設定していきます。  
初めてキーボードを接続する場合、macから設定するように求められますので、言われた通りに入力すればOKです。

■CapsLockとCtrlを入れ替え  
何も接続していないとctrlがcapslockになっているので入れ替えます。

システム環境設定　→　キーボード　→　修飾キー  
として、Caps LockとControlを入れ替えます。キーボードはUSB Keyboardを選択してください。

[<img src="http://shamaton.orz.hm/blog/images/posts/2015/01/keyboard.png" alt="keyboard" width="300" height="273" />][1]

■変換/無変換/カタカナキーを有効にする  
下図のようにチェックをいれると、変換/無変換キーが英数・かな切り替えになり、カタカナキーが右Commandキーになります。

[<img src="http://shamaton.orz.hm/blog/images/posts/2015/01/seil.png" alt="seil" width="300" height="282" />][2]

■左右のOption, Commandキーの設定  
各キーで変更したいアサインに設定します。

Windowsキー(左Command)を左Optionに  
※図が間違ってました。Command\_L to Option\_Lでした。[  
<img src="http://shamaton.orz.hm/blog/images/posts/2015/01/kara1.png" alt="kara1" width="300" height="290" />  
][3] 左Altを左Commandに  
[<img src="http://shamaton.orz.hm/blog/images/posts/2015/01/kara2.png" alt="kara2" width="300" height="290" />][4]  
右Altをfnにする[  
<img src="http://shamaton.orz.hm/blog/images/posts/2015/01/kara3.png" alt="kara3" width="300" height="290" />][5]  
右Altをfnキーにする。(音量とか操作できるように)  
[<img src="http://shamaton.orz.hm/blog/images/posts/2015/01/kara5.png" alt="kara5" width="300" height="290" />][6]

■半角/全角キーも有効に(ついで)  
デフォルトだとバッククォートになっているので、ついでに変更しておきます。

[<img src="http://shamaton.orz.hm/blog/images/posts/2015/01/kara4.png" alt="kara4" width="300" height="290" />][7]

■外部キーボードだけ有効にする  
忘れずに設定しておかないと、内蔵キーボードにも設定が反映されてしまうので、チェックしておきます。

[<img src="http://shamaton.orz.hm/blog/images/posts/2015/01/kara.png" alt="kara" width="469" height="454" />][8]

これでmacと同じ配列にキーボードを設定することができました。  
できれば、キーボードの刻印も変更できたらいいのですが、ここは我慢ですねヽ(´ｴ\`)ﾉ  
以上です。

■参考リンク  
<a href="http://www.airpucci.com/2013/10/windows%E3%82%AD%E3%83%BC%E3%83%9C%E3%83%BC%E3%83%89%E3%82%92mac%E3%81%A7%E4%BD%BF%E3%81%86/" target="_blank" rel="noopener">WindowsキーボードをMacで使う</a>  
<a href="http://fukajun.org/28" target="_blank" rel="noopener">MacにWindowsキーボードをつないだ時の覚書</a>  
<a href="http://mac-physics.ldblog.jp/archives/53635159.html" target="_blank" rel="noopener">Option keyをfn keyにしたい…KeyRemap4MacBook</a>  
<a href="http://free3.seesaa.net/article/237110319.html" target="_blank" rel="noopener">WindowsキーボードFILCO Majestouch Tenkeyless をMacでも使いやすくKeyremapする</a>

 [1]: http://shamaton.orz.hm/blog/images/posts/2015/01/keyboard.png
 [2]: http://shamaton.orz.hm/blog/images/posts/2015/01/seil.png
 [3]: http://shamaton.orz.hm/blog/images/posts/2015/01/kara1.png
 [4]: http://shamaton.orz.hm/blog/images/posts/2015/01/kara2.png
 [5]: http://shamaton.orz.hm/blog/images/posts/2015/01/kara3.png
 [6]: http://shamaton.orz.hm/blog/images/posts/2015/01/kara5.png
 [7]: http://shamaton.orz.hm/blog/images/posts/2015/01/kara4.png
 [8]: http://shamaton.orz.hm/blog/images/posts/2015/01/kara.png