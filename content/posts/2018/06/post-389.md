---
title: '[Unity] クラスをKeyにしたDictionaryを作るには'
author: しゃまとん
type: post
date: 2018-06-24T13:46:11+00:00
url: /archives/389
featured_image: /wp-content/uploads/2017/03/dictionary.png
is_comment_form_freeze:
  - on
comment_form_freeze_message:
  - ' '
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

以前からclassをkeyにしたDictionaryって出来ないのかなーと思っていて、仕方なくnewなんかしてイケてないなーと思いながら実装していたのですが、こっちの方が良さげなのでメモしておきたいと思います。

やり方はただSystem.Typeを使えば良かったのですね。という話でした。  
ということで下記のような感じで確認コードを書いて適当なオブジェクトに付けておきます。

<pre class="lang:c# decode:true " title="TypeMap.cs">using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class TypeMap : MonoBehaviour {

  static private Dictionary&lt;System.Type, string&gt; map = new Dictionary&lt;System.Type, string&gt;() {
    {typeof(TestA),  "This is A"},
    // {typeof(TestB),  "This is B"},
    {typeof(TestC),  "This is C"},
  };

  // Use this for initialization
  void Start () {
    log&lt;TestA&gt;();
    log&lt;TestB&gt;();
    log&lt;TestC&gt;();
  }

  // check
  private void log&lt;T&gt;() {
    System.Type type = typeof(T);
    if (map.ContainsKey(type)) {
      Debug.Log(map[type]);
    } else {
      Debug.Log("not found type!!");
    }
  }
}

// sample classes
public class TestA {}
public class TestB {}
public class TestC {}</pre>

準備ができたら実行してみます。  
下図のような感じでclassをKeyにして、Valueを取得することができました。

[<img src="http://shamaton.orz.hm/blog/wp-content/uploads/2017/03/typemap.png" alt="" width="373" height="146" class="aligncenter size-full wp-image-394" />][1]

これだけだと何に使うのという感じなんですが、例えば指定のクラスに対して、指定のメソッドを実行するとか、指定のリクエストをするとか・・・でしょうか？  
何にせよ、つっかえが取れた感じなので今後に活かせそうかなと思います！

何かに参考になれば幸いです。  
以上です。

 [1]: http://shamaton.orz.hm/blog/wp-content/uploads/2017/03/typemap.png