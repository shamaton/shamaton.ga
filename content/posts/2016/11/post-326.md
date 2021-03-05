---
title: '[Docker] docker.qcow2のサイズを小さくする回避策'
author: しゃまとん
date: 2016-11-02T15:08:52+00:00
url: /posts/326
featured_image: /images/posts/2016/10/small_v-dark.png
is_comment_form_freeze:
  - on
comment_form_freeze_message:
  - ' '
categories:
  - docker
  - Mac
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

Docker for Macに関するちょっとしたメモです。  
現状のDockerではどうやら使っていると容量がどんどん大きくなってしまうようです。

容量を一旦クリアにするには、ファイルを削除してしまうのがシンプルな解決方法みたいです。

[Docker for macでrmやrmiで消しまくってもストレージが開放されない][1]

今あるデータを消したくないという場合は下記の方法が回避策みたいです。  
対応の際は自己責任でお願いします。

docker.qcow2というファイルに処置をするのですが、qemu-imgというコマンド使うようです。
qemu系のコマンドは最初から入ってないので、インストールします。

```shell
brew install qemu
```

下記のコマンドを実行する場合はDockerに関連するアプリケーションは停止しておいた方がよいと思います。

```shell
cd ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux
# ファイルリネーム
mv Docker.qcow2 Docker.qcow2.original
# 変換
qemu-img convert -O qcow2 Docker.qcow2.original Docker.qcow2
# 元ファイルを削除
rm Docker.qcow2.original
```

自分の環境では15G程あったのが11Gになりました。あまり小さくならなかったのですが、
参考先にもあるように環境によっては小さくなるのではないでしょうか。

以上です。

■ 参考
{{< blogcard url="https://github.com/docker/for-mac/issues/371" >}}

 [1]: http://qiita.com/junkjunctions/items/ad971fd84fb8c30816d6