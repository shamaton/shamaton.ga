---
title: '[GKE] kube-legoを使ってドメインからアクセスできるようにする'
author: しゃまとん
type: post
date: 2018-05-15T22:41:44+00:00
url: /archives/477
featured_image: /wp-content/uploads/2017/11/kube_logo.png
categories:
  - GCP
  - Kubernetes

---
お世話になっております。  
しゃまとんです。

GKEでサービスをある程度作って公開しようとなったとき、HTTPSで通信させたくなります。  
というものアプリだとHTTPSを推奨していたり、Webサービスなら検索に影響したりするからです。

GKEでどう実現したらいいのか、まだあまり詳しくはないのですがKubernetes上でHTTPSを簡単に実現できる[kube-lego][1]というものがあるのでそちらを利用してやってみました。

まずはリポジトリをクローンして、設定ファイルを変更します。

<pre class="lang:default decode:true"># 取得
git clone https://github.com/jetstack/kube-lego.git

# ファイル編集
cd kube-lego
vim examples/gce/lego/configmap.yaml

apiVersion: v1
metadata:
  name: kube-lego
  namespace: kube-lego
data:
  # modify this to specify your address（ここを自分のメールアドレスにする）
  lego.email: "your.address@example.com"
  # configure letsencrypt's production api
  lego.url: "https://acme-v01.api.letsencrypt.org/directory"
kind: ConfigMap</pre>

これで環境にapplyします。

<pre class="lang:default decode:true">kubectl apply -f  kube-lego/examples/gce/lego

# 3つ生成される
namespace "kube-lego" created
configmap "kube-lego" created
deployment "kube-lego" created</pre>

独自ドメインでのアクセスには静的IPが必要なので、GCPのコンソール等から操作して取得しておきましょう。（VPCネットワーク　→　外部IPアドレス）もしくは下記コマンドで作成しましょう。

<pre class="lang:default decode:true ">gcloud compute addresses create example-ip --global</pre>

次にドメインですが、<a href="https://px.a8.net/svt/ejp?a8mat=2TE6SZ+7KOJUA+50+2HHVNM" target="_blank" rel="nofollow noopener">お名前.com</a><img src="https://www18.a8.net/0.gif?a8mat=2TE6SZ+7KOJUA+50+2HHVNM" border="0" alt="" width="1" height="1" />とかどこでもいいですが今回は無料で取得できる[freenom][2]を利用してみました。こんな感じで簡単に確認できるので希望するものがとれれば選びましょう。取得できたら、CloudDNSでドメインを登録し、Aレコード取得したIPアドレスを設定しておきます。（黒塗りして申し訳ないですが&#8230;）

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2017/12/2.png" alt="" width="506" height="366" class="aligncenter size-full wp-image-503" />][3]

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2017/12/3.png" alt="" width="910" height="377" class="aligncenter size-full wp-image-502" />][4]

freenom側でもnameserverを設定しておきます。  
Services → MyDomains → ManageDomain → Management Tools → Nameserversで設定できます。例なのでE1のところは登録時で変わると思います。

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2017/12/4.png" alt="" width="1131" height="523" class="aligncenter size-full wp-image-504" />][5]

これでドメインとIPの紐付けができたので、後はWebサーバ的な返答を返すものを用意していきます。Goで簡単に動作するものを用意しました。image化についてはこちらを参照してください。  
<https://shamaton.orz.hm/blog/archives/494>

<pre class="lang:go decode:true ">package main

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
  http.Handle("/health", String("health"))
  http.ListenAndServe("localhost:9999", nil)
}
</pre>

これをPodとして配置したいので、設定ファイルを作ります。雑ですがこんな感じです。  
Port:9999でListenし、Serviceで80ポートで通信できるようにしておきます。

<pre class="lang:default decode:true" title="deployment.yaml">apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 2
  template:
    metadata:
      labels:
        name: app
    spec:
      containers:
      - image: asia.gcr.io/（プロジェクトID）/（コンテナ名）:（タグ）
        imagePullPolicy: Always
        name: app
        ports:
        - containerPort: 9999
        livenessProbe:
          httpGet:
            path: /health
            port: 9999
---
apiVersion: v1
kind: Service
metadata:
  name: app
spec:
  selector:
    name: app
  type: NodePort
  ports:
    - port: 80
      targetPort: 9999</pre>

最後にkube-legoを使うingressを用意します。設定ファイルはこちら。

<pre class="lang:default decode:true" title="ingress.yaml">apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: app
  annotations:
    kubernetes.io/ingress.global-static-ip-name: "example-ip"
    kubernetes.io/ingress.class: "gce"
    kubernetes.io/tls-acme: "true"
spec:
  tls:
  - secretName: example-ip-tls
    hosts:
      - example.com
  rules:
    - host: example.com
      http:
        paths:
        - path: /
          backend:
            serviceName: app
            servicePort: 80
  backend:
    serviceName: app
    servicePort: 80
</pre>

こちらを適応させましょう。

<pre class="lang:default decode:true ">kubectl apply -f deployment.yaml
kubectl apply -f ingress.yaml</pre>

