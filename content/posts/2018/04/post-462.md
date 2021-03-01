---
title: '[GCP] GKEでredis-clusterを自力で作ってみた（ボツ）'
author: しゃまとん
date: 2018-04-09T15:06:13+00:00
url: /posts/462
featured_image: /images/posts/2017/11/kube_logo.png
categories:
  - GCP

---
お世話になっております。  
しゃまとんです。

前に[redis-clusterを試してみた記事](/posts/461)を作成したのですが、今回はGKEでクラスタを作ってみることにしました。
というのもWebDB PRESSで特集されていてやってみようと思ったのがきっかけです。

{{< blogcard url="http://gihyo.jp/magazine/wdpress/archive/2017/vol99" >}}

という記事なんですが、結局、ボツに至ったものになります。
作業ログとして残しておきたいと思います。本自体はとても勉強になりました。

やったこととしては独自のredisコンテナ作成し、起動時に自分の名前に沿ってmaster/slaveを判断して、
clusterに参加するといったものです。それではまずredisイメージファイル（Dockerfile）になります。

```dockerfile
FROM redis:3.2.4-alpine

MAINTAINER shamaton

RUN apk add --update bind-tools curl && \
    rm -rf /var/cache/apk/*

COPY bootstrap.sh /bootstrap.sh
COPY redis.conf /conf/redis.conf

CMD ["redis-server", "/conf/redis.conf"]
```

公式にRedisコンテナに起動スクリプトと設定ファイル（conf）を追加しています。  
起動スクリプト(bootstrap.sh)と設定ファイル(redis.conf)は下記のような感じです。
スクリプトにはchmodで実行権限をつけておいてください（chmod +x）

```shell
#!/bin/sh
# set -e

SEQ_START=0
SEQ_END=16383

PET_NUMBER=$(cat /etc/podinfo/pod_name | cut -d- -f3)
PET_ORDINAL=$(expr ${PET_NUMBER} % ${REPLICA_SIZE})

MASTER_NUMBER=$(expr ${PET_NUMBER} - ${PET_ORDINAL})

# master server setting
if [ ${PET_ORDINAL} = "0" ]; then
  GROUP_NUM=$(expr ${REPLICA_NUM} / ${REPLICA_SIZE})
  GROUP_END_INDEX=$(expr ${GROUP_NUM} - 1)

  SEQ_PER=$(expr ${SEQ_END} / ${GROUP_NUM})

  GROUP_INDEX=$(expr ${PET_NUMBER} / ${REPLICA_SIZE})

  SET_SEQ_START=$(expr ${SEQ_PER} \* ${GROUP_INDEX} + 1)
  SET_SEQ_END=$(expr ${SET_SEQ_START} + ${SEQ_PER} - 1)

  if [ ${GROUP_INDEX} = "0" ]; then
    SET_SEQ_START=0
  fi

  if [ ${GROUP_INDEX} = "${GROUP_END_INDEX}" ]; then
    SET_SEQ_END=${SEQ_END}
  fi

  echo "cluster set seq : ${SET_SEQ_START} ${SET_SEQ_END}"

  redis-cli cluster addslots $(seq ${SET_SEQ_START} ${SET_SEQ_END})
fi

# meet redis-cluster-0
if [ ! ${PET_NUMBER} = "0" ]; then

  MASTER_ZERO_FULLNAME=redis-cluster-0.redis-cluster.default.svc.cluster.local
  MASTER_ZERO_IP=$(host ${MASTER_ZERO_FULLNAME} | cut -d ' ' -f 4)

  echo "MASTER_ZERO_IP : ${MASTER_ZERO_IP}"

  redis-cli cluster meet ${MASTER_ZERO_IP} 6379
fi

sleep 1

# slave server setting
if [ ! ${PET_ORDINAL} = "0" ]; then

  MASTER_FULLNAME=redis-cluster-${MASTER_NUMBER}.redis-cluster.default.svc.cluster.local
  MASTER_IP=$(host ${MASTER_FULLNAME} | cut -d ' ' -f 4)

  echo "MASTER_IP : ${MASTER_IP}"

  MASTER_ID=$(redis-cli cluster nodes | grep ${MASTER_IP} | cut -d ' ' -f 1)

  echo "MASTER_ID : ${MASTER_ID}"

  redis-cli cluster replicate ${MASTER_ID}
fi

wait
```

