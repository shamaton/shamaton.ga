---
title: '[Kubernetes] Skaffoldをとりあえず試してみた'
author: しゃまとん
type: post
date: 2018-03-29T15:05:12+00:00
url: /archives/530
featured_image: /wp-content/uploads/2017/11/kube_logo.png
categories:
  - GCP
  - Kubernetes

---
お世話になっております。  
しゃまとんです。

Skaffoldというツールがリリースされたので試してみました。  
（確認したバージョンはv0.2.0です。最新版では挙動が異なる場合あるので注意してください）

このツールですが、kubernetesを開発環境としても手軽に利用できるようにと開発されているもののようです。  
こちらの記事で紹介されています。

<a href="http://www.publickey1.jp/blog/18/googlekubernetesskaffoldkubernetesminikube.html" target="_blank" rel="noopener">Google、開発者のためのKubernetes用コマンドラインツール「Skaffold」オープンソースで公開。開発環境にもKubernetes、ローカルのMinikubeにも対応 － Publickey</a>

ちなみにskaffoldとは直訳では「足場」のようですが、いい感じに環境を組み上げてくれる仕組みに対して使われる言葉としてもあるようです。  
（よくわからんですね(；・∀・)）

ということで物は試しで進めていきましょう。  
試すにはkubernetesを使える環境が必要になります。minikubeでもいけるみたいですが、  
Dockerのedgeバージョンをインストールするとkubernetesを使えるのでこちらを使いました。

セットアップはこちらのページが参考になります。  
<a href="https://qiita.com/taishin/items/920d62a641c9cd58f289" target="_blank" rel="noopener">Docker for Mac with Kubernetes &#8211; Qiita</a>