実行後、設定されるまでに時間がかかるのでログをみておくと状態がわかるので安心です。  
下記コマンドを実行しておいたときのログ例も載せておきます。

<pre class="lang:default decode:true ">kubectl logs -f --namespace kube-lego $(kubectl get pod --namespace kube-lego -l app=kube-lego -o name)</pre>

<pre class="lang:default decode:true">level=info msg="created an ACME account (registration url: https://acme-v01.api.letsencrypt.org/acme/reg/24364503)" context=acme
 level=info msg="Attempting to create new secret" context=secret name=kube-lego-account namespace=kube-lego
 level=info msg="Secret successfully stored" context=secret name=kube-lego-account namespace=kube-lego
 level=debug msg="testing reachability of http://example.com/.well-known/acme-challenge/_selftest" context=acme domain=example.com
 
 # 最初は404
 level=debug msg="error while authorizing: reachability test failed: wrong status code '404'" context=acme domain=example.com
 level=warning msg="authorization failed after 1m0s: reachability test failed: wrong status code '404'" context=acme domain=example.com
 level=error msg="Error while processing certificate requests: no domain could be authorized successfully" context=kubelego
 level=debug msg="worker: done processing true" context=kubelego
 level=debug msg="worker: begin processing true" context=kubelego
 level=debug msg=reset context=provider provider=gce
 level=debug msg=finalize context=provider provider=gce
 level=debug msg="setting up svc endpoint" context=provider namespace=default pod_ip=10.16.0.9 provider=gce
 level=debug msg=reset context=provider provider=nginx
 level=debug msg=finalize context=provider provider=nginx
 level=info msg="disable provider no TLS hosts found" context=provider provider=nginx
 level=info msg="process certificate requests for ingresses" context=kubelego
 level=debug msg="UPDATE ingress/default/app" context=kubelego
 level=debug msg="UPDATE ingress/default/app" context=kubelego
 level=debug msg="testing reachability of http://example.com/.well-known/acme-challenge/_selftest" context=acme domain=example.com
 level=debug msg="error while authorizing: reachability test failed: wrong status code '404'" context=acme domain=example.com
 level=warning msg="authorization failed after 1m0s: reachability test failed: wrong status code '404'" context=acme domain=example.com
 level=error msg="Error while processing certificate requests: no domain could be authorized successfully" context=kubelego
 level=debug msg="worker: done processing true" context=kubelego
 level=debug msg="worker: begin processing true" context=kubelego
 level=debug msg="setting up svc endpoint" context=provider namespace=default pod_ip=10.16.0.9 provider=gce
 level=debug msg="testing reachability of http://example.com/.well-known/acme-challenge/_selftest" context=acme domain=example.com

　# 502になった
 level=debug msg="error while authorizing: reachability test failed: wrong status code '502'" context=acme domain=example.com
 level=debug msg="responding to challenge request" basePath="/.well-known/acme-challenge" context=acme host=example.com token="QxQNZDyx5ZnMR7LP_7KbHLal5T3y7DOSr2YZmUrv6QU"
 level=debug msg="got authorization: &{URI:https://acme-v01.api.letsencrypt.org/acme/challenge/bdYLpT-18ASFdaclX6ZX_f7FSdK1CLJhPlNsdt42_MQ/2487891091 Status:valid Identifier:{Type: Value:} Challenges:[] Combinations:[]}" context=acme domain=example.com

 # 成功
 level=info msg="authorization successful" context=acme domain=example.com
 level=info msg="successfully got certificate: domains=[example.com] url=https://acme-v01.api.letsencrypt.org/acme/cert/0380189d6956fc46add22b4c13ed3d190e2e" context=acme
^C</pre>

これで指定のドメインにアクセスしてみると&#8230;

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2017/12/5.png" alt="" width="388" height="167" class="aligncenter size-full wp-image-505" />][6]

httpsでアクセスすることができました！（ドメインは伏せました）  
とりあえずこれでいけそうですね！

以上です。

■ 参考  
<a href="https://qiita.com/apstndb/items/2fef0a80d4510516cb1f" target="_blank" rel="noopener">GKE でサービスを HTTPS と HTTP/2 に対応する(kube-lego 編)</a>  
<a href="https://qiita.com/esplo/items/bd2e36cfae797d0d480a" target="_blank" rel="noopener">GCP上で独自ドメインを使う</a>  
<a href="https://qiita.com/teekay/items/135dc67e39f24997019e" target="_blank" rel="noopener">無料のドメインを取得する</a>  
<a href="https://www.compiere-distribution-lab.net/idempiere-lab/install-advance/google-cloud-dns/" target="_blank" rel="noopener">GCE（Google Compute Engine）で独自ドメインを設定する</a>

 [1]: https://github.com/jetstack/kube-lego
 [2]: http://www.freenom.com/ja/index.html
 [3]: https://shamaton.orz.hm/blog/wp-content/uploads/2017/12/2.png
 [4]: https://shamaton.orz.hm/blog/wp-content/uploads/2017/12/3.png
 [5]: https://shamaton.orz.hm/blog/wp-content/uploads/2017/12/4.png
 [6]: https://shamaton.orz.hm/blog/wp-content/uploads/2017/12/5.png