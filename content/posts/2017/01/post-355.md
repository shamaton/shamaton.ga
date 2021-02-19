---
title: '[Docker] prometheusをとりあえず動かしてみた'
author: しゃまとん
type: post
date: 2017-01-19T15:01:38+00:00
url: /archives/355
featured_image: /wp-content/uploads/2017/01/prometheus_logo.png
categories:
  - docker
  - Linux
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

少し前にTLにqiita経由で流れてきた[prometheus][1]についてずっと気になっていたので  
触ってみました。  
読みはプロメテウスでしょうか、、、最近注目を浴びている監視ツールだそうです。  
[github][2]で公開されてて、スター数がすごいです。

で、prmetheusがどんなものかですが前述の通りqiitaであったり、他の方が記事にしてくださっているので、そちらを参照していただくとよいです。

[qiita &#8211; 次世代監視の大本命！ Prometheus を実運用してみた][3]

とりあえず自分は動かしてみたいと思ったので、簡単な構成でやってみたメモになります。構成はprometheusをdockerで動作（ローカルPC）させ、[dply][4]で用意したホストを監視させました。

<blockquote class="wp-embedded-content">
  <p>
    <a href="https://shamaton.orz.hm/blog/archives/341">dplyのボタンつくってみた</a>
  </p>
</blockquote>



※ 用語が間違ってたらごめんなさい

■ nodeとしてdplyでサーバを作成  
dplyでなんでもお好きなOSでサーバを作成します。dplyが嫌という方は用意できれば何でも良いと思います。  
作成後、node_exporterを取得し実行します。手順はこんな感じです。  
※ 最新版は[こちら][5]で確認してください

<pre class="lang:sh decode:true"># 取得して解凍
wget https://github.com/prometheus/node_exporter/releases/download/v0.13.0/node_exporter-0.13.0.linux-amd64.tar.gz
tar -zxvf node_exporter-0.13.0.linux-amd64.tar.gz
# 実行
cd node_exporter-0.13.0.linux-amd64
./node_exporter</pre>

実行すると、9100でlistenします。

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2017/01/exec_node.png" alt="" width="494" height="240" class="aligncenter wp-image-359 size-full" />][6]

■ Prometheusを起動する  
prometheusは[dockerhub][7]で直ぐ使えるものが用意されているため、すぐに使い始めることができます。（docker run -p 9090:9090 prom/prometheus）  
今回はnodeとして立ち上がっているサーバを登録するため、設定ファイルを作成して利用するようにします。

とりあえずベースファイルを取得します。

<pre class="lang:sh decode:true ">curl -O https://raw.githubusercontent.com/prometheus/prometheus/master/documentation/examples/prometheus.yml</pre>

取得したymlファイルに追記します。（一番下の数行です）



準備ができたので、下記コマンドで実行してみます。

<pre class="lang:sh decode:true ">docker run -p 9090:9090 -v ${working_directory}/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus</pre>

9090でlistenしているので、アクセスすると画面が表示されるます。プルダウンからnode_cpuを選んでExecuteすると、グラフが表示されるようになります。consoleにはdplyで定義したnodeに関しての値が表示され、取得出来ていることがわかります。

<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2017/01/node_cpu.png" alt="" width="470" height="237" class="aligncenter size-full wp-image-358" /> 

&nbsp;

<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2017/01/prometheus.png" alt="" width="471" height="353" class="aligncenter size-full wp-image-357" /> 

あとは例えば、ノードを追加してグループにしたりとか、アラート（alertmanager）を設定して通知したりとか・・・DBサーバも監視対象にしたりとか・・・でしょうか。  
AWS等クラウドサービスにも対応しているようなので、少しづつ理解しながら使えるようになれればいいなと思います。

便利そうなニオイがプンプンしますね！  
以上です。

■ 参考  
[【入門】PrometheusでサーバやDockerコンテナのリソース監視  
][8] [Prometheusのインストール][9]

 [1]: https://prometheus.io/
 [2]: https://github.com/prometheus/prometheus
 [3]: http://qiita.com/sugitak/items/ff8f5ad845283c5915d2
 [4]: https://dply.co
 [5]: https://prometheus.io/download/
 [6]: https://shamaton.orz.hm/blog/wp-content/uploads/2017/01/exec_node.png
 [7]: https://hub.docker.com/u/prom/
 [8]: http://pocketstudio.jp/log3/2015/02/11/what_is_prometheus_monitoring/
 [9]: http://qiita.com/hana_shin/items/16a7ee88ef502a3fc0eb