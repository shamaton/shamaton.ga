---
title: '[Jenkins] Jenkinsからaws-cliを使えるようにするメモ'
author: しゃまとん
date: 2017-03-15T15:05:05+00:00
url: /posts/314
featured_image: /images/posts/2016/08/headshot.png
categories:
  - メモ

---
お世話になっております。  
しゃまとんです。

macでJenkinsを使う際のちょっとしたメモです。Jenkinsでawsの操作を実行したいなぁと思い、
macにaws-cliをインストールしました。そこでJenkinsにジョブを作成し、シェルの実行タスクからawsコマンドを利用してみると

```shell
/var/folders/zh/k0xxxxxxxxxxxxxxxxxxxxx/T/hudsonxxxxxxxxxxxxx.sh: line **: aws: command not found
```

となり、awsコマンドが使えませんでした。  
通常のターミナルからは使えるのになぜだろうと思い調べていたら、案の定パスが通っていなかったのが原因でした。

aws-cliはインストールすると、`/usr/local/bin`にコマンドが配置されています。

```shell
ls /usr/local/bin/aws*
/usr/local/bin/aws                  /usr/local/bin/aws.cmd              /usr/local/bin/aws_completer        /usr/local/bin/aws_zsh_completer.sh
```

ということで、Jenkins側の設定を変えてやります。

[Jenkinsの管理] → [システムの設定] を選択し、グローバルプロパティにPATHを追加しました。

{{< figure src="/images/posts/2016/10/set_path.png" >}}

これで、再度確認してみると

{{< figure src="/images/posts/2016/10/aws_help.png" >}}

実行することができました。Jenkinsからシェルを実行すると環境変数が違っていたりするのでちょっと配慮が必要ですね。

あとシェルの実行で頭に#!/bin/shを書くとエラーが発生したときに
途中でexitしないみたいなのでコピペする際は注意したほうが良さそうですね。

{{< blogcard url="http://qiita.com/mechamogera/items/f689b95670127d5bf046">}}

以上です。
