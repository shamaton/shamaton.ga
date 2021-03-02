---
title: '[Jenkins] Jenkinsからaws-cliを使えるようにするメモ'
author: しゃまとん
type: post
date: 2017-03-15T15:05:05+00:00
url: /posts/314
featured_image: /images/posts/2016/08/headshot.png
categories:
  - メモ

---
お世話になっております。  
しゃまとんです。

macでJenkinsを使う際のちょっとしたメモです。Jenkinsでawsの操作を実行したいなぁと思い、macにaws-cliをインストールしました。そこでJenkinsにジョブを作成し、シェルの実行タスクからawsコマンドを利用してみると

<pre class="lang:default decode:true ">/var/folders/zh/k0xxxxxxxxxxxxxxxxxxxxx/T/hudsonxxxxxxxxxxxxx.sh: line **: aws: command not found</pre>

となり、awsコマンドが使えませんでした。  
通常のターミナルからは使えるのになぜだろうと思い調べていたら、案の定パスが通っていなかったのが原因でした。

aws-cliはインストールすると、/usr/local/binにコマンドが配置されています。

<pre class="lang:sh decode:true ">$ ls /usr/local/bin/aws*
/usr/local/bin/aws                  /usr/local/bin/aws.cmd              /usr/local/bin/aws_completer        /usr/local/bin/aws_zsh_completer.sh
</pre>

ということで、Jenkins側の設定を変えてやります。

[Jenkinsの管理] → [システムの設定] を選択し、グローバルプロパティにPATHを追加しました。

[<img src="https://shamaton.orz.hm/blog/images/posts/2016/10/set_path.png" alt="set_path" width="990" height="205" class="aligncenter size-full wp-image-315" />][1]

これで、再度確認してみると

[<img src="https://shamaton.orz.hm/blog/images/posts/2016/10/aws_help.png" alt="aws_help" width="1320" height="704" class="aligncenter size-full wp-image-316" />][2]

実行することができました。Jenkinsからシェルを実行すると環境変数が違っていたりするのでちょっと配慮が必要ですね。

あとシェルの実行で頭に#!/bin/shを書くとエラーが発生したときに途中でexitしないみたいなのでコピペする際は注意したほうが良さそうですね。

[Jenkinsのシェルの実行について][3]

以上です。

 [1]: https://shamaton.orz.hm/blog/images/posts/2016/10/set_path.png
 [2]: https://shamaton.orz.hm/blog/images/posts/2016/10/aws_help.png
 [3]: http://qiita.com/mechamogera/items/f689b95670127d5bf046