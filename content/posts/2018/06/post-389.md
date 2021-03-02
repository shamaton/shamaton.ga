---
title: '[Unity] クラスをKeyにしたDictionaryを作るには'
author: しゃまとん
date: 2018-06-11T13:46:11+00:00
url: /posts/389
featured_image: /images/posts/2017/03/dictionary.png
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

以前からclassをkeyにしたDictionaryって出来ないのかなーと思っていて、
仕方なくnewなんかしてイケてないなーと思いながら実装していたのですが、
こっちの方が良さげなのでメモしておきたいと思います。

やり方はただSystem.Typeを使えば良かったのですね。という話でした。  
ということで下記のような感じで確認コードを書いて適当なオブジェクトに付けておきます。



```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class TypeMap : MonoBehaviour {

  static private Dictionary<System.Type, string> map = new Dictionary<System.Type, string>() {
    {typeof(TestA),  "This is A"},
    // {typeof(TestB),  "This is B"},
    {typeof(TestC),  "This is C"},
  };

  // Use this for initialization
  void Start () {
    log<TestA>();
    log<TestB>();
    log<TestC>();
  }

  // check
  private void log<T>() {
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
public class TestC {}
```

準備ができたら実行してみます。  
下図のような感じでclassをKeyにして、Valueを取得することができました。

{{< figure src="/images/posts/2017/03/typemap.png">}}

これだけだと何に使うのという感じなんですが、例えば指定のクラスに対して、
指定のメソッドを実行するとか、指定のリクエストをするとか...でしょうか？  
何にせよ、つっかえが取れた感じなので今後に活かせそうかなと思います！

何かに参考になれば幸いです。  
以上です。