できたら、次にgithubを開いてすすめていきます。  
[GoogleCloudPlatform/skaffold &#8211; Github][1]

最初にインストールからですね。Macの場合は下記コマンドを実行します。

<pre class="lang:sh decode:true">curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-darwin-amd64 && chmod +x skaffold && sudo mv skaffold /usr/local/bin</pre>

完了するとskaffoldがコマンドとして使えるようになります

<pre class="lang:default decode:true">$ skaffold help
A tool that makes the onboarding of existing applications to Kubernetes Engine simple and repeatable.

Usage:
  skaffold [command]

Available Commands:
  dev         runs a pipeline file in development mode
  docker      A set of commands related to developing with docker
  help        Help about any command
  run         runs a pipeline file
  version     print the version of skaffold

Flags:
  -h, --help               help for skaffold
  -v, --verbosity string   Log level (debug, info, warn, error, fatal, panic (default "warning")

Use "skaffold [command] --help" for more information about a command.</pre>

kubernetes環境はローカルのものを使うので、コンテキストを一応確認しておきます。  
kubectlはDockerのkuberenetesインストール時に使えるようになっています。

<pre class="lang:sh decode:true ">kubectl config get-contexts
# CURRENTがdocker-for-desktopになっていればOK、
# なってければ下記を実行する
kubectl config use-context docker-for-desktop</pre>

次に実際に使ってみましょう。  
適当なフォルダにcloneして、sampleまで移動します。

<pre class="lang:sh decode:true ">git clone https://github.com/GoogleCloudPlatform/skaffold
cd examples/getting-started</pre>

次にskaffold devするのですが、その前にskaffold.yamlを少しだけ変更してみます。  
そのまま実行しても動作してくれます。ちなみにデフォルトではskaffold.yamlを参照する仕組みになっているようです。（-fでファイル指定も出来ますよ）

<pre class="lang:yaml decode:true " title="skaffold.yaml">apiVersion: skaffold/v1alpha1
kind: Config
build:
  artifacts:
  - imageName: skaffold-sample
    workspace: ./container
  local: {}
deploy:
  kubectl:
    manifests:
    - paths:
      - k9s-*
      parameters:
        IMAGE_NAME: skaffold-sample</pre>

imageNameは適当に変更しつつ、workspaceとpathsにも変更をしたので、移動やリネームをしておきます。

<pre class="lang:sh decode:true">mkdir container
mv Dockerfile container
mv main.go container
mv k8s-pod.yaml k9s-pod.yaml</pre>

些細ですが、main.goはDockerfileから参照しているため同じ階層に存在していないとエラーになってしまいます。  
ではコレを実行してみます。

<pre class="lang:default decode:true ">$ skaffold dev
Starting build...
Found minikube or Docker for Desktop context, using local docker daemon.
Sending build context to Docker daemon 3.072kB
Step 1/5 : FROM golang:1.9.4-alpine3.7
---&gt; fb6e10bf973b
Step 2/5 : WORKDIR /go/src/github.com/GoogleCloudPlatform/skaffold/examples/getting-started
---&gt; Using cache
---&gt; 16374b174422
Step 3/5 : CMD ["./app"]
---&gt; Running in 0f1d8ffa68eb
---&gt; 8adde95a0823
Step 4/5 : COPY main.go .
---&gt; eaecb11ddccc
Step 5/5 : RUN go build -o app main.go
---&gt; Running in a346974a556e
---&gt; c0ecf6ce1930
Successfully built c0ecf6ce1930
Successfully tagged d5929250298f24ed0aa19e077f26e492:latest
Successfully tagged skaffold-sample:c0ecf6ce1930e65d7185384b7cfaf74eaba120f952d5c0a153730e05329edb45
Build complete.
Starting deploy...
Deploying k9s-pod.yaml...
Deploy complete.
[getting-started getting-started] Hello world!</pre>

これらから見るにworkspaceのDockerfileを参照してイメージを作成し、k9s-pod.yamlをkubectl apply -f している感じじゃないかと思います。  
次にコードを変更してみましょう。main.goを開いて&#8230;

<pre class="lang:go decode:true ">fmt.Println("Hello Shamaton!")</pre>

にしてみます。すると、自動的にdeployされてHello world!が切り替わります。

<pre class="lang:default decode:true ">[getting-started getting-started] Hello world!
[getting-started getting-started] Hello world!
Starting build...
Found minikube or Docker for Desktop context, using local docker daemon.
Sending build context to Docker daemon 3.072kB
[getting-started getting-started] Hello world!
Step 1/5 : FROM golang:1.9.4-alpine3.7
---&gt; fb6e10bf973b
Step 2/5 : WORKDIR /go/src/github.com/GoogleCloudPlatform/skaffold/examples/getting-started
---&gt; Using cache
---&gt; 7db3074bce0a
Step 3/5 : CMD ["./app"]
---&gt; Using cache
---&gt; 8e2658a11d85
Step 4/5 : COPY main.go .
---&gt; a3aff99ad423
Step 5/5 : RUN go build -o app main.go
---&gt; Running in 419ccc215f8a
[getting-started getting-started] Hello world!
[getting-started getting-started] Hello world!
---&gt; 22d6738635a1
Successfully built 22d6738635a1
Successfully tagged bcf6d7576074a13ed9d320c93d528b6f:latest
Successfully tagged skaffold-sample:22d6738635a11e3648f8c7ca771848e320f2954a148123c46e1a524b0059079c
Build complete.
Starting deploy...
Deploying k9s-pod.yaml...
Deploy complete.
[getting-started getting-started] Hello world!
[getting-started getting-started] Hello world!
[getting-started getting-started] Hello world!
[getting-started getting-started] Hello world!
[getting-started getting-started] Hello world!
[getting-started getting-started] Hello Shamaton!
[getting-started getting-started] Hello Shamaton!</pre>

一応imageも確認しておくと、一度更新したのでsampleが2つになっていますね。

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2018/03/p1.png" alt="" width="1036" height="116" class="aligncenter size-full wp-image-532" />][2]

すっごいimage増えそう。止める時はどうするのかな&#8230;  
kubectl実行すればいいのだろうか。

<pre class="lang:sh decode:true">kubectl delete -f k8s-pod.yaml</pre>

とりあえずこんな感じで。  
ローカルでkubernetesを簡単に試せるのがいいですね（skaffold関係ない）！  
以上です。

 [1]: https://github.com/GoogleCloudPlatform/skaffold
 [2]: https://shamaton.orz.hm/blog/wp-content/uploads/2018/03/p1.png