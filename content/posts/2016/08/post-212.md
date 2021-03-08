---
title: '[Unity5]OnCollision系とかOnTrigger系のコールバックを入力補間する'
author: しゃまとん
date: 2016-08-15T15:01:28+00:00
url: /posts/212
featured_image: /images/posts/2016/06/codetemplate.gif
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

MonoBehaviourにあるOnCollisionEnterやOnTriggerEnterなどの衝突判定時のコールバック系メソッドを
実装しようと思ったら都度調べたりして、面倒なことがあります。

そこでエディタでOn&#8230;などと入力すると補間してくれるの機能があるので、利用するのが吉です。
入力補間できるようにするファイルも公開されています。

今回は以下の環境になります。

  * OS：Mac
  * Unity：Unity5
  * エディタ：MonoDevelop、Unity5での対応



いつもの如くテラシュールさんのブログが参考になるのですが、
公開されているファイルの適用方法が少しだけ変わっているのでメモしておきます。

まずはこちらからzipもしくはgit経由で取得してきます。

{{< blogcard url="https://github.com/anchan828/unity-snippets" >}}

{{< figure src="/images/posts/2016/06/snippets.png" >}}

取得したら、Unity.template.xmlをMonoDevelop側にコピーします。配置場所は

```text
/Users/{UserName}/Library/MonoDevelop-Unity-5.0/Snippets
```

{{< figure src="/images/posts/2016/06/monodevelop.png" >}}

となります。この状態でMonoDevelopを再起動しておきます。  
これでOn...と入力してみると

{{< figure src="/images/posts/2016/06/codetemplate.gif" >}}

上記のように補間してくれるようになります。  
補間された後にTabを押すことでテンプレートが展開されます。テンプレートはこれ以外にも色々使えそうですね。

以上です。
