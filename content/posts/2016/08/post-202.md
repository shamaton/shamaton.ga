---
title: '[Unity] CSVからScriptableObjectを作成してみる'
author: しゃまとん
type: post
date: 2016-08-04T14:49:00+00:00
url: /posts/202
featured_image: /images/posts/2016/06/csv_scriptable.png
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

ExcelからScriptableObjectを生成する記事はあるのですが、CSVからScriptableObjectを生成するものが見当たらなかったのでちょっと実装してみました。

■ 参考  
エクセルからScriptableObjectを生成するやつ



&nbsp;

すみません、コチラの記事はノーコーディングにはなりませぬ。。。  
ただimporterはExcel系の処理をしないので割りとシンプルです。

ScriptableObjectにしたいやつです。

<pre class="brush: csharp; gutter: true">using UnityEngine;
using System.Collections;
using System.Collections.Generic;

// ScriptableObject化するマスタ
public class Data : ScriptableObject { 
  public List&lt;Param&gt; param = new List&lt;Param&gt; ();

  [System.SerializableAttribute]
  public class Param {
    public int      intValue;
    public float    floatValue;
    public string   stringValue;
  }
}</pre>

Importerはこれ。該当のCSV(Data.csv)に変更があると、更新処理が走ります。

<pre class="brush: csharp; gutter: true">using UnityEngine;
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
      Data data = AssetDatabase.LoadAssetAtPath&lt;Data&gt;(exportFile);

      // 見つからなければ作成する
      if (data == null) {
        data = ScriptableObject.CreateInstance&lt;Data&gt;();
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
}</pre>

CSVファイル（Data.csv）はこんな感じで適当に作ります。

<pre class="brush: text; gutter: true">intValue,floatValue,stringValue
1,2.3,text
4,5.6,テキスト
7,8.9,てきすと</pre>

今回の場合、Assets/CSVに上記のファイルを配置すると、importが開始されましてData.assetが生成されます。中身も入ってますね。

[<img src="https://shamaton.orz.hm/blog/images/posts/2016/06/csv_scriptable.png" alt="csv_scriptable" width="278" height="275" class="aligncenter size-full wp-image-204" />][1]

エクセルだと中身がどのように変わったのかわからないので嫌だな〜と思ったりするのですが、何かいい術はないものか。。いろいろあれですね。  
以上です。

&nbsp;

 [1]: https://shamaton.orz.hm/blog/images/posts/2016/06/csv_scriptable.png