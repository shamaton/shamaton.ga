---
title: '[Docker] gcloudとkubectlが使えるコンテナを用意してみる'
author: しゃまとん
type: post
date: 2018-01-14T04:41:26+00:00
url: /archives/464
featured_image: /wp-content/uploads/2017/11/kube_logo.png
categories:
  - docker
  - GCP

---
お世話になっております。  
しゃまとんです。

gcloudとkubectlを自分のPCにinstallして使えるようにしているのですが、  
ふとコンテナでも使えるんかなーと思いやってみました。

とりあえず自分の環境を汚さずに使ってみたいという人はありかも？です。  
それでは手順です。

コンテナのベースにはubuntuを使います。  
まずはubuntuコンテナを作って入ります。

<pre class="lang:default decode:true ">docker pull ubuntu
docker run --name ubuntu_gcloud -d -i -t -p 8080 ubuntu /bin/bash
docker exec -i -t ubuntu_gcloud /bin/bash
※ port設定いらないかも</pre>

ここからはコンテナ内で行います。  
まずはgcloudをインストールするために必要なパッケージを入れておきます。

<pre class="lang:default decode:true ">apt-get -y update
apt-get -y install curl python</pre>

次に下記コマンドを実行します。  
コレを実行するとgcloudにインストールを行ってくれます。

<pre class="lang:default decode:true ">curl https://sdk.cloud.google.com | bash</pre>

途中で質問があるので答えておきます。  
何回か答えると、インストールが完了します。

<pre class="lang:default decode:true">Installation directory (this will create a google-cloud-sdk subdirectory) (/root): 空ENTER

Do you want to help improve the Google Cloud SDK (Y/n)? n（Yでもいいです）

Modify profile to update your $PATH and enable shell command
completion?

Do you want to continue (Y/n)? y

The Google Cloud SDK installer will now prompt you to update an rc
file to bring the Google Cloud CLIs into your environment.

Enter a path to an rc file to update, or leave blank to use
[/root/.bashrc]: 空ENTER
</pre>

直後はgcloudが有効になってないので、読み直しておきます。

<pre class="lang:default decode:true ">source ~/.bashrc</pre>

次に自分のアカウントと紐づけしていきます。  
ここでも質問があるので順にすすめます。  
まずは下記コマンドを実行。

<pre class="lang:default decode:true ">gcloud init</pre>

ログインしたいか聞かれるのでyにして表示されたURLをコピーしてブラウザに貼り付けます。  
すると認証コードが表示されるので、さらにコピーしてverification codeに貼り付けます。

<pre class="lang:default decode:true">You must log in to continue. Would you like to log in (Y/n)? y

Go to the following link in your browser:

# URLにアクセス
https://accounts.google.com/o/oauth2/auth?redirect_uri=xxxxxxxxxxxxxxxxxxx

Enter verification code: "認証コード"</pre>

認証が通ると、アカウントが表示されます。

そのまま、プロジェクトの選択を要求されます。今回は既存のプロジェクトを選択しました。

<pre class="lang:default decode:true ">You are logged in as: [youraccount@email.com].

Pick cloud project to use:
[1] project-xxxxx
[2] Create a new project
Please enter numeric choice or text value (must exactly match list
item): 1

Your current project has been set to: [project-xxxxx].</pre>

さらに普段使うzoneとregionも設定するか聞かれます。

日本がいい！ということであればasia-notheast1のどれかを選択しましょう。  
（nを選択しても後から、設定できます）

<pre class="lang:default decode:true">(https://cloud.google.com/compute) settings (Y/n)? Y

If you do not specify a zone via a command line flag while working
with Compute Engine resources, the default is assumed.
...
[4] asia-northeast1-c
[5] asia-northeast1-b
[6] asia-northeast1-a
...
</pre>

これでコマンドからインスタンスを作成する場合にzoneとregionがasia-northeastになります。

次にkubectlを使えるようにしておきます。  
下記コマンドを実行してyにするだけです。Update done!と表示されれば完了です。

<pre class="lang:default decode:true ">gcloud components install kubectl

Do you want to continue (Y/n)? y</pre>

一応コマンドが使えるか確認してみます。

とりあえずクラスタを作成してみます。

<pre class="lang:default decode:true ">root@df1165c2a16f:/# gcloud container clusters create test-cluster --machine-type=f1-micro
Creating cluster test-cluster...done.
Created [https://container.googleapis.com/v1/projects/project-xxxxx/zones/asia-northeast1-a/clusters/test-cluster].
kubeconfig entry generated for test-cluster.
NAME ZONE MASTER_VERSION MASTER_IP MACHINE_TYPE NODE_VERSION NUM_NODES STATUS
test-cluster asia-northeast1-a 1.7.8-gke.0 xxx.xxx.xxx.xxx f1-micro 1.7.8-gke.0 3 RUNNING</pre>

クラスタ情報を確認してみます。

<pre class="lang:default decode:true ">root@df1165c2a16f:/# kubectl cluster-info
Kubernetes master is running at https://xxx.xxx.xxx.xxx
GLBCDefaultBackend is running at https://xxx.xxx.xxx.xxx/api/v1/namespaces/kube-system/services/default-http-backend/proxy
Heapster is running at https://xxx.xxx.xxx.xxx/api/v1/namespaces/kube-system/services/heapster/proxy
KubeDNS is running at https://xxx.xxx.xxx.xxx/api/v1/namespaces/kube-system/services/kube-dns/proxy</pre>

最後に後始末しておきます。

<pre class="lang:default decode:true ">root@df1165c2a16f:/# gcloud container clusters delete test-cluster</pre>

コンテナ内でも変わらず使えるっぽいことがわかりました。  
コンテナは使い捨てが出来るので気楽に試せていいですね。  
以上です。

■ 参考  
<a href="https://qiita.com/kentarosasaki/items/2232113b44b016a56adc" target="_blank" rel="noopener">GCPのgcloudコマンドをインストールする</a>  
<a href="https://cloud.google.com/kubernetes-engine/docs/quickstart?hl=ja" target="_blank" rel="noopener">Google Kubernetes Engine クイックスタート</a>