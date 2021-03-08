---
title: '[golang]pre-commit-hookでビルドチェックしてみる'
author: しゃまとん
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

gitでプロジェクトやコードを管理する場合、commit時にhookをいれて変なcommitが混ざらないようにすることができます。
pre-commit-hookという機能なのですが、phpに関しての設定記事はよく見るのですが、goでもやりたいなーということで考えてみました。

やはりやりたいのはコードのビルドが通るかなので、、コンパイルチェック的なものをしたいのですが、
そういうのはスクリプト言語のチェック機構なので普通にビルドして確認することにしてみます。（phpならphp -l、perlならperl -cでしょうか）

下記がチェックスクリプトです。  
通常のビルドをしているのですが、問題なく成功したときに実行ファイルができてしまうので、削除処理をしてから最後の手続きをしています。

{リポジトリトップ}/.git/hooks/pre-commitあたりに記載するとcommit時に実行してくれます。
（今回はtouch pre-commitで作成しました）

```shell
#!/bin/sh

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
exit 0
```

実行するとこんな感じ。（わかりづらいですが...）

```shell
 git commit
build checking...

# abortしました
Aborting commit due to empty commit message.
```

今回の場合はコミット対象のものを単一ファイルでビルドチェックをしていくのですが、
記載をかえれば大きなアプリケーションにもチェックを入れられると思います。

以上です。

■参考
{{< blogcard url="http://blog.manaten.net/entry/645" >}}