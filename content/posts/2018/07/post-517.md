---
title: '[GCP] CloudSQLのデータをCSV出力するには'
author: しゃまとん
type: post
date: 2018-07-29T14:06:10+00:00
url: /archives/517
featured_image: /wp-content/uploads/2017/12/gcp_logo.png
categories:
  - GCP

---
お世話になっております。  
しゃまとんです。

GCPのマネージドデータベースであるCloudSQLにて、DBからCSV化を行いたいと思いもろもろを調べていたのですが、ちょっと手こずったのでメモしておきたいと思います。

CSV出力はどのようにするかぐぐってみると、よく出てくるのが

SELECT &#8230; INTO OUTFILE

でのやり方です。

ですが、これをCloudSQLに対して実行すると

<pre class="lang:default decode:true ">Access denied for user user@cloudsqlproxy~% (using password: YES)</pre>

ようなエラーとなり拒否されてしまいます。  
これは公式にもあるのですが、CloudSQLではOUTFILEがサポートされていません。

<a href="https://cloud.google.com/sql/docs/features?hl=ja" target="_blank" rel="noopener">Cloud SQL の特長</a>

やり方として、CloudStorageに対してエクスポートする手法を公式で説明していますがいちいち経由するのがちと煩わしいなというところで、別の対処法でCSV化することができます。

やり方は結構シンプルでシェル上でデータを取得（SELECT）して、出力された文字を置換していくだけです。

<pre class="lang:default decode:true ">mysql -h localhost --protocol TCP -P 3306 -uuser -ppassword -e "select * from user;" | sed -e 's/^/"/g' | sed -e 's/$/"/g' | sed -e 's/\t/","/g' &gt; ./output.csv</pre>

上記の例では、全ての項目に対して「&#8221;」ではさみ、カンマで区切ったものになります。

<pre class="lang:default decode:true">"id","name"
"1","taro"
"2","jiro"</pre>

これでシェル上では対応できるのですが、自分はスクリプト化したかったのでGoで同じような処理をするものを書いてみました。

<pre class="lang:go decode:true">func db2csv(user, pass, host, port,　dataBase, tableName string) {
    
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
}</pre>

シェルスクリプトと若干違うのですが、最初の行のカラムには「&#8221;」をつけないようにしています。CSVからINSERT文を生成するさいに邪魔になっていたので。

<pre class="lang:default decode:true ">id,name
"1","taro"
"2","jiro"</pre>

最初は、シェルの処理をそのままGo内で使おうとしていたのですが、なかなかうまくいかなかった（パイプが結構めんどい）ので、他の機能を使って同じような処理を行いました。

ちなみに、メモリコピーが走ったりするので、簡易的なスクリプト等にとどめていただけると幸いです。  
以上です。

■ 参考  
<a href="https://qiita.com/kurkuru/items/9daee5e9d18d0a7154d5" target="_blank" rel="noopener">MySQLのCSV出力が権限によって出来ない場合の対処法</a>  
<a href="http://qiita.com/yuroyoro/items/9358cd25b5f7fe9dd37f" target="_blank" rel="noopener">Goで外部コマンドをパイプして実行する</a>