---
title: '[GCP] kubernetesでノードを新しく用意してPodを移動させてみる'
author: しゃまとん
type: post
date: 2018-04-29T14:26:22+00:00
url: /archives/496
featured_image: /wp-content/uploads/2017/11/kube_logo.png
categories:
  - GCP
  - Kubernetes

---
お世話になっております。  
しゃまとんです。

kubernetesでサーバー運営をする際、ノード上の配置される様々なPodでサービス運営を行うわけですが、今回試しに移動させてみました。

背景としては、アプリがヒットした場合（なかなかそんなことない）、各サービスをスケールアウトさせないといけなかったり、ヒットしない場合（よくある）スケールダウンさせないといけなかったりします。

私は、コマンドで全てを操作できるほど熟知してはないので適時Webコンソールで対応しました。ちなみに動作環境では下記のPodが動作していました。

・Redisクラスタ（master x 3, slave x 3)  
・APIサーバ（2台）  
・Ubuntu（Redisクラスタ操作用）※運用には必要ない

これらのPodを移動させる必要があります。調べてみた感じだとdrainを使うことによって、指定のノードからノードにpodを移動できるようですが、Redisクラスタが最小構成で組まれていたため、手動でPodを新しいノードへ移動させることにしました。  
（Redisクラスタも、新しいノードにslaveを余分作っておけばdrainで行けそうな気がします）

仮にこのようなノード構成だとします。  
実際のノードはkubectl get nodeで確認できます。

<pre class="lang:default decode:true">gke-current-pool-1
gke-current-pool-2
gke-current-pool-3
gke-new-pool-1
gke-new-pool-1
gke-new-pool-1</pre>

まずは削除したいノードにこれ以上Podが生成されないようにしておきます。  
実行後にノードのステータスを確認すると、Ready,SchedulingDisabledになっていることがわかります。

<pre class="lang:default decode:true">$ kubectl cordon gke-current-pool-1
node "gke-current-pool-1" cordoned
$ kubectl cordon gke-current-pool-2
node "gke-current-pool-2" cordoned
$ kubectl cordon gke-current-pool-3
node "gke-current-pool-3" cordoned</pre>

これで移動させる準備ができたのでRedisを1つ移動させてみます。

<pre class="lang:default decode:true ">kubectl apply -f redis-6.yaml
kubectl delete -f redis-6.yaml
kubectl create -f redis-6.yaml

クラスタ構成を更新するのでUbuntu側でredis-trib.rbを実行しておきます
redis-trib.rb add-node --slave 10.19.255.6:6379 10.19.255.1:6379
>&gt;&gt; Send CLUSTER MEET to node 10.19.255.6:6379 to make it join the cluster.

クラスタに接続して、ノード情報を確認します

root@ubuntu:/# redis-cli -h redis -c
redis:6379&gt; cluster nodes
e43cb2f05e9e8265ceb0ebfb443a2674cfaaf440 10.16.1.10:6379 master - 0 1512833161767 2 connected 5461-10922
f2d55c8de1f7b568d4272e6d961f9b6e8e86bbf1 10.16.2.10:6379 slave,fail 80a53be9d7281ba560d81af3e0c3c89416d2c10a 1512832594104 1512832593000 6 disconnected
80a53be9d7281ba560d81af3e0c3c89416d2c10a 10.16.0.7:6379 master - 0 1512833161265 3 connected 10923-16383
ca68d56f6e92a077c1e907d5c2614cbc90176667 10.16.5.2:6379 slave 80a53be9d7281ba560d81af3e0c3c89416d2c10a 0 1512833160262 3 connected
5f25d4a85c5c20ce16f6b0f9688566e23ea85ce9 10.16.2.8:6379 slave 26502f5d58cf84327911e9cb2578243b46e0eb89 0 1512833161265 4 connected
0fb050edb5fe569fa31dea20133833854c73cf35 10.16.2.9:6379 myself,slave e43cb2f05e9e8265ceb0ebfb443a2674cfaaf440 0 0 5 connected
26502f5d58cf84327911e9cb2578243b46e0eb89 10.16.1.9:6379 master - 0 1512833160765 1 connected 0-5460</pre>

1つfailになっていますが、稼働中のmaster/slaveが3つずつあることがわかります。  
もしデータが入っていれば、getして確認してみるのもいいですね。  
これで新しいノードに移動できることがわかったので、全てのredis（今回は1〜5）に対して実行していきます。（移動した際のキャプチャ取るのを忘れた。。）

全て実行するとfailなものが残っているので、気持ち悪い場合は対処しておきます。  
下記コマンドできれいになります。（はず）

<pre class="lang:default decode:true">./redis-trib.rb call 10.16.3.3:6379 CLUSTER FORGET 5f25d4a85c5c20ce16f6b0f9688566e23ea85ce9</pre>

APIサーバーは使い捨てのコンテナになっているので、podを指定して削除してみました。

<pre class="lang:default decode:true">kubectl delete pod app-1127512366-6759r
kubectl delete pod app-1127512366-nghkf</pre>

最終的にpodはこのようになりました。

<pre class="lang:default decode:true">NAME          READY       STATUS      RESTARTS    AGE
app-1127512366-6759r    2/2     Running     0       13d
app-1127512366-nghkf    2/2     Running     1       13d
redis-1-61ztz       1/1     Running     0       22d
redis-2-56sxf       1/1     Running     0       22d
redis-3-tf1qf       1/1     Running     0       22d
redis-4-jgqck       1/1     Running     0       22d
redis-5-63kt9       1/1     Running     0       22d
redis-6-t0jjt       1/1     Running     0       22d
ubuntu          1/1     Running     0       7m

↓

NAME            READY       STATUS      RESTARTS    AGE
app-1127512366-3dwz1    2/2     Running     0       3m
app-1127512366-n1k2j    2/2     Running     1       1m
redis-1-wvpnj       1/1     Running     0       11m
redis-2-wqsf4       1/1     Running     0       14m
redis-3-phjld       1/1     Running     0       17m
redis-4-19l64       1/1     Running     0       19m
redis-5-tndn5       1/1     Running     0       22m
redis-6-7283h       1/1     Running     0       23m</pre>

移動させたら不要なノードは削除しておきます。  
Webコンソールからでも削除可能です。

<pre class="lang:default decode:true ">gcloud container node-pools delete current-node-pool</pre>

今回は手動で行ったので、すこし手間のかかる作業になってしまいました。drain使うのが多分いいですね。kubernetesは耐障害性があるので、この辺を把握しておくとちょっとよいかもですね。  
以上です。

参考  
<a href="http://shepherdmaster.hateblo.jp/entry/2015/07/21/044119" target="_blank" rel="noopener">Redis Clusterの操作を簡単にするredis-trib.rbの使い方</a>  
<a href="http://blog.applibot.co.jp/blog/2016/12/27/kubernetes-zero-downtime/" target="_blank" rel="noopener">kubernetesの無停止運用を意識した検証<br /> </a>