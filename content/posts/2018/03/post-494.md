---
title: '[GCP] GoのコンテナをContainerRegistryに登録して使う'
author: しゃまとん
type: post
date: 2018-03-26T14:52:36+00:00
url: /archives/494
featured_image: /wp-content/uploads/2017/12/gcp_logo.png
categories:
  - GCP

---
お世話になっております。  
しゃまとんです。

KubernetesでGoを使いたいのですが、そのままコードを配置みたいな感じではなく、Dockerの様にコンテナイメージを作成してPodとして配置するような形式をとります。  
（勿論、やり方はコレ以外にもありそう？）

GCPには[Dockerhub][1]のようなコンテナのホスティングサービス（[Container Registry][2]）があり、yamlからアップロード済みイメージを利用することができるようになっています。

イメージを作成して、それを配置するだけとなると運用も楽ですしデプロイもRollingUpdateをいい感じにしてくれるみたいなので、それを前提にして考えたいと思います。

まずは簡単にテスト用のコードを用意します。  
アドレスで簡単な返答をするサンプルです。

<pre class="lang:go decode:true " title="main.go">package main

import (
  "fmt"
  "net/http"
)

type String string

func (s String) ServeHTTP(w http.ResponseWriter, r *http.Request) {
  fmt.Fprint(w, s)
}

func main() {
  http.Handle("/", String("ok"))
  http.Handle("/sample", String("sample"))
  http.ListenAndServe("localhost:9999", nil)
}
</pre>

次にこれをコンテナイメージにしていきます。  
Dockerfileを下記のような感じで作成しておきます。  
これをビルドしてイメージが作成されたら、Container Registryにアップロードしましょう。（your-project-idは置き換えてください）

<pre class="lang:default decode:true" title="Dockerfile">FROM golang:1.9.2-alpine

MAINTAINER shamaton

# copy source
COPY main.go main.go

# port
EXPOSE 9999

CMD ["go", "run", "main.go"]</pre>

<pre class="lang:default decode:true"># イメージ作成
docker build -t sample-web-app:ver1.0.0 .

# タグつけ
docker tag sample-web-app:ver1.0.0 asia.gcr.io/your-project-id/sample-web-app:ver1.0.0

# container registryにpush
gcloud docker -- push asia.gcr.io/your-project-id/sample-web-app:ver1.0.0</pre>

イメージが存在しているかコンソールから確認してみましょう。  
以下のようにsample-web-appが追加されていますね。

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2017/12/container_registry.png" alt="" width="1013" height="298" class="aligncenter size-full wp-image-509" />][3]

試しにローカルにpullして確認してみます。一応手元のビルドしたものを削除してしまいます。

<pre class="lang:default decode:true"># 手元のイメージを削除
docker rmi sample-web-app:ver1.0.0
docker rmi gcr.io/your-project-id/sample-web-app:ver1.0.0

# container registryから取得
gcloud docker -- pull gcr.io/your-project-id/sample-web-app:ver1.0.0

# コンテナ起動
docker run --name sample-app --rm -p 9999:9999 -t gcr.io/your-project-id/sample-web-app:ver1.0.0</pre>

アクセスすると&#8230;

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2017/12/access.png" alt="" width="546" height="258" class="aligncenter size-full wp-image-508" />][4]

同じように使えますね！Kubernetesからも同じように動作して使えますが、Dockerと一緒で使い捨てされることを考慮しておかないといけないので、運用の際には用途を決めておく必要がありそうです。

以上です。

■ 参考  
<a href="https://www.topgate.co.jp/gcp08-how-to-use-docker-image-container-registry" target="_blank" rel="noopener">【GCP入門編・第8回】 Container Registry での Docker イメージの使用方法！</a>

&nbsp;

 [1]: https://hub.docker.com/
 [2]: https://cloud.google.com/container-registry/?hl=ja
 [3]: https://shamaton.orz.hm/blog/wp-content/uploads/2017/12/container_registry.png
 [4]: https://shamaton.orz.hm/blog/wp-content/uploads/2017/12/access.png