```text
appendonly yes
cluster-enabled yes
cluster-require-full-coverage no
cluster-node-timeout 5000
cluster-config-file nodes.conf
cluster-require-full-coverage yes
cluster-migration-barrier 1
protected-mode no
```

用意できたら、イメージを作成して[Container Registry](https://cloud.google.com/container-registry/docs/overview?hl=ja)に追加します。

```shell
docker build -t redis-cluster:${version} .
docker tag redis-cluster:${version} asia.gcr.io/your_project_id/redis-cluster:${version}
gcloud docker -- push asia.gcr.io/your_project_id/redis-cluster:${version}
```

それではクラスタを作成して、redisクラスタをKubernetes上で作成してみましょう。  
配置にひつようなyamlファイル(cluster.yaml)は下記のようになります。（プロジェクトIDは置き換えてください）

```yaml
#
# Redis Cluster service
#
apiVersion: v1
kind: Service
metadata:
  name: redis-cluster
  labels:
    app: redis-cluster
spec:
  ports:
  - port: 6379
    name: redis
  clusterIP: None
  selector:
    app: redis-cluster
---
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: redis-cluster
spec:
  serviceName: redis-cluster
  replicas: 6
  template:
    metadata:
      labels:
        app: redis-cluster
    spec:
      containers:
      - name: redis-cluster
        image: asia.gcr.io/your_project_id/redis-cluster:${version}
        imagePullPolicy: Always
        lifecycle:
          postStart:
            exec:
              command: [ "/bootstrap.sh" ]
        ports:
          - containerPort: 6379
            name: client
          - containerPort: 16379
            name: gossip
        readinessProbe:
          exec:
            command:
            - sh
            - -c
            - "redis-cli -h $(hostname) ping"
          initialDelaySeconds: 15
          timeoutSeconds: 5
        livenessProbe:
          exec:
            command:
            - sh
            - -c
            - "redis-cli -h $(hostname) ping"
          initialDelaySeconds: 20
          periodSeconds: 3
        env:
          - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
          - name: REPLICA_NUM
            value: "6"
          - name: REPLICA_SIZE
            value: "2"
        volumeMounts:
        - name: podinfo
          mountPath: /etc/podinfo
          readOnly: false
      volumes:
      - name: podinfo
        downwardAPI:
          items:
            - path: "labels"
              fieldRef:
                fieldPath: metadata.labels
            - path: "annotations"
              fieldRef:
                fieldPath: metadata.annotations
            - path: "pod_name"
              fieldRef:
                fieldPath: metadata.name
            - path: "pod_namespace"
              fieldRef:
                fieldPath: metadata.namespace
```

これを適用させます。すると各redisのpodが起動スクリプトで自分の役割を把握し必要に応じたredisコマンドを実行していきます。

```shell
gcloud container clusters create test-cluster --machine-type=f1-micro
kubectl apply -f cluster.yaml
#ログ確認コマンド

# pod一覧
kubectl get pods

# ログ確認
kubectl logs ${pod_name}

# service確認
kubectl get service
```

試しに接続確認してみましょう。podを1つ用意して接続してみます。  
Serviceから接続できるはずです。

```shell
kubectl run -i --tty ubuntu --image=ubuntu --restart=Never /bin/bash
apt-get update
apt-get install ruby vim wget redis-tools

# 接続
redis-cli -c -h redis-cluster
```

```text
redis-cluster:6379&gt; set hoge fuga
-> Redirected to slot [1525] located at 10.48.0.7:6379
OK

10.48.0.7:6379>; get hoge
"fuga"</pre>
```

このようにクラスタが自動で生成されて、接続確認することができました。  
ただこの方法だとpodが死んでしまった場合に、redis-cluster側が再配置するのと、
kubernetes側のpod再生成で辻褄が合わなくなるのではと思いました。何か対処が必要になる感じだったので、
redis-trib.rbを使ったクラスタ生成に変えることにしました。

ファイル自体の定義もちょっとイケてないものだし、特性を活かすには物足りないものだったかなと思います。
それでもredis-clusterがどのように構築されるかわかったので良かったと思います。

以上です。

あ、後片付けが必要な方はお忘れなく！

```shell
kubectl delete -f cluster.yaml
gcloud container clusters delete test-cluster
```

■参考

{{< blogcard url="https://qiita.com/sawada_masahiko/items/c58ff2953e04c2956c6f" >}}
