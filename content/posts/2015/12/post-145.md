---
title: '[golang]標準入力を受け取りつつ、システムコマンド(mysqlなど)を実行する'
author: しゃまとん
type: post
date: 2015-12-06T09:47:34+00:00
url: /posts/145
featured_image: /images/posts/2015/11/gopher.jpg
categories:
  - go
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

最近、golangに触れる機会が多くなっております。Gopherかわいい(*´∀｀)

さて、goでDBを初期化するためのバッチファイルを書いていたのですが、

<pre class="brush: bash; gutter: true">$ mysql -uroot -ppassword dbname &gt; hoge.sql</pre>

上記のようにgoからシステムコマンドを実行したかったのですが、もろもろ調査して標準入力含めて、実行できた際のメモです。

はじめは、上記のコマンドをそのまま、exec.Commandに渡してやろうとしたのですが、&#8221;<&#8220;, &#8220;>&#8221;のような特殊文字は渡せないみたいで、StdinPipeを使うことで入力を渡すことができるようです。

下記がサンプルになります。

<pre class="brush: text; gutter: true">package main

import (
	"fmt"
	"io/ioutil"
	"os/exec"
)

func main() {
	user := "root"
	password := "password"
	dbName := "dbname"

	path, err := exec.LookPath("mysql")
	cmd := exec.Command(path, "-u"+user, "-p"+password, dbName)

	// 入力SQLファイル
	readFile := "/home/shamaton/test.sql"
	input, err := ioutil.ReadFile(readFile)
	if err != nil {
		panic(err)
	}
	stdin, _ := cmd.StdinPipe()
	stdin.Write(input)
	stdin.Close()

	var stdErr, stdOut bytes.Buffer
	cmd.Stderr = &stdErr
	cmd.Stdout = &stdOut

	// exec
	err = cmd.Run()
	if err != nil {
		fmt.Println(stdErr.String())
		panic(err)
	}
	fmt.Println(stdOut.String())
}</pre>

paswwordに関しては-pにくっつけておかないとエラーになるかと思います。  
mysqlでの例ですが、変数等は適宜置き換えたらいいのではないかと思います。

以上です。

■参考  
<a href="http://tkuchiki.hatenablog.com/entry/2014/11/10/123447" target="_blank">golang で外部コマンドを実行して標準出力を取得する</a>