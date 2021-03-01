---
title: '[GKE] kube-legoを使ってドメインからアクセスできるようにする'
author: しゃまとん
date: 2018-05-15T22:41:44+00:00
url: /posts/477
featured_image: /images/posts/2017/11/kube_logo.png
categories:
  - GCP
  - Kubernetes

---
お世話になっております。  
しゃまとんです。

GKEでサービスをある程度作って公開しようとなったとき、HTTPSで通信させたくなります。  
というものアプリだとHTTPSを推奨していたり、Webサービスなら検索に影響したりするからです。

GKEでどう実現したらいいのか、まだあまり詳しくはないのですがKubernetes上でHTTPSを簡単に実現できるkube-legoというものが
あるのでそちらを利用してやってみました。

{{< blogcard url="https://github.com/jetstack/kube-lego" >}}

まずはリポジトリをクローンして、設定ファイルを変更します。

```shell
# 取得
git clone https://github.com/jetstack/kube-lego.git

# ファイル編集
cd kube-lego
vim examples/gce/lego/configmap.yaml
```

```yaml
apiVersion: v1
metadata:
  name: kube-lego
  namespace: kube-lego
data:
  # modify this to specify your address（ここを自分のメールアドレスにする）
  lego.email: "your.address@example.com"
  # configure letsencrypt's production api
  lego.url: "https://acme-v01.api.letsencrypt.org/directory"
kind: ConfigMap
```

これで環境にapplyします。

```shell
$ kubectl apply -f  kube-lego/examples/gce/lego

# 3つ生成される
namespace "kube-lego" created
configmap "kube-lego" created
deployment "kube-lego" created
```

独自ドメインでのアクセスには静的IPが必要なので、GCPのコンソール等から操作して取得しておきましょう。 
（VPCネットワーク　→　外部IPアドレス）

もしくは下記コマンドで作成しましょう。

```shell
gcloud compute addresses create example-ip --global
```

次にドメインですが、[お名前.com](https://px.a8.net/svt/ejp?a8mat=2TE6SZ+7KOJUA+50+2HHVNM)とかどこでもいいですが
今回は無料で取得できる[freenom][2]を利用してみました。
こんな感じで簡単に確認できるので希望するものがとれれば選びましょう。
取得できたら、CloudDNSでドメインを登録し、Aレコード取得したIPアドレスを設定しておきます。（黒塗りして申し訳ないですが...）

{{< figure src="/images/posts/2017/12/2.png" >}}

---

{{< figure src="/images/posts/2017/12/3.png" >}}

freenom側でもnameserverを設定しておきます。  
Services → MyDomains → ManageDomain → Management Tools → Nameserversで設定できます。例なのでE1のところは登録時で変わると思います。

{{< figure src="/images/posts/2017/12/4.png" >}}

これでドメインとIPの紐付けができたので、後はWebサーバ的な返答を返すものを用意していきます。
Goで簡単に動作するものを用意しました。image化については[こちら](/posts/494)を参照してください。  

```go
package main

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
```

これをPodとして配置したいので、設定ファイル(deployment.yaml)を作ります。雑ですがこんな感じです。  
Port:9999でListenし、Serviceで80ポートで通信できるようにしておきます。

```yaml
apiVersion: apps/v1beta1
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
      targetPort: 9999
```

最後にkube-legoを使うingressを用意します。設定ファイル(ingress.yaml)はこちら。

```yaml
apiVersion: extensions/v1beta1
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
```

こちらを適応させましょう。

```shell
kubectl apply -f deployment.yaml
kubectl apply -f ingress.yaml
```

実行後、設定されるまでに時間がかかるのでログをみておくと状態がわかるので安心です。  
下記コマンドを実行しておいたときのログ例も載せておきます。

```shell
kubectl logs -f --namespace kube-lego $(kubectl get pod --namespace kube-lego -l app=kube-lego -o name)
```

```text
 level=info msg="created an ACME account (registration url: https://acme-v01.api.letsencrypt.org/acme/reg/24364503)" context=acme
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
^C
```

これで指定のドメインにアクセスしてみると...

{{< figure src="/images/posts/2017/12/5.png" >}}

httpsでアクセスすることができました！（ドメインは伏せました）  
とりあえずこれでいけそうですね！

以上です。

■ 参考

{{< blogcard url="https://qiita.com/apstndb/items/2fef0a80d4510516cb1f" >}}
{{< blogcard url="https://qiita.com/esplo/items/bd2e36cfae797d0d480a" >}}
{{< blogcard url="https://qiita.com/teekay/items/135dc67e39f24997019e" >}}
{{< blogcard url="https://www.compiere-distribution-lab.net/idempiere-lab/install-advance/google-cloud-dns/" >}}
