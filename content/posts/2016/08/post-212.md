---
title: '[Unity5]OnCollision系とかOnTrigger系のコールバックを入力補間する'
author: しゃまとん
type: post
date: 2016-08-15T15:01:28+00:00
url: /archives/212
featured_image: /wp-content/uploads/2016/06/codetemplate.gif
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

MonoBehaviourにあるOnCollisionEnterやOnTriggerEnterなどの衝突判定時のコールバック系メソッドを実装しようと思ったら都度調べたりして、面倒なことがあります。

そこでエディタでOn&#8230;などと入力すると補間してくれるの機能があるので、利用するのが吉です。入力補間できるようにするファイルも公開されています。

今回は以下の環境になります。

  * OS：Mac
  * Unity：Unity5
  * エディタ：MonoDevelop、Unity5での対応



いつもの如くテラシュールさんのブログが参考になるのですが、公開されているファイルの適用方法が少しだけ変わっているのでメモしておきます。

まずはこちらからzipもしくはgit経由で取得してきます。  
<https://github.com/anchan828/unity-snippets>

[<img src="http://shamaton.orz.hm/blog/wp-content/uploads/2016/06/snippets.png" alt="snippets" width="483" height="254" class="aligncenter size-full wp-image-214" />][1]

取得したら、Unity.template.xmlをMonoDevelop側にコピーします。配置場所は

<pre class="brush: text; gutter: true">/Users/{UserName}/Library/MonoDevelop-Unity-5.0/Snippets</pre>

[<img src="http://shamaton.orz.hm/blog/wp-content/uploads/2016/06/monodevelop.png" alt="monodevelop" width="500" height="253" class="aligncenter size-full wp-image-213" />][2]

となります。この状態でMonoDevelopを再起動しておきます。  
これでOn&#8230;と入力してみると

[<img src="http://shamaton.orz.hm/blog/wp-content/uploads/2016/06/codetemplate.gif" alt="codetemplate" width="500" height="286" class="aligncenter size-full wp-image-215" />][3]

上記のように補間してくれるようになります。  
補間された後にTabを押すことでテンプレートが展開されます。テンプレートはこれ以外にも色々使えそうですね。

以上です。

 [1]: http://shamaton.orz.hm/blog/wp-content/uploads/2016/06/snippets.png
 [2]: http://shamaton.orz.hm/blog/wp-content/uploads/2016/06/monodevelop.png
 [3]: http://shamaton.orz.hm/blog/wp-content/uploads/2016/06/codetemplate.gif