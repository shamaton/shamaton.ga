---
title: MySQLの空コミットが気になったので調べてみた
author: しゃまとん
date: 2015-11-07T09:19:33+00:00
url: /posts/136
featured_image: /images/posts/2015/11/mysql_logo.png
categories:
  - go
  - mysql

---
お世話になっております。  
しゃまとんです。

webの開発をやっていると、データベースは切っても切れないものだと思います。  
自分はMySQLを使うことが多いのですが、最近MySQLを使った開発を行っているときにトランザクション管理ちゃんと
しないとなと思い考えていた時、

MySQLで空コミットしたときって処理重くなるのかな・・・？  
と思い、すこし調べてみました。  
（webでググっても、サクッとでてこなかった）

簡単に処理を何個か作成しループを100回させてみました。

申し訳ないですが、自分はそこまで精通している人間ではないので、参考程度に読んでいただけると幸いです。  
言語にはgoを使っています。

```go
package main

import (
	"database/sql"
	"log"
	"time"

	_ "github.com/go-sql-driver/mysql"
)

type User struct {
	Id    int32
	Name  string
	Score int32
}

const loop = 100

func main() {
	cnn, _ := sql.Open("mysql", "user:pass@tcp(localhost:port)/database")
	defer cnn.Close()

	// txなしselect
	f1 := func() {
		var user User
		if err := cnn.QueryRow("SELECT id, name, score FROM user WHERE id = ? LIMIT 1", 1).Scan(&user.Id, &user.Name, &user.Score); err != nil {
			log.Fatal(err)
		}
	}

	latency1 := exec(f1)
	log.Println("f1 : ", latency1)

	// txありselect
	f2 := func() {
		tx, _ := cnn.Begin()
		var user User
		if err := tx.QueryRow("SELECT id, name, score FROM user WHERE id = ? LIMIT 1", 1).Scan(&user.Id, &user.Name, &user.Score); err != nil {
			log.Fatal(err)
		}
		tx.Commit()
	}

	latency2 := exec(f2)
	log.Println("f2 : ", latency2)

	// updateレコード変更なし
	f3 := func() {
		tx, _ := cnn.Begin()
		if _, err := tx.Exec("UPDATE user SET score = score WHERE id = ?", 1); err != nil {
			log.Fatal(err)
		}
		tx.Commit()
	}

	latency3 := exec(f3)
	log.Println("f3 : ", latency3)

	// update レコード変更ありrollback
	f4 := func() {
		tx, _ := cnn.Begin()
		if _, err := tx.Exec("UPDATE user SET score = score + 1 WHERE id = ?", 1); err != nil {
			log.Fatal(err)
		}
		tx.Rollback()
	}

	latency4 := exec(f4)
	log.Println("f4 : ", latency4)

	// update レコード変更ありcommit
	f5 := func() {
		tx, _ := cnn.Begin()
		if _, err := tx.Exec("UPDATE user SET score = score + 1 WHERE id = ?", 1); err != nil {
			log.Fatal(err)
		}
		tx.Commit()
	}

	latency5 := exec(f5)
	log.Println("f5 : ", latency5)

}

func exec(f func()) time.Duration {

	t := time.Now()

	for i := 0; i &lt; loop; i++ {
		f()
	}

	latency := time.Since(t)
	return latency

}
```

結果は下記のようになりました。  
見た感じですと

select(txなし) << select(txあり) <= update(空コミット) << update(commit) <= update(rollback)

となりました。selectの速度差はトランザクション開始の処理が入っているので当然なのですが、
oracleの空コミットと同様に何も処理されない(?)ので、select(txあり)とほぼ変わらないっぽいですね。

勉強になりました。

```text
 go run main.go 
f1 :  32.243891ms
f2 :  46.160667ms
f3 :  50.834508ms
f4 :  83.56319ms
f5 :  76.303116ms
---
f1 :  33.905533ms
f2 :  46.795762ms
f3 :  54.755387ms
f4 :  84.56939ms
f5 :  74.385792ms
---
f1 :  33.64808ms
f2 :  46.192082ms
f3 :  52.673612ms
f4 :  85.065523ms
f5 :  80.058726ms
---
f1 :  32.868818ms
f2 :  51.224252ms
f3 :  51.798962ms
f4 :  81.424738ms
f5 :  71.117165ms
---
f1 :  33.805161ms
f2 :  45.889625ms
f3 :  50.216731ms
f4 :  78.674484ms
f5 :  78.635478ms
```