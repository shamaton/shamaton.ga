---
title: '[Docker] prometheusをとりあえず動かしてみた'
author: しゃまとん
date: 2017-01-19T15:01:38+00:00
url: /posts/363
featured_image: /images/posts/2017/01/prometheus_logo.png
categories:
  - docker
  - Linux
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

少し前にTLにqiita経由で流れてきたprometheusについてずっと気になっていたので  
触ってみました。

{{< blogcard url="https://prometheus.io/" >}}

読みはプロメテウスでしょうか、、、最近注目を浴びている監視ツールだそうです。  
githubで公開されてて、スター数がすごいです。

{{< blogcard url="https://github.com/prometheus/prometheus" >}}

で、prmetheusがどんなものかですが前述の通りqiitaであったり、他の方が記事にしてくださっているので、
そちらを参照していただくとよいです。

{{< blogcard url="http://qiita.com/sugitak/items/ff8f5ad845283c5915d2" >}}

とりあえず自分は動かしてみたいと思ったので、簡単な構成でやってみたメモになります。
構成はprometheusをdockerで動作（ローカルPC）させ、[dply][4]で用意したホストを監視させました。

{{< blogcard url="/posts/341" >}}

※ 用語が間違ってたらごめんなさい

■ nodeとしてdplyでサーバを作成  
dplyでなんでもお好きなOSでサーバを作成します。dplyが嫌という方は用意できれば何でも良いと思います。  
作成後、node_exporterを取得し実行します。手順はこんな感じです。  
※ 最新版は[こちら][5]で確認してください

```shell
# 取得して解凍
wget https://github.com/prometheus/node_exporter/releases/download/v0.13.0/node_exporter-0.13.0.linux-amd64.tar.gz
tar -zxvf node_exporter-0.13.0.linux-amd64.tar.gz
# 実行
cd node_exporter-0.13.0.linux-amd64
./node_exporter
```

実行すると、9100でlistenします。

{{< figure src="/images/posts/2017/01/exec_node.png" >}}

■ Prometheusを起動する  
prometheusは[dockerhub][7]で直ぐ使えるものが用意されているため、すぐに使い始めることができます。
（docker run -p 9090:9090 prom/prometheus）  
今回はnodeとして立ち上がっているサーバを登録するため、設定ファイルを作成して利用するようにします。

とりあえずベースファイルを取得します。

```shell
curl -O https://raw.githubusercontent.com/prometheus/prometheus/master/documentation/examples/prometheus.yml
```

取得したymlファイルに追記します。（一番下の数行です）

{{<gist shamaton 86a8f21503c8ebf69dcedb9d2964059c >}}

準備ができたので、下記コマンドで実行してみます。

```shell
docker run -p 9090:9090 -v ${working_directory}/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

9090でlistenしているので、アクセスすると画面が表示されるます。プルダウンからnode_cpuを選んでExecuteすると、グラフが表示されるようになります。consoleにはdplyで定義したnodeに関しての値が表示され、取得出来ていることがわかります。

{{< figure src="/images/posts/2017/01/node_cpu.png" >}}

{{< figure src="/images/posts/2017/01/prometheus.png" >}}

あとは例えば、ノードを追加してグループにしたりとか、アラート（alertmanager）を設定して通知したりとか・・・DBサーバも監視対象にしたりとか・・・でしょうか。  
AWS等クラウドサービスにも対応しているようなので、少しづつ理解しながら使えるようになれればいいなと思います。

便利そうなニオイがプンプンしますね！  
以上です。

■ 参考
{{< blogcard url="http://pocketstudio.jp/log3/2015/02/11/what_is_prometheus_monitoring/" >}}
{{< blogcard url="http://qiita.com/hana_shin/items/16a7ee88ef502a3fc0eb" >}}

 [4]: https://dply.co
 [5]: https://prometheus.io/download/
 [7]: https://hub.docker.com/u/prom/