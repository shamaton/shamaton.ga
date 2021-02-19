---
title: '[gRPC] golangでgRPCを使ってみる'
author: しゃまとん
type: post
date: 2018-08-08T13:45:46+00:00
url: /archives/542
featured_image: /wp-content/uploads/2018/08/grpc.svg
categories:
  - go
  - gRPC

---
お世話になっております。  
しゃまとんです。

[gRPC][1]触ってみたいぞ！ってことで、Unity x golang x gRPC を試してみましょう。  
今回はよくある方のgolangです。

まずは必要なパッケージたちを準備していきましょう。  
gRPCはprotoファイルを作成して、そこから各言語毎のソースコードを生成します。  
そのためgolang用のgrpcのダウンロードとprotocコマンドを使えるようにしておきます。

<pre class="lang:default decode:true"># golang用のgRPCパッケージを取得
go get -u google.golang.org/grpc

# protocを使えるようにする
# go get -u github.com/golang/protobuf/protoc-gen-go でもOK（要PATH設定）
brew install protobuf

#　確認
protoc --version
libprotoc 3.*.*</pre>

protocのバージョンは3以上にしましょう（何も考えなければそうなる）

それではGOPATHの通ったどこかに作業用ディレクトリを作成し、まずはprotoファイルを作成します。今回はgrpctestとしました。

<pre class="lang:default decode:true"># 作業用フォルダ作成
cd $GOPATH/src
mkdir grpctest
cd grpctest

# protoファイルの置き場
mkdir helloworld
touch helloworld.proto</pre>

protoファイルは公式と同じ感じです！

<pre class="lang:default decode:true" title="helloworld.proto">syntax = "proto3";

package helloworld;

// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}</pre>

用意できたらコードを生成してみましょう。

<pre class="lang:default decode:true"># goのコードを生成
protoc --go_out=plugins=grpc:. ./helloworld/*.proto

# 確認（pb.goが作成されている）
ls helloworld/
helloworld.pb.go helloworld.proto</pre>

次にclient / serverのコードを作成します。先程生成したpb.goを利用しつつコードを書く感じですね。まずはserverです。

<pre class="lang:go decode:true" title="server.go">package main

import (
    "log"
    "net"

    pb "grpctest/helloworld"
    "golang.org/x/net/context"
    "google.golang.org/grpc"
)

type service struct{}

func (s *service) SayHello(ctx context.Context, req *pb.HelloRequest) (*pb.HelloReply, error) {
    log.Println("call from", req.Name)
    rsp := new(pb.HelloReply)
    rsp.Message = "Hello " + req.Name + "."
    return rsp, nil
}

func main() {

    l, err := net.Listen("tcp", ":9999")
    if err != nil {
        log.Fatalln(err)
    }
    s := grpc.NewServer()
    pb.RegisterGreeterServer(s, &service{})
    s.Serve(l)

}
</pre>

次にclientです。

<pre class="lang:go decode:true" title="client.go">package main

import (
    "log"
    "os"
    "time"

    "golang.org/x/net/context"
    "google.golang.org/grpc"
    pb "grpctest/helloworld"
)

const (
    address     = "localhost:9999"
    defaultName = "world"
)

func main() {
    // Set up a connection to the server.
    conn, err := grpc.Dial(address, grpc.WithInsecure())
    if err != nil {
        log.Fatalf("did not connect: %v", err)
    }
    defer conn.Close()
    c := pb.NewGreeterClient(conn)

    // Contact the server and print out its response.
    name := defaultName
    if len(os.Args) &gt; 1 {
        name = os.Args[1]
    }
    ctx, cancel := context.WithTimeout(context.Background(), time.Second)
    defer cancel()
    r, err := c.SayHello(ctx, &pb.HelloRequest{Name: name})
    if err != nil {
        log.Fatalf("could not greet: %v", err)
    }
    log.Printf("Greeting: %s", r.Message)
}</pre>

それでは実行してみましょう！ターミナルを2つ用意して&#8230;先にserverを実行してからclientを実行してみると下記のように表示されます。

<pre class="lang:default decode:true  "># client
$ go run client.go
2018/08/08 21:50:43 Greeting: Hello world.

$ go run client.go shamaton
2018/08/08 21:50:47 Greeting: Hello shamaton.

# server
$ go run server.go
2018/08/08 21:50:43 call from world
2018/08/08 21:50:47 call from shamaton</pre>

こんな感じでgolang単体で動作するようになりました！  
次はクライアントをUnityにして、動作させてみたいと思います。

以上です。

<https://shamaton.orz.hm/blog/archives/553>

■ 参考  
<a href="https://qiita.com/oohira/items/63b5ccb2bf1a913659d6" target="_blank" rel="noopener">Qiita &#8211; gRPCって何？</a>  
<a href="https://blog.fenrir-inc.com/jp/2016/10/grpc-go.html" target="_blank" rel="noopener">gRPC(Go) で API を実装する</a>  
<a href="https://grpc.io/docs/quickstart/go.html" target="_blank" rel="noopener">Go Quick Start</a>

 [1]: https://grpc.io/