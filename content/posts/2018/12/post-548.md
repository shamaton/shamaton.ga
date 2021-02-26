---
title: '[Unity/Golang]データ圧縮にdeflateが使えなかった'
author: しゃまとん
type: post
date: 2018-12-18T15:03:44+00:00
url: /posts/548
featured_image: /images/posts/2016/03/unity-logo.png
categories:
  - go
  - unity

---
お世話になっております。  
しゃまとんです。

以前にデータ圧縮の記事を書いたのですが、

{{< blogcard url="/posts/544">}}

どうやら、Androidで利用しようとするとエラーになってしまうようです。  
よって、モバイル向けに利用する際には注意が必要でした。

Unityでは（私個人が）大体モバイル向けにビルドするため、
これだと使えないということで別の手を使って圧縮を行うように変更することにしました。  
deflateの代わりとしては、DotNetZipやMessagePackなども使えそうですが今回は[SharpZibLib][1]を使ってみることにしました。

ちなみにライセンスはMITです。以前はGPLだったようですが変わったみたいですね。

導入にはこちらのサイトがとても参考になります。

{{< blogcard url="http://baba-s.hatenablog.com/entry/2017/08/24/100000">}}

ということで、以前のコードを書き直しました。  
Unity側はDeflateを使っていた箇所を置き換えるだけです。

```csharp
using System.IO;
using ICSharpCode.SharpZipLib.GZip;

public class Compressor {
  
  public static byte[] Compress(byte[] source) {
    MemoryStream ms = new MemoryStream();
    var CompressedStream = new GZipOutputStream(ms);

    CompressedStream.Write(source, 0, source.Length);
    CompressedStream.Close();
    return ms.ToArray();
  }

  public static byte[] Decompress(byte[] source) {
    MemoryStream ms = new MemoryStream(source);
    MemoryStream ms2 = new MemoryStream();

    var CompressedStream = new GZipInputStream(ms);

    while (true) {
      int rb = CompressedStream.ReadByte();
      if (rb == -1) {
        break;
      }
      ms2.WriteByte((byte)rb);
    }
    return ms2.ToArray();
  }
}
```

サーバー側も下記のように修正しておきます。

```go
func compress(target string) ([]byte, error) {

    var buf bytes.Buffer
    writer := gzip.NewWriter(&buf)

    dat := bytes.NewBufferString(target)
    if _, e := io.Copy(writer, dat); e != nil {
        writer.Close()
        return nil, e
    }

    e := writer.Flush()
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
    fr, e := gzip.NewReader(bytes.NewReader(compressed))
    if e != nil {
        fr.Close()
        return nil, e
    }

    d, e := ioutil.ReadAll(fr)
    if e != nil {
        fr.Close()
        return nil, e
    }

    e = fr.Close()
    if e != nil {
        return nil, e
    }
    return d, nil
}
```

これでAndroidでも使えるようになりました。  
めでたしめでたし。以上です。

 [1]: https://icsharpcode.github.io/SharpZipLib/