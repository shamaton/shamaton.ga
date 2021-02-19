---
title: '[golang]interfaceに変換されたstructの値を比較する'
author: しゃまとん
type: post
date: 2015-11-14T15:42:25+00:00
url: /archives/138
featured_image: /wp-content/uploads/2015/11/gopher.jpg
categories:
  - go
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

完全にメモな感じです。  
goで汎用的に使えるメソッドを用意するとなると、interfaceやreflectを使う場面がでてきます。

試作していた部分で構造体の中身が同じかどうか比較したいことがあり、下記のようにすれば確認することができました。

やっていることとしては、Elemで要素を取得して、value.Interface()で値が取得できるので、それを比較しています。

<pre class="brush: text; gutter: true">type Hoge struct {
	Id int
}

func main() {
	a := new(Hoge)
	a.Id = 1
	b := *a
	IsEqual(a, &b) // true
}

func IsEqual(i1 Interface{}, i2 Interface{}) bool {
	e1 := reflect.ValueOf(i1).Elem()
	e2 := reflect.ValueOf(i2).Elem()
	if e1.Interface() == e2.Interface() {
		return true
	}
	return false
}</pre>

今回はポインタで構造体を渡しているケースなので、型次第ではinterfaceに至るまでの過程が変わってくると思いますが、基本的は上記のような形で大丈夫だと思います。

以上です。