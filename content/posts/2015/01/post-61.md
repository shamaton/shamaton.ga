---
title: windowsのキーボードをmacで使う
author: しゃまとん
date: 2015-01-06T04:43:28+00:00
url: /posts/61
featured_image: /images/posts/2015/01/keyboard_photo.jpg
categories:
  - Mac
  - メモ

---
MacでSierraをお使いの方は使えないのでこちらを参照してください〜

{{< blogcard url="/posts/446" >}}

お世話になっております。

しゃまとんです。

ちょっと前からmacを使いはじめました。  
その前まではwindowsPCを使っていて、filcoのMajestouchを使用していたのですが、macでも使えたらいいなーと思いつつ、
公式ではサポートしていないとのこと。

{{< blogcard url="http://www.diatec.co.jp/products/det.php?prod_c=760" >}}

でも使いたい！とのことで、調べておりましたら、カスタマイズしている方もちらほらいたみたいなので、
参考にしながら、設定してみました。

キーボード環境としては  
mac : JIS配列(スペースの横に英数・かながあるやつ)  
使いたいキーボード : flico majestouch(ちなみに茶軸)  
となります。

<!--more-->

やりたいこととしては  
* macのJIS配列にキーアサインする

これだけです。

キーアサインの変更に伴いインストールが必要なアプリケーションがあるので、インストールしてください。

[Seil(旧PCKeyboardHack)](https://pqrs.org/osx/karabiner/seil.html.ja)  
[Karabiner(旧KeyRemap4MacBook)](https://pqrs.org/osx/karabiner/index.html.ja)  

高機能キーボードカスタマイズツールです。

では、順番に設定していきます。  
初めてキーボードを接続する場合、macから設定するように求められますので、言われた通りに入力すればOKです。

■CapsLockとCtrlを入れ替え  
何も接続していないとctrlがcapslockになっているので入れ替えます。

システム環境設定　→　キーボード　→　修飾キー  
として、Caps LockとControlを入れ替えます。キーボードはUSB Keyboardを選択してください。

{{< figure src="/images/posts/2015/01/keyboard.png" >}}

■変換/無変換/カタカナキーを有効にする  
下図のようにチェックをいれると、変換/無変換キーが英数・かな切り替えになり、カタカナキーが右Commandキーになります。

{{< figure src="/images/posts/2015/01/seil.png" >}}

■左右のOption, Commandキーの設定  
各キーで変更したいアサインに設定します。

Windowsキー(左Command)を左Optionに  
※図が間違ってました。Command\_L to Option\_Lでした。

{{< figure src="/images/posts/2015/01/kara1.png" >}}

左Altを左Commandに  

{{< figure src="/images/posts/2015/01/kara2.png" >}}

右Altをfnにする

{{< figure src="/images/posts/2015/01/kara3.png" >}}

右Altをfnキーにする。(音量とか操作できるように)  
{{< figure src="/images/posts/2015/01/kara5.png" >}}

■半角/全角キーも有効に(ついで)  
デフォルトだとバッククォートになっているので、ついでに変更しておきます。

{{< figure src="/images/posts/2015/01/kara4.png" >}}

■外部キーボードだけ有効にする  
忘れずに設定しておかないと、内蔵キーボードにも設定が反映されてしまうので、チェックしておきます。

{{< figure src="/images/posts/2015/01/kara.png" >}}
[<img src="http://shamaton.orz.hm/blog/images/posts/2015/01/kara.png" alt="kara" width="469" height="454" />][8]

これでmacと同じ配列にキーボードを設定することができました。  
できれば、キーボードの刻印も変更できたらいいのですが、ここは我慢ですねヽ(´ｴ\`)ﾉ  
以上です。

■参考リンク  
{{< blogcard url="http://free3.seesaa.net/article/237110319.html" >}}