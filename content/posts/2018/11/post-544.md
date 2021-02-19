---
title: '[Unity/Golang] 環境間でデータを圧縮して解凍してみる'
author: しゃまとん
type: post
date: 2018-11-25T13:48:58+00:00
url: /archives/544
featured_image: /wp-content/uploads/2016/03/unity-logo.png
categories:
  - go
  - unity

---
お世話になっております。  
しゃまとんです。

今回は、通信時に圧縮したデータを相互に送り合って使えるようにしたいなと思って試してみました。データを小さくしたりリクエストの回数を減らしたほうがいいですよね。

今回もサーバ側はGolangです。

環境  
Unity2018.1f1  
Go1.10

まずはUnity側です。  
テキストデータを圧縮し、Base64化したものをサーバーにPOSTするものを作りました。  
またサーバーからのレスポンスを受け取って解凍するようにしました。

<pre class="lang:c# decode:true " title="Sample.cs">using System;
using System.Collections;
using System.Text;
using UnityEngine;
using UnityEngine.Networking;

public class Sample : MonoBehaviour {
    
  public void OnButton() {
    StartCoroutine(request());
  }

  IEnumerator request() {
    var message = "send from unity!!";
    var byteMessage = Encoding.UTF8.GetBytes(message);
    var comp = Compressor.Compress(byteMessage);
    var base64 = Convert.ToBase64String(comp, Base64FormattingOptions.InsertLineBreaks);

    WWWForm form = new WWWForm();
    form.AddField("data", base64);

    UnityWebRequest www = UnityWebRequest.Post("http://localhost:8080/", form);
    yield return www.SendWebRequest();

    if (www.isNetworkError) {
      Debug.Log(www.error);
    }
    else {
      var text = www.downloadHandler.text;
      var bytes = Convert.FromBase64String(text);
      var decomp = Compressor.Decompress(bytes);
      var result = Encoding.UTF8.GetString(decomp);
      Debug.Log(result);
    }
  }
}
</pre>

圧縮の処理ですが、今回は外部DLL等を入れることなく使えるものを利用しました。  
簡単ですが、このような感じです。

<pre class="lang:c# decode:true" title="Compressor.cs">using System.IO;
using System.IO.Compression;

public class Compressor {
  
  public static byte[] Compress(byte[] source) {
    MemoryStream ms = new MemoryStream();
    DeflateStream CompressedStream = new DeflateStream(ms, CompressionMode.Compress, true);

    CompressedStream.Write(source, 0, source.Length);
    CompressedStream.Close();
    return ms.ToArray();
  }

  public static byte[] Decompress(byte[] source) {
    MemoryStream ms = new MemoryStream(source);
    MemoryStream ms2 = new MemoryStream();

    DeflateStream CompressedStream = new DeflateStream(ms, CompressionMode.Decompress);

    while (true) {
      int rb = CompressedStream.ReadByte();
      if (rb == -1) {
        break;
      }
      ms2.WriteByte((byte)rb);
    }
    return ms2.ToArray();
  }
}</pre>

次にサーバですが、Unity側と同じ圧縮アルゴリズムを利用して処理を実装しました（当然ですが）。実装の際、結構ハマっていたポイントがあるのですが、なぜかunexpected EOFというエラーが出てしまっていました。原因は、圧縮処理を行うwriterのCloseを行わずしてデータを返却していたことでした。

これが結構厄介なところで、Goだとよく defer writer.Close() とかするのでそれでよいと思い込んでいたのですが、Closeを行ってからじゃないと正しく処理されたデータを取得することができません。陥りがな罠に自分もハマっていました・・・。

処理自体はリクエストが来たらパラメータの解凍とデコードをして、その値プラスで追加メッセージをいれて、エンコードと圧縮をするようなものです。

<pre class="lang:go decode:true " title="server.go">package main

import (
    "bytes"
    "compress/flate"
    "encoding/base64"
    "fmt"
    "io"
    "io/ioutil"
    "log"
    "net/http"
)

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}

func handler(w http.ResponseWriter, r *http.Request) {
    data := r.FormValue("data")

    dec, e := fromBase64(data)
    if e != nil {
        log.Println(e)
        return
    }
    raw, e := decompress(dec)
    if e != nil {
        log.Println(e)
        return
    }

    message := string(raw) + "send from server!!"
    comp, e := compress(message)
    if e != nil {
        log.Println(e)
        return
    }

    fmt.Fprintf(w, "%s", toBase64(comp))
}

func compress(target string) ([]byte, error) {

    var buf bytes.Buffer
    writer, e := flate.NewWriter(&buf, flate.DefaultCompression)
    if e != nil {
        writer.Close()
        return nil, e
    }

    dat := bytes.NewBufferString(target)
    if _, e := io.Copy(writer, dat); e != nil {
        writer.Close()
        return nil, e
    }

    e = writer.Flush()
    if e != nil {
        writer.Close()
        return nil, e
    }

    e = writer.Close()
    if e != nil {
        return nil, e
    }

    return buf.Bytes(), nil
}

func decompress(compressed []byte) ([]byte, error) {
    fr := flate.NewReader(bytes.NewReader(compressed))

    strcuture, e := ioutil.ReadAll(fr)
    if e != nil {
        fr.Close()
        return nil, e
    }

    e = fr.Close()
    if e != nil {
        return nil, e
    }
    return strcuture, nil
}

func fromBase64(text string) ([]byte, error) {
    return base64.StdEncoding.DecodeString(text)
}

func toBase64(data []byte) string {
    return base64.StdEncoding.EncodeToString(data)
}
</pre>

これでサーバ側の起動（go run server.goなど）しておいて、Unityを実行してみます。  
下のような感じで、サーバで文字が追加されたものを解凍することができました。

[<img src="https://shamaton.orz.hm/blog/wp-content/uploads/2018/06/console.png" alt="" width="420" height="75" class="aligncenter size-full wp-image-546" />][1]

これで何か大きなデータをやり取りするときは相互にデータサイズを減らしていけそうです。  
以上です。

■ 参考  
<a href="http://gurizuri0505.halfmoon.jp/20121211/52033" target="_blank" rel="noopener">【C#】文字列->圧縮処理->BASE64->解凍処理->文字列な処理コード</a>  
<a href="https://qiita.com/kenmazsyma/items/65149ac736303238bff6" target="_blank" rel="noopener">GO言語のcompressパッケージで陥りがちな罠</a>

 [1]: https://shamaton.orz.hm/blog/wp-content/uploads/2018/06/console.png