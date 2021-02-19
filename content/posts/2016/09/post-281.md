---
title: '[Unity] AnimationClip内のSpriteのプロパティ名を変更したい'
author: しゃまとん
type: post
date: 2016-09-05T12:54:54+00:00
url: /archives/281
featured_image: /wp-content/uploads/2016/09/not_sprite.gif
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

悩んでおりましたが良い解決方法は見つからず。。  
UnityでAnimationClipを作成して、結構な数を作成つくったあとにプロパティ名を変更したい・・・ということがあると思います。

例えばTransformとかのプロパティに関して変更するときはエディタ上から修正することが可能です。（画像では参照ができなくなりMissingになっています）

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2016/09/enable_change.gif" alt="enable_change" width="291" height="120" class="aligncenter size-full wp-image-286" />][1]

同じようにSpriteも変更しようとすると、変更されずコンソールにはエラーが表示されます。

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2016/09/not_sprite.gif" alt="not_sprite" width="291" height="120" class="aligncenter size-full wp-image-287" />][2]

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2016/09/sprite_error.png" alt="sprite_error" width="343" height="118" class="aligncenter size-full wp-image-285" />][3]

サクッと変更できないだと・・・  
ということで、巷で公開してくださっているエディタ拡張などで変更を試みましたが、解決には至りませんでした。

ということで、色々調べた挙句の対処法は「Animファイルを直接編集する」になりました。  
では以下、手順です。（Unityは5.4.0f3です）

■ シリアライズの形式をforce textにする  
Menu → Edit → Project Settings → EditorでAsset Serializationを下記のようにします。  
こうすることで、Animファイルがテキストで見えるようになります。

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2016/09/force_text_setting.png" alt="force_text_setting" width="380" height="503" class="aligncenter size-full wp-image-283" />][4]

■ 変更したいファイルをテキストエディタで開く  
（ファイルを開く前に念のためにUnity閉じておく方がいいかもしれません。）  
好きなテキストエディタでファイルを開きます。その中からpathを探してみてください。階層構造でプロパティ名が記載されていると思います。これを参照させたい名称に変更します。

<p style="text-align: center;">
  <a href="https://shamaton.orz.hm/blog/wp-content/uploads/2016/09/missing.png"><img src="https://shamaton.orz.hm/blog/wp-content/uploads/2016/09/missing.png" alt="missing" width="293" height="99" class="aligncenter size-full wp-image-288" /></a>
</p>

<p style="text-align: left;">
  画面は変更のイメージで、実際はSprite_Object_Changeのみになります。
</p>

<p style="text-align: center;">
  <a href="https://shamaton.orz.hm/blog/wp-content/uploads/2016/09/text_change.png"><img src="https://shamaton.orz.hm/blog/wp-content/uploads/2016/09/text_change.png" alt="text_change" width="315" height="60" class="aligncenter size-full wp-image-289" /></a>
</p>

これでUnityを再度立ち上げます。これで確認してみると・・・

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2016/09/ok.png" alt="ok" width="298" height="102" class="aligncenter size-full wp-image-290" />][5]

プロパティ名が変更され、Missingが解決されます＾＾  
あとは1つずつファイルを開いて・・・って、数が多い場合はつらいので一括置換したい場合はターミナルでsedコマンドを使うといいかなと思います。

下記のような感じでxargsを組み合わせて使います。  
Macではsedの挙動が少し違うらしく-iのあとに””を記載しないと動作しないみたいです。

<pre class="lang:sh decode:true ">find {対象のファイル} -type f | xargs sed -i ""  "s/{置換したい文字}/{置換後の文字}/g"</pre>

実行すると、ファイルの中身が変更されるので心配な場合は-iをつけずに出力して確認してみるといいかもです。同様にUnityは閉じた状態でやってみてください。

Unityのバージョンアップでエディタ上から変更できるようになるといいですね。。  
以上です。

&nbsp;

■ 参考  
[Unity で Animation を作り込んだ後の Hierarchy 変更][6]  
[BSD(Mac) で find xargs sed を使う際の注意点][7]

 [1]: https://shamaton.orz.hm/blog/wp-content/uploads/2016/09/enable_change.gif
 [2]: https://shamaton.orz.hm/blog/wp-content/uploads/2016/09/not_sprite.gif
 [3]: https://shamaton.orz.hm/blog/wp-content/uploads/2016/09/sprite_error.png
 [4]: https://shamaton.orz.hm/blog/wp-content/uploads/2016/09/force_text_setting.png
 [5]: https://shamaton.orz.hm/blog/wp-content/uploads/2016/09/ok.png
 [6]: http://qiita.com/nekobako/items/b647a701b6070d1ca872
 [7]: http://tkuchiki.hatenablog.com/entry/2013/02/27/130114