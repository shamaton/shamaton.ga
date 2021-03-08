---
title: '[Unity] CSVからScriptableObjectを作成してみる'
author: しゃまとん
date: 2016-08-04T14:49:00+00:00
url: /posts/202
featured_image: /images/posts/2016/06/csv_scriptable.png
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

ExcelからScriptableObjectを生成する記事はあるのですが、
CSVからScriptableObjectを生成するものが見当たらなかったのでちょっと実装してみました。

■ 参考  
エクセルからScriptableObjectを生成するやつ

{{< blogcard url="http://tsubakit1.hateblo.jp/entry/20131010/1381411760" >}}

すみません、コチラの記事はノーコーディングにはなりませぬ。。。  
ただimporterはExcel系の処理をしないので割りとシンプルです。

ScriptableObjectにしたいやつです。

```csharp
using UnityEngine;
using System.Collections;
using System.Collections.Generic;

// ScriptableObject化するマスタ
public class Data : ScriptableObject { 
  public List<Param> param = new List<Param> ();

  [System.SerializableAttribute]
  public class Param {
    public int      intValue;
    public float    floatValue;
    public string   stringValue;
  }
}
```

Importerはこれ。該当のCSV(Data.csv)に変更があると、更新処理が走ります。

```csharp
using UnityEngine;
using UnityEditor;
using System.Collections;
using System.IO;

public class CSV_Impoter : AssetPostprocessor {

  static void OnPostprocessAllAssets(string[] importedAssets, string[] deletedAssets, string[] movedAssets, string[] movedFromAssetPaths) {

    string targetFile = "Assets/CSV/Data.csv";
    string exportFile = "Assets/CSV/Data.asset";

    foreach (string asset in importedAssets) {

      // 合致しないものはスルー
      if (!targetFile.Equals(asset)) continue;

      // 既存のマスタを取得
      Data data = AssetDatabase.LoadAssetAtPath<Data>(exportFile);

      // 見つからなければ作成する
      if (data == null) {
        data = ScriptableObject.CreateInstance<Data>();
        AssetDatabase.CreateAsset((ScriptableObject)data, exportFile);
      }
      // 中身を削除
      data.param.Clear();

      // CSVファイルをオブジェクトへ保存
      using (StreamReader sr = new StreamReader(targetFile)) {

        // ヘッダをやり過ごす
        sr.ReadLine();

        // ファイルの終端まで繰り返す
        while (!sr.EndOfStream) {
          string   line     = sr.ReadLine();
          string[] dataStrs = line.Split(',');

          // 追加するパラメータを生成
          Data.Param p = new Data.Param();
          // 値を設定する
          p.intValue    = int.Parse(dataStrs[0]);
          p.floatValue  = float.Parse(dataStrs[1]);
          p.stringValue = dataStrs[2];
          // 追加
          data.param.Add(p);
        }
      }

      // 保存
      AssetDatabase.SaveAssets();

      Debug.Log("Data updated.");
    }
  }
}
```

CSVファイル（Data.csv）はこんな感じで適当に作ります。

```text
intValue,floatValue,stringValue
1,2.3,text
4,5.6,テキスト
7,8.9,てきすと
```

今回の場合、Assets/CSVに上記のファイルを配置すると、importが開始されましてData.assetが生成されます。
中身も入ってますね。

{{< figure src="/images/posts/2016/06/csv_scriptable.png" >}}

エクセルだと中身がどのように変わったのかわからないので嫌だな〜と思ったりするのですが、
何かいい術はないものか。。いろいろあれですね。  
以上です。
