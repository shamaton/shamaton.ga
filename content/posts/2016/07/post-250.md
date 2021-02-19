---
title: '[Unity] Shaderで波紋のようなものを出してみる（2Dだよ）'
author: しゃまとん
type: post
date: 2016-07-18T02:17:58+00:00
url: /archives/250
featured_image: /wp-content/uploads/2016/07/ripple_eye.gif
categories:
  - shader
  - unity
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

UnityのShader初心者でございます。ちょっとエフェクト作ってみようかなということで、触っているのですが、ちょっと勝手が違っていて調べたり試行錯誤が大変ですね。  
UnityでShader頑張ってみた系の記事を作ってくださっている方に感謝しきりです。

で、何か波打ってるようなものを作ろうとということで波紋的なものを出せないかなということをやっていて、なんとかできたのでメモしておきます。

ちなみにこの記事ですが3Dの波紋表現ではないので、ご注意くださいませ。

それでは、Shaderはこんな感じです。

<pre class="brush: text; gutter: true">Shader "Custom/Sprite-Ripple" {
  Properties
  {
    [PerRendererData] _MainTex ("Sprite Texture", 2D) = "white" {}
    _Color ("Tint", Color) = (1,1,1,1)
    [MaterialToggle] PixelSnap ("Pixel snap", Float) = 0
  }

  SubShader
  {
    Tags
    { 
      "Queue"="Transparent" 
      "IgnoreProjector"="True" 
      "RenderType"="Transparent" 
      "PreviewType"="Plane"
      "CanUseSpriteAtlas"="True"
    }

    Cull Off
    Lighting Off
    ZWrite Off
    Fog { Mode Off }
    Blend SrcAlpha OneMinusSrcAlpha

    Pass
    {
    CGPROGRAM
      #pragma vertex vert
      #pragma fragment frag
      #pragma multi_compile DUMMY PIXELSNAP_ON
      #include "UnityCG.cginc"
      
      struct appdata_t
      {
        float4 vertex   : POSITION;
        float4 color    : COLOR;
        float2 texcoord : TEXCOORD0;
      };

      struct v2f
      {
        float4 vertex   : SV_POSITION;
        fixed4 color    : COLOR;
        half2 texcoord  : TEXCOORD0;
      };
      
      fixed4 _Color;

      v2f vert(appdata_t IN)
      {
        v2f OUT;
        OUT.vertex = mul(UNITY_MATRIX_MVP, IN.vertex);
        OUT.texcoord = IN.texcoord;
        OUT.color = IN.color * _Color;
        #ifdef PIXELSNAP_ON
        OUT.vertex = UnityPixelSnap (OUT.vertex);
        #endif

        return OUT;
      }

      sampler2D _MainTex;

      fixed4 frag(v2f_img IN) : COLOR
      {
        half4 tex = tex2D(_MainTex, IN.uv);

        float c = 120.0;
        float t = _Time.x * c;
        float s = abs(sin(-t + c * length(float2(IN.uv.x - 0.5, IN.uv.y - 0.5))));
        tex.rgb = red * s + tex.rgb * (1-s);
        return tex;
      }
    ENDCG
    }
  }
}</pre>

波紋のような表示をさせる部分はフラグメントシェーダーの部分のみ（frag関数）で他の部分は何もいじっていません。

波紋にはsin関数を使って、その波をそのまま利用しています。適当に係数（c）をつけることで変化を細かくするようにしています。  
sin関数の引数にはテクスチャの中心点から波が発生していくような数値を設定しています。

<pre class="brush: text; gutter: false">length(float2(IN.uv.x - 0.5, IN.uv.y - 0.5))</pre>

この部分で中心点でるように計算されています。

シェーダを作成後、マテリアルを生成して適当なSpriteに設定します。  
実行するとこんな感じになります。

[<img src="http://shamaton.orz.hm/blog/wp-content/uploads/2016/07/ripple_eye.gif" alt="ripple_eye" width="198" height="197" class="aligncenter size-full wp-image-251" />][1]

なんか洗脳されそうですねｗ  
行数にしたら数行ですが、これだけで出来ちゃうShaderすごい。もっと色々な表現できるようになりたいものです。

以上です。

■ UnityのShader関連  
<a href="http://qiita.com/edo_m18/items/591925d7fc960d843afa" target="_blank">[Unity] シェーダで使える定義済値</a>  
[Unity のシェーダの基礎を勉強してみたのでやる気出してまとめてみた][2]

■ UnityのShaderじゃないけど勉強になります！[  
[連載]やってみれば超簡単！ WebGL と GLSL で始める、はじめてのシェーダコーディング（１）][3]

 [1]: http://shamaton.orz.hm/blog/wp-content/uploads/2016/07/ripple_eye.gif
 [2]: http://tips.hecomi.com/entry/2014/03/16/233943
 [3]: http://qiita.com/doxas/items/b8221e92a2bfdc6fc211