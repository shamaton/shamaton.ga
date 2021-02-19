---
title: '[Unity] テキストボックスで何行表示されているかを知る'
author: しゃまとん
type: post
date: 2017-11-14T14:55:17+00:00
url: /archives/380
featured_image: /wp-content/uploads/2017/03/line_count.gif
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

決められた範囲でテキストを入力させたいなーと思って、色々と試行錯誤してました。InputFieldでOnChangedValueを監視して、入力内容をReplaceしたり、指定文字列をCountしたり&#8230;とやっていたのですが、なかなかうまくいかず悩んでました。

要は画面に表示されているテキストボックスの範囲内だけに入力をさせたいみたいない感じです。何行まで表示が可能かわかっていて、現在何行表示されてるかわかればいいなぁ・・・というので調べてみるとtextコンポーネント内にプロパティがありました。

[textGenerator][1]とはレンダリング用のテキストを生成するために使用されるクラスでその中のlineCountというプロパティにアクセスすると表示上何行になっているか把握できます。

<pre class="lang:c# decode:true">inputField.textComponent.cachedTextGenerator.lineCount</pre>

試しに入力しながら表示させてみると&#8230;

<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2017/03/line_count.gif" alt="" width="202" height="308" class="aligncenter size-full wp-image-385" /> 

こんな感じで折り返し、改行に対応して表示上の行数が把握できます。  
そんなに用途無いかもですが、参考になれば幸いです。確認用のコードは簡単ですが、一応載せておきます。

<pre class="lang:c# decode:true ">using UnityEngine;
using UnityEngine.UI;
using System.Collections;

public class TestScript : MonoBehaviour {

  public InputField inputField;
  public Text text;

  // Use this for initialization
  void Start () {

  }

  // Update is called once per frame
  void Update () {
    text.text = inputField.textComponent.cachedTextGenerator.lineCount.ToString() + "行です ";
  }
}
</pre>

以上です。

■ 参考  
<http://answers.unity3d.com/questions/784199/how-to-get-the-current-best-fit-size-of-a-text-com.html>

 [1]: https://docs.unity3d.com/ja/current/ScriptReference/TextGenerator.html