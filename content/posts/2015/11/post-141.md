---
title: '[golang]redisで構造体を扱ってみる'
author: しゃまとん
type: post
date: 2015-11-21T14:15:52+00:00
url: /posts/141
featured_image: /images/posts/2015/11/gopher.jpg
is_comment_form_freeze:
  - on
comment_form_freeze_message:
  - ' '
categories:
  - go
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

goで[redis][1]を使ったデータ永続化の際に、構造体を保存し取り出すメモです。

encoding/jsonを使うことで簡単に扱うことができます。  
redisの操作に関しては[redigo][2]を利用しました。

以下、簡単なサンプルです。

<pre class="lang:default decode:true brush: text; gutter: true ">package main

import (
    "encoding/json"
    "github.com/garyburd/redigo/redis"
    "log"
)

type User struct {
    Id    int32
    Name  string
    Score int32
}

func main() {

    c, err := redis.Dial("tcp", ":6379")
    if err != nil {
        log.Fatal(err)
    }
    defer c.Close()

    // struct to JSON
    user := &User{Id: 1, Name: "name", Score: 2}
    serialized, _ := json.Marshal(user)
    log.Println("serialized : ", string(serialized))

    // set
    c.Do("SET", "test", serialized)

    // get
    data, _ := redis.Bytes(c.Do("GET", "test"))
    log.Println("data : ", data)

    // JSON to struct
    if data != nil {
        deserialized := new(User)
        json.Unmarshal(serialized, deserialized)
        log.Println("deserialized : ", deserialized)
    }

}</pre>

実行結果は下になります。

<pre class="brush: bash; gutter: true">$ go run main.go 
 serialized :  {"Id":1,"Name":"name","Score":2}
 data :  [123 34 73 100 34 58 49 44 34 78 97 109 101 34 58 34 110 97 109 101 34 44 34 83 99 111 114 101 34 58 50 125]
 deserialized :  &{1 name 2}</pre>

以上です。

 [1]: http://redis.io/
 [2]: https://github.com/garyburd/redigo