---
title: UnityとGoで暗号化してみた
author: しゃまとん
date: 2016-02-08T15:58:47+00:00
url: /posts/142
featured_image: /images/posts/2015/11/gopher.jpg
categories:
  - go
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

Unity環境とGo環境でSHA256で同一ハッシュ値を得るにはどうしたらいいかと、ちょっと試した時のメモです。

どちらも外部のライブラリ等は不要で対応できます。  
これでUnity <&#8211;> Goで同じハッシュ値を得ることができます。

以下、コードと実行結果です。  
Goの方がコード量が少ないです。

■Unity

```csharp
using UnityEngine;
using System.Collections;

public class crypt : MonoBehaviour {

    // Use this for initialization
    void Start () {
    string res = sha256("shamaton.orz.hm", "secret_key");
    Debug.Log (res);
    }   

  private string sha256(string planeStr, string key) {
    // バイト化
    System.Text.UTF8Encoding ue = new System.Text.UTF8Encoding();
    byte[] planeBytes = ue.GetBytes(planeStr);
    byte[] keyBytes   = ue.GetBytes(key);

    // SHA256化
    System.Security.Cryptography.HMACSHA256 sha256 = new System.Security.Cryptography.HMACSHA256(keyBytes);
    byte[] hashBytes = sha256.ComputeHash(planeBytes);

    // 文字列化
    string hashStr = "";
    foreach(byte b in hashBytes) {
      hashStr += string.Format("{0,0:x2}", b);
    }
    return hashStr;
  }
}
```

{{< figure src="/images/posts/2016/02/unity_sha256.png" >}}

■Go

```go
package main

import (
    "crypto/hmac"
    "crypto/sha256"
    "fmt"
)

func main() {
    // SHA256
    hash := hmac.New(sha256.New, []byte("secret_key"))
    hash.Write([]byte("shamaton.orz.hm"))
    hashSum := fmt.Sprintf("%x", hash.Sum(nil))
    fmt.Println(hashSum)
}
```

```shell
$ go run sha.go
9e3103953957435e407752ec0d4b063058a55338c6ac2e15ea4f07c21f90e10c
```


以上です。