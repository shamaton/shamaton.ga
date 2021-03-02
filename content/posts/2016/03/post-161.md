---
title: '[Unity]LitJsonで入れ子なJsonの使い方'
author: しゃまとん
type: post
date: 2016-03-06T10:31:19+00:00
url: /posts/161
featured_image: /images/posts/2016/01/JSON_logo.png
categories:
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

UnityでLobiを利用したランキングなんぞ使ってみようと画策しております。  
他のサービスでは、サービス側の画面でランキングを表示するような感じですが、LobiではランキングをJSONで取得できるようになっており、ゲーム内に合わせた表示ができるっぽくなっています。

データはこんな感じ。すべてstringで返ってくるようです。（抜粋）

<pre class="brush: text; gutter: false">{
  "id" : "test_ranking",
  "status_code" : "0",
  "result" : {
    "cursor" : "1",
    "limit"  : "10",
    "orders" : [
      {
        "display_score" : "12345ほん",
        "name"          : "こぬこ1",
        "rank"          : "1",
        "score"         : "12345"
      },
      {
        "display_score" : "1234ほん",
        "name"          : "こぬこ2",
        "rank"          : "2",
        "score"         : "1234"
      }
    ],
    "ranking" : {
      "icon"       : "/img/icon/none_appranking_120.png",
      "id"         : "shippo",
      "join_count" : "100",
      "name"       : "しっぽすう"
    },
    "self_order" : {
      "display_score" : "123456ほん",
      "name"          : "こぬこ1",
      "rank"          : "1",
      "score"         : "12345"
    },
    "total_results" : "10"
  }
}</pre>

というところで、自分はLitJsonを使ってJSONの処理をしているのですが、記事ではややこしい構造のJSONを扱っている記事があまりなかったので、調べたついでにメモしておきます。

やり方としては2つ  
・LitJson.JsonData側で各要素にアクセス  
・ToObject時に構造体、クラスに突っ込む  
です。各やり方は下記のとおりです。

■ JsonData型でのアクセス  
ToObjectの時に型を指定しないとJsonData型になり、Mapのように扱えます。

<pre class="brush: csharp; gutter: true">private void keyAccess() {

  TextAsset json = Resources.Load("Json/test") as TextAsset;
  JsonData info = JsonMapper.ToObject(json.text);
  Debug.Log("id : " + info["id"] + " st : " + info["status_code"]);
  Debug.Log("result : " + info["result"]["cursor"]);


  JsonData result = (JsonData)info["result"];

  for (int i = 0; i &lt; result["orders"].Count; i++) {
    Debug.Log(result["orders"][i]["name"]);
  }
}</pre>

■構造体を定義してのアクセス  
ToObject時に指定すると、構造体に展開してくれます。データと型が異なる場合はエラーになります。

<pre class="brush: csharp; gutter: true">private void toStruct() {

  TextAsset json = Resources.Load("Json/test") as TextAsset;

  Response res = JsonMapper.ToObject&lt;Response&gt;(json.text);
  Debug.Log(res.id);
  Debug.Log(res.status_code);
  Debug.Log(res.result.cursor);
  Debug.Log(res.result.limit);
  foreach (Response.Result.User user in res.result.orders) {
    Debug.Log(user.display_score);
  }
  Debug.Log(res.result.ranking.icon);
  Debug.Log(res.result.total_results);
  Debug.Log(res.result.self_order.display_score);

  return "";
}


public struct Response {

  public struct Result {
    public struct User {
      public string display_score { get; private set; }
      public string icon          { get; private set; }
      public string is_self       { get; private set; }
      public string name          { get; private set; }
      public string rank          { get; private set; }
      public string score         { get; private set; }
      public string uid           { get; private set; }
    };

    public struct Ranking {
      public string icon       { get; private set; }
      public string id         { get; private set; }
      public string join_count { get; private set; }
      public string name       { get; private set; }
    }
    public string limit { get; private set; }
    public string cursor { get; private set; }
    public string total_results { get; private set; }
    public List&lt;User&gt; orders { get; private set;}
    public Ranking ranking { get; private set;}
    public User self_order { get; private set;}
  }
  public string id          { get; private set; }
  public string status_code { get; private set; }
  public Result result      { get; private set; }
}</pre>

構造体で定義された変数が存在しない場合は無視されるようですね。  
キャストも必要なく、コードも見やすいので個人的には構造体かなーと思いました。

以上です。