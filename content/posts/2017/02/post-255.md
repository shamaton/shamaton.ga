---
title: '[Unity/C#] List<T>.ConvertAllの実行速度について'
author: しゃまとん
type: post
date: 2017-02-07T13:56:54+00:00
url: /posts/255
featured_image: /images/posts/2016/08/a252291db008f1ed7cbf0d54c4b39950.png
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

開発のしているとListをよく使うのですが、色々と実装している中でリストをまとめて別の型に置き換えて使いたいなーという事がありました（親クラスのリストを小クラスで利用したいとか？）

調べているとどうやらListにはまとめて変換して、そのListを返してくれる[ConvertAll][1]という機能が用意されているようで便利そうだと思い利用しております。

ただなんとなくですが、Allとか書いてるし処理重そうだな〜、あんまり使わない方がいいかな・・・とちょっと気になったので、実行速度を簡単に調べてみました。

検証コードはこんな感じ。List<float>をList<int>にしてます。  
要素を10、100、1000、10000、100000と変えて実行してみました。

<pre class="brush: csharp; gutter: true">using UnityEngine;
using System.Collections;
using System.Collections.Generic;

public class ConvTest : MonoBehaviour {

  private List&lt;float&gt; list = new List&lt;float&gt;();

  // Use this for initialization
  void Start () {
    for (int i = 0; i &lt; 100000; i++) {
      list.Add(Random.Range(1f, 100000f));
    }
    StartCoroutine(check());
  }

  private IEnumerator check() {

    int count = 0;
    while (count++ &lt; 5) {
      float startTime = Time.realtimeSinceStartup;
      // all convert
      list.ConvertAll(content =&gt; (int)content);
      float msec = (Time.realtimeSinceStartup - startTime) * 1000f;
      Debug.Log(msec + " msec");
      yield return new WaitForSeconds(1f);
    }

  }
}</pre>

実行結果のまとめはこんな感じでした。  
なぜか2回めからのConvertが早くなっています。なぜでしょうか・・・？ただ、要素数が増えるとその辺もなくなってしまうようです。

|     | 10          | 100        | 1000       | 10000     | 100000   |
| --- | ----------- | ---------- | ---------- | --------- | -------- |
| 1回目 | 0.3820658   | 0.4429817  | 0.3560781  | 0.8010864 | 4.02391  |
| 2回目 | 0.003099442 | 0.01096725 | 0.0808239  | 0.7491112 | 6.491899 |
| 3回目 | 0.002861023 | 0.01502037 | 0.1180172  | 0.4088879 | 8.046865 |
| 4回目 | 0.003814697 | 0.01096725 | 0.08487701 | 0.7128716 | 7.349968 |
| 5回目 | 0.004291534 | 0.01192093 | 0.08678436 | 0.7219315 | 7.339954 |

※単位はmsec

現状の使い方はそんなに要素数は多くないので、2回めから早くなる恩恵にあやかりつつ、使っていこうかなと思います。

以上です。

 [1]: https://msdn.microsoft.com/ja-jp/library/73fe8cwf(v=vs.110).aspx