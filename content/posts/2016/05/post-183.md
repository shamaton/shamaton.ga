---
title: '[golang]pre-commit-hookでビルドチェックしてみる'
author: しゃまとん
type: post
date: 2016-05-14T12:48:14+00:00
url: /posts/183
featured_image: /images/posts/2016/05/git.png
categories:
  - git
  - go
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

gitでプロジェクトやコードを管理する場合、commit時にhookをいれて変なcommitが混ざらないようにすることができます。pre-commit-hookという機能なのですが、phpに関しての設定記事はよく見るのですが、goでもやりたいなーということで考えてみました。

やはりやりたいのはコードのビルドが通るかなので、、コンパイルチェック的なものをしたいのですが、そういうのはスクリプト言語のチェック機構なので普通にビルドして確認することにしてみます。（phpならphp -l、perlならperl -cでしょうか）

下記がチェックスクリプトです。  
通常のビルドをしているのですが、問題なく成功したときに実行ファイルができてしまうので、削除処理をしてから最後の手続きをしています。

{リポジトリトップ}/.git/hooks/pre-commitあたりに記載するとcommit時に実行してくれます。（今回はtouch pre-commitで作成しました）

<pre class="brush: text; gutter: true">#!/bin/sh

HAS_ERROR=0
TMP="___build_check"

# go file num
NUM=`git diff --cached --name-status | cut -f2 | grep &#039;.go$&#039; | wc -l`

# build check
if [ ${NUM} -ne 0 ]; then
    echo "build checking..."
    for f in `git diff --cached --name-status | cut -f2 | grep &#039;.go$&#039;`;
    do
        if ! go build -o ${TMP} ${f}; then
            HAS_ERROR=1
        fi
    done
fi

# delete test build file
if [ -e ${TMP} ]; then
    rm ${TMP}
fi

# has error?
if [ ${HAS_ERROR} -ne 0 ]; then
    echo "build error found!!"
    exit 1
fi

# 正常終了
exit 0</pre>

実行するとこんな感じ。（わかりづらいですが&#8230;）

<pre class="brush: bash; gutter: true">$ git commit
build checking...

# abortしました
Aborting commit due to empty commit message.</pre>

今回の場合はコミット対象のものを単一ファイルでビルドチェックをしていくのですが、記載をかえれば大きなアプリケーションにもチェックを入れられると思います。

以上です。

■参考  
<a href="http://blog.manaten.net/entry/645" target="_blank">gitのpre-commit hookを使って、綺麗なPHPファイルしかコミットできないようにする<br /> </a><a href="http://www.unitrust.co.jp/%E3%81%8B%E3%82%86%E3%81%84%E3%81%A8%E3%81%93%E3%82%8D%E3%81%AB%E6%89%8B%E3%81%8C%E5%B1%8A%E3%81%8Fgit%E3%83%95%E3%83%83%E3%82%AF%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%97%E3%83%88%E3%82%AF%E3%83%A9/" target="_blank">かゆいところに手が届くgitフックスクリプト(クライアントサイド編)</a>