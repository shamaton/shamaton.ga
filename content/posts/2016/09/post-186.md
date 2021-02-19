---
title: '[Unity] Colliderの比較するのは何がいいのか'
author: しゃまとん
type: post
date: 2016-09-25T13:13:28+00:00
url: /archives/186
featured_image: /wp-content/uploads/2016/05/space-92348_640.jpg
is_comment_form_freeze:
  - on
comment_form_freeze_message:
  - ' '
categories:
  - unity
  - プログラミング関連

---
お世話になっております  
しゃまとんです。

最近、当たり判定について考えていたのですが、Colliderを受け取ったら何を対象に比較するのがいいのだろう  
・名前  
・タグ  
・ID

色々な情報から判定が出来そうなのですが、個人的に  
・文字より数値のほうが比較が早い  
・常にユニークな値  
がいいなぁと思っていたので、何かユニークなIDはないのか！と探すと

GetInstanceIDというものがあったのですね。使ったことなかった。  
Unityで生成されているオブジェクトは必ずIDがついていて、ユニークな値になっているそうです。



じゃあ、これでいい！

<pre class="brush: csharp; gutter: false">collider.GetInstanceId()</pre>

とすると、ColliderのIDになってしまうらしく、判定がうまくいかないので注意です。  
もしオブジェクトとの比較ならば

<pre class="brush: csharp; gutter: false">collider.gameObject.GetInstanceID()</pre>

とする必要があるようです。gameObjectを参照する形になるのが嫌だなーと思ったのですが  
衝突が大量に発生しないようならこれでもいいかなと落ち着きました。なんかパフォーマンス気にしすぎな気もするのですが&#8230;

簡単にまとめるとこんな感じに。

<pre class="brush: csharp; gutter: true">using UnityEngine;
using System.Collections;

public class Attack : MonoBehaviour {

  private int enemyId = 0;

  public void SetEnemyId(int id) {
    enemyId = id;
  }

  void OnTriggerEnter2D(Collider2D c) {
    if (c.gameObject.GetInstanceID() == enemyId) {
      // 何かしらの処理
    }
  }
}</pre>

複数のオブジェクトを対象にするときはDictionaryとか使って何が当たっているのか判断したらよいかなと思っています。  
開発が進んできて処理が重いのに悩むのなるべく避けたいですよね&#8230;  
以上です。