---
title: '[Unity] あるクラスに指定のInterfaceがあるか確認する'
author: しゃまとん
type: post
date: 2017-09-15T15:29:18+00:00
url: /archives/396
featured_image: /wp-content/uploads/2017/04/csharp.png
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

c#でジェネリックを使う場合、whereを利用して指定の条件のものしか使用できないようにしますよね。下記みたいな。

<pre class="lang:c# decode:true">public class Child&lt;T&gt; : Parent where T : SomeInterface {...}
public class Hoge : Child&lt;Fuga&gt; {...}</pre>

それで、似たような感じでSystem.Typeな変数に対して、指定の型を含んでいるか確認できないかなーと思い、ちょっと調べることになったのでメモしておきます。  
（今回は指定のInterfaceがあるか確認したかった）

結果的には含まれていてほしいクラスやインターフェース越しにIsAssignableFromというメソッドを使うと判断できるみたいです。  
使い方はこんな感じで。

<pre class="lang:c# decode:true">// HogeがSomeInterfaceを持っていたらtrue

// 方法：1
typeof(SomeInterface).IsAssignableFrom(typeof(Hoge))

// 方法：2
typeof(Hoge).GetInterfaces().Contains(typeof(SomeInterface))</pre>

上記の場合だとHogeクラスのTypeから何か取得するかなーと思っていのですが、逆みたいでした。Hogeクラスからも判定できるけど、IsAssignableのほうがベターっぽいですね。

このやり方を使うと、実行時のエラーになるのでビルドだけ通ったからOKではないので注意が必要ですね。ジェネリックはビルド時にエラーになるけど。  
（常に実行して確認しろ！というのは至極当然ですが&#8230;）

今日も勉強になりました。  
以上です。

■参考  
<a href="http://qiita.com/chocolamint/items/8b1be93359a7a26fe7c3" target="_blank" rel="noopener">Type.IsAssignableFrom はどっちがどっち？</a>  
<a href="http://stackoverflow.com/questions/4963160/how-to-determine-if-a-type-implements-an-interface-with-c-sharp-reflection" target="_blank" rel="noopener">How to determine if a type implements an interface with C# reflection</a>