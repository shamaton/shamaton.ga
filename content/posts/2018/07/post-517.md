---
title: '[GCP] CloudSQLのデータをCSV出力するには'
author: しゃまとん
date: 2018-07-29T14:06:10+00:00
url: /posts/517
featured_image: /images/posts/2017/12/gcp_logo.png
categories:
  - GCP

---
お世話になっております。  
しゃまとんです。

GCPのマネージドデータベースであるCloudSQLにて、DBからCSV化を行いたいと思いもろもろを調べていたのですが、
ちょっと手こずったのでメモしておきたいと思います。

{{< blogcard url="https://cloud.google.com/sql?hl=ja" >}}

CSV出力はどのようにするかぐぐってみると、よく出てくるのが

`SELECT ... INTO OUTFILE`

でのやり方です。

ですが、これをCloudSQLに対して実行すると

```text
Access denied for user user@cloudsqlproxy~% (using password: YES)
```

ようなエラーとなり拒否されてしまいます。  
これは公式にもあるのですが、CloudSQLではOUTFILEがサポートされていません。

{{< blogcard url="https://cloud.google.com/sql/docs/features?hl=ja">}}

やり方として、CloudStorageに対してエクスポートする手法を公式で説明していますが
いちいち経由するのがちと煩わしいなというところで、別の対処法でCSV化することができます。

やり方は結構シンプルでシェル上でデータを取得（SELECT）して、出力された文字を置換していくだけです。

```shell
mysql -h localhost --protocol TCP -P 3306 \
  -uuser -ppassword \
  -e "select * from user;" | sed -e 's/^/"/g' | sed -e 's/$/"/g' | sed -e 's/\t/","/g' > ./output.csv
```

上記の例では、全ての項目に対して「"」ではさみ、カンマで区切ったものになります。

```text
"id","name"
"1","taro"
"2","jiro"
```

これでシェル上では対応できるのですが、自分はスクリプト化したかったのでGoで同じような処理をするものを書いてみました。

```go
func db2csv(user, pass, host, port,　dataBase, tableName string) {
    
    csvFileName := tableName + ".csv"

    dataSQL := "SELECT * FROM " + tableName
    ret := execCommand(
        "mysql", "-h", host, "--protocol", "TCP", "-P", port, "-u"+user, "-p"+pass, dataBase, "-e", dataSQL,
    )

    scanner := bufio.NewScanner(strings.NewReader(string(ret)))
    var columns []string
    isHeaderChecked := false
    for scanner.Scan() {
        str := ""
        if isHeaderChecked {
            str = "\"" + scanner.Text() + "\""
            str = strings.Replace(str, "\t", "\",\"", -1)
        } else {
            str = strings.Replace(scanner.Text(), "\t", ",", -1)
            isHeaderChecked = true
        }
        columns = append(columns, str)
    }
    columnStr := strings.Join(columns, "\n")

    err := ioutil.WriteFile(csvFileName, []byte(columnStr), 0644)
    if err != nil {
        panic(err)
    }
}

func execCommand(command string, args ...string) []byte {

    path, err := exec.LookPath(command)
    if err != nil {
        panic(err)
    }

    cmd := exec.Command(path, args...)

    var stdErr, stdOut bytes.Buffer
    cmd.Stderr = &stdErr
    cmd.Stdout = &stdOut

    // exec
    err = cmd.Run()
    if err != nil {
        fmt.Println(stdErr.String())
        panic(err)
    }

    return stdOut.Bytes()
}
```

シェルスクリプトと若干違うのですが、最初の行のカラムには「"」をつけないようにしています。
CSVからINSERT文を生成するさいに邪魔になっていたので。

```text
id,name
"1","taro"
"2","jiro"
```

最初は、シェルの処理をそのままGo内で使おうとしていたのですが、なかなかうまくいかなかった（パイプが結構めんどい）ので、
他の機能を使って同じような処理を行いました。

ちなみに、メモリコピーが走ったりするので、簡易的なスクリプト等にとどめていただけると幸いです。  
以上です。

■ 参考

{{< blogcard url="https://qiita.com/kurkuru/items/9daee5e9d18d0a7154d5" >}}