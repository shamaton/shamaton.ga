---
title: '[Unity] ジェネリックを使っている親クラス内のクラスはインスペクタに表示できない'
author: しゃまとん
type: post
date: 2016-04-29T05:59:15+00:00
url: /posts/180
featured_image: /images/posts/2016/04/test_child.png
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

タイトルの通りなのですが、クラス内に定義した内部クラスや構造体をインスペクタに表示して、色々調整した場合があると思います。

上記の場合[System.Serializable]と[[SerializableField]を利用すれば問題なく表示されます。  
これに加えて、親クラス（基底クラス）でジェネリックを利用しつつ内部クラスを定義して小クラスからインスペクタ上で操作したいと思い下記みたいな感じにしました。

<pre class="brush: csharp; gutter: true">using UnityEngine;
using System.Collections;

public class TestParent&lt;T&gt; : MonoBehaviour where T : MonoBehaviour {
  static public T instance;

  [System.Serializable]
  public struct TestInnerParent {
    public int   hoge;
    public float fuga;
  }

  [SerializeField]
  private TestInnerParent innerValue;

  // シングルトンにする処理...
}</pre>

<pre class="brush: csharp; gutter: true">using UnityEngine;
using System.Collections;

public class TestChild : TestParent&lt;TestChild&gt; {
  public int childHoge;
  public int childFuga;
}</pre>

こうすると表示されなくなってしまいました。

[![https://gyazo.com/df3eb5de6e8824aeaffe28673f552f4e][1]][2]

どうしても表示したい場合は内部クラスを外に出せば表示できるようです。

<pre class="brush: csharp; gutter: true">using UnityEngine;
using System.Collections;

public class TestParent&lt;T&gt; : MonoBehaviour where T : MonoBehaviour {
  static public T instance;

  [SerializeField]
  private TestInnerParent innerValue;

  // シングルトンにする処理...
}

[System.Serializable]
public struct TestInnerParent {
  public int   hoge;
  public float fuga;
}</pre>

これで表示されます。

[![https://gyazo.com/da588e9bbab7c83d47598c8b7fcc41e4][3]][4]

他の方も書かれていますが、ジェネリック周りはちとうまくいかないのですね。  
以上です。

■参考  
<a href="http://qiita.com/k_yanase/items/3bb59963a6f477f5a523" target="_blank">UnityのInspectorで変数を表示する方法まとめ</a>  
<a href="http://ftvoid.com/blog/post/732" target="_blank">[Unity] 自前のクラスをインスペクタから編集できるようにする<br /> </a>

 [1]: https://i.gyazo.com/df3eb5de6e8824aeaffe28673f552f4e.png
 [2]: https://gyazo.com/df3eb5de6e8824aeaffe28673f552f4e
 [3]: https://i.gyazo.com/da588e9bbab7c83d47598c8b7fcc41e4.png
 [4]: https://gyazo.com/da588e9bbab7c83d47598c8b7fcc41e4