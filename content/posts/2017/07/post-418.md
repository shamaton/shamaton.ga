---
title: '[Unity] UnityでGit LFSを使う時の設定'
author: しゃまとん
date: 2017-07-01T16:13:52+00:00
url: /posts/418
featured_image: /images/posts/2016/05/git.png
categories:
  - git
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

なかなか個人制作レベルだとgitを使うぐらいになりますが、例えばエクセルデータだったり、
大きなサイズのファイルもgitで管理したい！ってなることが（多分）あります（よね）。

ただgitの性質的に大きなサイズのバイナリファイルを扱うのが苦手なようで、
ソースコードと同じ様に追加しているとリポジトリのサイズがえらいことになってるなんてこともあるかもしれません。


そんなのを解決するのがGit LFSというやつです。  
Large File Storageの略です。わかりやすいですね。

{{< blogcard url="https://git-lfs.github.com/" >}}
{{< slideshare sgNg3SCqXPQ49s >}}

使うにはちょっとの準備だけして、LFSを使いたいリポジトリに`.gitattributes`を置けばOK。

まず使えるようにするには下記を実行。

```shell
# mac
brew install git-lfs
git lfs install

# windows
# ここからダウンロード
https://git-lfs.github.com/

# 確認
$ git lfs version
git-lfs/2.2.0 (GitHub; darwin amd64; go 1.8.3)
```

そしてリポジトリではgitattributesを配置しておきます。  
lfsのコマンドもあって、登録もできます。

```shell
git lfs track "*.psd"
```

これでpsdファイルがLFSファイルになります。  
以降の操作はgitをいつも通り操作するだけです！

一点だけ注意しておくとcloneだけは下記のコマンドを使うのがよいです。

```shell
git lfs clone ${REPOSITORY_URL}
```

あと、.gitattributesのテンプレ的なのないのかなーと思ったので、[こちら][1]を参考にすると良さそう！

```text
# Disable EOL conversions by default
* -text

## Unity ##

*.cs diff=csharp text
*.cginc text
*.shader text

*.mat merge=unityyamlmerge
*.anim merge=unityyamlmerge
*.unity merge=unityyamlmerge
*.prefab merge=unityyamlmerge
*.physicsMaterial2D merge=unityyamlmerge
*.physicsMaterial merge=unityyamlmerge
*.asset merge=unityyamlmerge
*.meta merge=unityyamlmerge
*.controller merge=unityyamlmerge


## git-lfs ##

#Image
*.jpg filter=lfs diff=lfs merge=lfs
*.jpeg filter=lfs diff=lfs merge=lfs
*.png filter=lfs diff=lfs merge=lfs
*.gif filter=lfs diff=lfs merge=lfs
*.psd filter=lfs diff=lfs merge=lfs
*.ai filter=lfs diff=lfs merge=lfs

#Audio
*.mp3 filter=lfs diff=lfs merge=lfs
*.wav filter=lfs diff=lfs merge=lfs
*.ogg filter=lfs diff=lfs merge=lfs

#Video
*.mp4 filter=lfs diff=lfs merge=lfs
*.mov filter=lfs diff=lfs merge=lfs

#3D Object
*.FBX filter=lfs diff=lfs merge=lfs
*.fbx filter=lfs diff=lfs merge=lfs
*.blend filter=lfs diff=lfs merge=lfs
*.obj filter=lfs diff=lfs merge=lfs

#ETC
*.a filter=lfs diff=lfs merge=lfs
*.exr filter=lfs diff=lfs merge=lfs
*.tga filter=lfs diff=lfs merge=lfs
*.pdf filter=lfs diff=lfs merge=lfs
*.zip filter=lfs diff=lfs merge=lfs
*.dll filter=lfs diff=lfs merge=lfs
*.unitypackage filter=lfs diff=lfs merge=lfs
*.aif filter=lfs diff=lfs merge=lfs
*.ttf filter=lfs diff=lfs merge=lfs
*.rns filter=lfs diff=lfs merge=lfs
*.reason filter=lfs diff=lfs merge=lfs
*.lxo filter=lfs diff=lfs merge=lfs
```

ちなみにSourceTree使いな方は初期設定を気にしなくても使えます！  
（.gitattributesは個々に設定が必要です）

そんなこんなでとても便利そうなのですが、利用量に制限がありますのでバンバン使えるぜ！
ってわけではないのでご注意ください。

{{< blogcard url="https://docs.github.com/ja/github/managing-large-files/about-storage-and-bandwidth-usage" >}}

以上です。

■ 参考
{{< blogcard url="https://mseeeen.msen.jp/how-to-increase-git-lfs-data-capacity/" >}}
