---
title: '[Unity] AnimationClip内のSpriteのプロパティ名を変更したい'
author: しゃまとん
date: 2016-09-05T12:54:54+00:00
url: /posts/281
featured_image: /images/posts/2016/09/not_sprite.gif
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

悩んでおりましたが良い解決方法は見つからず。。  
UnityでAnimationClipを作成して、結構な数を作成つくったあとにプロパティ名を変更したい・・・ということがあると思います。

例えばTransformとかのプロパティに関して変更するときはエディタ上から修正することが可能です。（画像では参照ができなくなりMissingになっています）

{{< figure src="/images/posts/2016/09/enable_change.gif" >}}

同じようにSpriteも変更しようとすると、変更されずコンソールにはエラーが表示されます。

{{< figure src="/images/posts/2016/09/not_sprite.gif" >}}

{{< figure src="/images/posts/2016/09/sprite_error.png" >}}

サクッと変更できないだと・・・  
ということで、巷で公開してくださっているエディタ拡張などで変更を試みましたが、解決には至りませんでした。

ということで、色々調べた挙句の対処法は「Animファイルを直接編集する」になりました。  
では以下、手順です。（Unityは5.4.0f3です）

■ シリアライズの形式をforce textにする  
Menu → Edit → Project Settings → EditorでAsset Serializationを下記のようにします。  
こうすることで、Animファイルがテキストで見えるようになります。

{{< figure src="/images/posts/2016/09/force_text_setting.png" >}}

■ 変更したいファイルをテキストエディタで開く  
（ファイルを開く前に念のためにUnity閉じておく方がいいかもしれません。）  
好きなテキストエディタでファイルを開きます。その中からpathを探してみてください。
階層構造でプロパティ名が記載されていると思います。これを参照させたい名称に変更します。

{{< figure src="/images/posts/2016/09/missing.png" >}}

画面は変更のイメージで、実際はSprite_Object_Changeのみになります。

{{< figure src="/images/posts/2016/09/text_change.png" >}}

これでUnityを再度立ち上げます。これで確認してみると・・・

{{< figure src="/images/posts/2016/09/ok.png" >}}

プロパティ名が変更され、Missingが解決されます＾＾  
あとは1つずつファイルを開いて・・・って、数が多い場合はつらいので一括置換したい場合はターミナルでsedコマンドを使うといいかなと思います。

下記のような感じでxargsを組み合わせて使います。  
Macではsedの挙動が少し違うらしく-iのあとに””を記載しないと動作しないみたいです。

```shell
find {対象のファイル} -type f | xargs sed -i ""  "s/{置換したい文字}/{置換後の文字}/g"
```

実行すると、ファイルの中身が変更されるので心配な場合は-iをつけずに出力して確認してみるといいかもです。
同様にUnityは閉じた状態でやってみてください。

Unityのバージョンアップでエディタ上から変更できるようになるといいですね。。  
以上です。

■ 参考

{{< blogcard url="http://qiita.com/nekobako/items/b647a701b6070d1ca872" >}}
{{< blogcard url="http://tkuchiki.hatenablog.com/entry/2013/02/27/130114" >}}