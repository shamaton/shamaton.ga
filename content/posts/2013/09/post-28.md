---
title: macでgitを導入するときにやっておくこと。
author: しゃまとん
date: 2013-09-03T16:16:22+00:00
url: /posts/28
featured_image: /images/posts/2016/05/git.png
is_comment_form_freeze:
  - on
comment_form_freeze_message:
  - ' '
categories:
  - git
  - プログラミング関連
  - メモ

---
お世話になります。  
しゃまとんです。

macで何か開発をしているときにGitを使って管理したいときに、見るメモというかリンク集です。

<!--more-->

■Gitコマンドを使えるように。  
流れとしては  
[MacPortsのインストール][1]  
↓  
[Xcodeのインストール][2]  
↓  
Xcode　⇒　Preferences　⇒　Downloads　⇒　Command Line Tools　⇒　install  
↓  
[Gitのインストール][3]  
↓  
Gitがコマンドラインで使える

■git-completionでGitコマンドをタブ補完させる  
下記サイトを参考に設定。findして、~/.bashrcに追記しておけばOK。  
[Gitコマンドをタブキーで補完できるようにする][4]

■git configでgit環境を色々便利にする。  
タブ補完されるので、いい感じだけど、さらに省略できる。  
あと、表示をカラーリングしたい。  
[git configの使い方][5]

■番外編：wgetコマンドを使いたい  
なんかmacはwgetコマンドが入ってないみたいなので、もし必要にならば下記サイトを参考にする。  
[Mac(Lion)に&#8221;wget&#8221;をインストールする][6]

■その後のこと  
git cloneとかpushとか。githubな事。  
[gitリモートリポジトリの作り方][5]

以上であります。

 [1]: http://weble.org/2010/06/17/macports
 [2]: http://www.cse.kyoto-su.ac.jp/~oomoto/lecture/program/tips/Xcode_install/
 [3]: http://weble.org/2011/02/14/git-mac-install
 [4]: http://mawatari.jp/archives/git-completion-bash
 [5]: http://transitive.info/article/git/command/config/
 [6]: http://rdstyle.cocolog-nifty.com/gm/2013/01/maclionwget-63f.html