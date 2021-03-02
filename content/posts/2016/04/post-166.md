---
title: '[Unity]複数のuGUIを円運動させる'
author: しゃまとん
type: post
date: 2016-04-09T04:34:25+00:00
url: /posts/166
featured_image: /images/posts/2016/04/circle.gif
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

UnityのuGUIを使って、画面上で円運動させたいなーと思いやってみました。  
Mathfにあるsin/cos関数をx，yに当てはめることで回転させられます。

sin/cos関数にはラジアンを渡します。MathfのDeg2Radを使うと角度を簡単にラジアンに変換できます。

<pre class="brush: csharp; gutter: true">float rad = angle * Mathf.Deg2Rad;

float x = Mathf.Cos(rad) * r;
float y = Mathf.Sin(rad) * r;</pre>

今回は複数のUIオブジェクトを均等に配置したかったので、ベースオブジェクトを用意してスクリプト上から複製して、座標計算して設定しています。  
半径はスクリプトで定義してもいいですし、ベースオブジェクトを0度などわかりやすい位置に置いて利用してもいいと思います。

<img src="https://shamaton.orz.hm/blog/images/posts/2016/04/UI_inspector.png" alt="UI_inspector" width="506" height="262" class="size-full wp-image-168 aligncenter" /> 

スクリプトは上記画像のUIオブジェクトにつけて、Baseを参照させます。  
コードはこんな感じです。

<pre class="brush: actionscript3; gutter: true">using UnityEngine;
using UnityEngine.UI;
using System.Collections;
using System.Collections.Generic;

public class UICircle : MonoBehaviour {
  // 回転速度
  private const float speed = 2f;
  // 生成数
  private const int num = 5;

  [SerializeField]
  private Image imgBase; // ベース
  private float r;       // 半径

  // 回転に利用する
  private List&lt;RectTransform&gt; transes = new List&lt;RectTransform&gt;();
  private List&lt;float&gt; startRads = new List&lt;float&gt;();

  // cache
  private Transform myTrans;

  void Start() {
    // キャッシュする
    myTrans = transform;

    // 半径を取得
    r = imgBase.rectTransform.localPosition.y;

    // ベースから生成
    GameObject objBase = imgBase.gameObject;
    for (int i = 0; i &lt; num; i++) {
      GameObject obj = Instantiate(objBase);
      obj.transform.SetParent(myTrans, false);

      RectTransform trans = obj.GetComponent&lt;RectTransform&gt;();

      // 角度
      float angle = i * (360 / num) + 90f;
      // ラジアン
      float rad = angle * Mathf.Deg2Rad;
      // 座標変換
      float x = Mathf.Cos(rad) * r;
      float y = Mathf.Sin(rad) * r;

      // 初期位置
      trans.anchoredPosition = new Vector2(x, y);

      // update用に取得
      transes.Add(trans);
      startRads.Add(rad);
    }

    // ベースを消す
    objBase.SetActive(false);
  }

  void Update () {
    // 回転処理
    for (int i = 0; i &lt; transes.Count; i++) {
      float nowRad = Time.time * speed + startRads[i];
      float x = Mathf.Cos(nowRad) * r;
      float y = Mathf.Sin(nowRad) * r;

      transes[i].anchoredPosition = new Vector2(x, y);
    }
  }
}</pre>

実行するとくるくると回ります。

<img src="https://shamaton.orz.hm/blog/images/posts/2016/04/circle.gif" alt="circle" width="222" height="212" class="size-full wp-image-169 aligncenter" /> 

以上です。

■ 参考  
<a href="http://matudozer.blog.fc2.com/blog-entry-21.html" target="_blank">三角関数を使って、振り子のような運動や円運動をさせる</a>