---
title: '[Unity] あるクラスに指定のInterfaceがあるか確認する'
author: しゃまとん
date: 2017-09-15T15:29:18+00:00
url: /posts/396
featured_image: /images/posts/2017/04/csharp.png
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

```csharp
public class Child<T> : Parent where T : SomeInterface {...}
public class Hoge : Child<Fuga> {...}
```

それで、似たような感じでSystem.Typeな変数に対して、指定の型を含んでいるか確認できないかなーと思い、ちょっと調べることになったのでメモしておきます。  
（今回は指定のInterfaceがあるか確認したかった）

結果的には含まれていてほしいクラスやインターフェース越しに`IsAssignableFrom`というメソッドを使うと判断できるみたいです。  
使い方はこんな感じで。

```csharp
/ HogeがSomeInterfaceを持っていたらtrue

// 方法：1
typeof(SomeInterface).IsAssignableFrom(typeof(Hoge))

// 方法：2
typeof(Hoge).GetInterfaces().Contains(typeof(SomeInterface))
```

上記の場合だとHogeクラスのTypeから何か取得するかなーと思っていのですが、逆みたいでした。
Hogeクラスからも判定できるけど、`IsAssignable`のほうがベターっぽいですね。

このやり方を使うと、実行時のエラーになるのでビルドだけ通ったからOKではないので注意が必要ですね。
ジェネリックはビルド時にエラーになるけど。  
（常に実行して確認しろ！というのは至極当然ですが...）

今日も勉強になりました。  
以上です。

■参考
{{< blogcard url="http://qiita.com/chocolamint/items/8b1be93359a7a26fe7c3" >}}
{{< blogcard url="http://stackoverflow.com/questions/4963160/how-to-determine-if-a-type-implements-an-interface-with-c-sharp-reflection" >}}