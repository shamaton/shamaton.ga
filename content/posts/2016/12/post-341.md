---
title: dplyのボタンつくってみた
author: しゃまとん
date: 2016-12-28T14:49:32+00:00
url: /posts/341
featured_image: /images/posts/2016/12/dply.png
is_comment_form_freeze:
  - on
comment_form_freeze_message:
  - ' '
categories:
  - Linux
  - Web関連
  - サーバー構築

---
お世話になっております。  
しゃまとんです。

[dply][1]という2時間無料で使えるVPSサービスがはじまったそうです。  
お金を払えば、長期間維持することが可能みたいですがとりあえずサーバで何か検証したい場合に使えそうですね。

dplyとはなんぞやについては下記を素敵に参照してくださると幸いです。

{{< blogcard url="https://uzulla.hateblo.jp/entry/2016/11/28/181949" >}}

自分もとりあえず触ってみた中で、使い捨てな感じでやっているとセットアップがやはりめんどくさいなということで、初期化スクリプト的なものを書く流れになったのですが、dplyではサーバのセットアップ手順を含めたボタンを作成して、共有できるそうです。

これ →[https://dply.co/button][2] 

ということで、ボタンをつくってみました。

![https://dply.co/b.svg](https://dply.co/b/GuQRnugy)

ボタンを押すとdplyに飛びます。locationやplanを設定しCREATE SERVERを押すと、以下の環境が作成されます。  
・CentOS6  
・go1.7.4/gitをインストール済  
・gitはcompletion対応させている（補完のみ）

雑なのですが、起動時に追加しているスクリプトは下記の通りです。

{{< gist shamaton c503424c3b5c7e89fc9bf6b77dba8205 >}}

上記のボタンを押したあとの画面に表示されているスクリプトと同じです。  
サーバが立ち上がると追加したスクリプトが処理されているので、ログを見て確認することができます。詳細は[こちら][5]にあります（英語）。

```shell
tail -f /var/log/cloud-init-output.log
```

現在（2016/12）は日本リージョンがないみたいなので、使うならシンガポールが一番近い感じですね。

サクサクっと使える環境が用意できてありがたいですね！  
以上です。

 [1]: https://dply.co
 [2]: https://dply.co/button
 [5]: https://dply.co/help/cloud-init