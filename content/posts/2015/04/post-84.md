---
title: '[cocos2dx]photonを使ってみた – その1'
author: しゃまとん
type: post
date: 2015-04-08T01:25:13+00:00
url: /posts/84
featured_image: /images/posts/2015/08/photon_realtime_turquoise.png
categories:
  - cocos2dx
  - photon
  - プログラミング関連

---
お世話になります。

しゃまとんです。

すこし前ですが、CEDECに行ってきた際にphoton cloudの存在に興味を引かれまして、セッションに参加してきたのですが、あまりの手軽さに驚きました。

ちなみにphotonというのは、オンラインゲーム(マルチプレイヤーゲーム)を簡単に作ることができる、ネットワークエンジンです。  
<a href="http://photoncloud.jp/" target="_blank" rel="noopener">photoncloud.jp</a>

その時はUnityでのチュートリアルだったのですが、cocos2d-xでもやってくれないかなと思っていたところ、GMOでサンプルを記事にしてくださっていました。(SDKはもともとあった)

<a href="http://recruit.gmo.jp/engineer/jisedai/blog/cocos2d-x_photon/" target="_blank" rel="noopener">cocos2d-xでphotonを使ってみよう</a>

上記の記事では、version3のSDKを使っていますが、4がリリースされていたので、今後は4が使われるだろうと思い、version4を使いました。  
合わせて、そのままやっていくと、ビルドできなかったり、Androidでうまく実行できなかったりしたので、ハマった部分をメモしておきます。

cocos2d-xのバージョンはv3.2でした。

<!--more-->

■photonとは　〜　photonダッシュボード

ここまでは、同様です。手順通りにすすめてください。

■photon SDKのダウンロード

こちらはクライアントSDK（v4）をダウンロードします。

[<img src="https://shamaton.orz.hm/blog/images/posts/2015/03/2015_0329_sdk.png" alt="2015_04_sdk" width="603" height="438" class="aligncenter size-full wp-image-86" />][1]

ダウンロード以外の手順は同じです。cocos2d-xでプロジェクトを作成し、SDKのフォルダをリネムして、同じ位置に格納しましょう。

■プロジェクトのセットアップ

iOSのセットアップは１〜３まで行ってください。4の64bitアーキテクチャの除外は設定しません。(2015/02から64bit対応は必須のはず)

Androidの設定ですが、手順のとおりにAndroid.mkを編集してもうまくいかなかったので、下記のように設定しました。主にcall周りの設定を変更しています。

<pre class="brush: text; gutter: true">LOCAL_PATH := $(call my-dir)
LOCAL_PHOTON_ROOT := $(LOCAL_PATH)/../Photon-AndroidNDK_SDK

include $(CLEAR_VARS)

$(call import-add-path,$(LOCAL_PATH)/../../cocos2d)
$(call import-add-path,$(LOCAL_PATH)/../../cocos2d/external)
$(call import-add-path,$(LOCAL_PATH)/../../cocos2d/cocos)

LOCAL_MODULE := cocos2dcpp_shared

LOCAL_MODULE_FILENAME := libcocos2dcpp

LOCAL_SRC_FILES_JNI_PREFIXED := \
    $(wildcard $(LOCAL_PATH)/../../Classes/*.cpp) \
    $(wildcard $(LOCAL_PATH)/../../Classes/**/*.c*)

LOCAL_SRC_FILES_JNI_UNPREFIXED := $(subst jni/,, $(LOCAL_SRC_FILES_JNI_PREFIXED))

LOCAL_SRC_FILES := hellocpp/main.cpp \
                   $(LOCAL_SRC_FILES_JNI_UNPREFIXED)

LOCAL_C_INCLUDES := $(LOCAL_PATH)/../../Classes
LOCAL_C_INCLUDES += $(LOCAL_PATH)/../Photon-AndroidNDK_SDK

LOCAL_WHOLE_STATIC_LIBRARIES := cocos2dx_static
LOCAL_WHOLE_STATIC_LIBRARIES += cocosdenshion_static

# LOCAL_WHOLE_STATIC_LIBRARIES += box2d_static
# LOCAL_WHOLE_STATIC_LIBRARIES += cocosbuilder_static
# LOCAL_WHOLE_STATIC_LIBRARIES += spine_static
# LOCAL_WHOLE_STATIC_LIBRARIES += cocostudio_static
# LOCAL_WHOLE_STATIC_LIBRARIES += cocos_network_static
# LOCAL_WHOLE_STATIC_LIBRARIES += cocos_extension_static

LOCAL_CFLAGS := -DEG_DEBUGGER -D__STDINT_LIMITS -D_EG_ANDROID_PLATFORM

LOCAL_STATIC_LIBRARIES := common-cpp-static-prebuilt
LOCAL_STATIC_LIBRARIES += photon-cpp-static-prebuilt
LOCAL_STATIC_LIBRARIES += loadbalancing-cpp-static-prebuilt

LOCAL_LDLIBS := -llog

LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)

include $(BUILD_SHARED_LIBRARY)

$(call import-module,.)
$(call import-module,audio/android)

# $(call import-module,Box2D)
# $(call import-module,editor-support/cocosbuilder)
# $(call import-module,editor-support/spine)
# $(call import-module,editor-support/cocostudio)
# $(call import-module,network)
# $(call import-module,extensions)

$(call import-add-path-optional, $(LOCAL_PHOTON_ROOT)/Common-cpp/lib)
$(call import-add-path-optional, $(LOCAL_PHOTON_ROOT)/Common-cpp)
$(call import-module,common-cpp-prebuilt)
$(call import-add-path-optional, $(LOCAL_PHOTON_ROOT)/Photon-cpp/lib)
$(call import-add-path-optional, $(LOCAL_PHOTON_ROOT)/Photon-cpp)
$(call import-module,photon-cpp-prebuilt)
$(call import-add-path-optional, $(LOCAL_PHOTON_ROOT)/LoadBalancing-cpp/lib)
$(call import-add-path-optional, $(LOCAL_PHOTON_ROOT)/LoadBalancing-cpp)
$(call import-module,loadbalancing-cpp-prebuilt)</pre>

■サンプルアプリケーション

画面のとおりに表示されるようにします！

<p style="text-align: center;">
  <a href="https://shamaton.orz.hm/blog/images/posts/2015/03/2015_0329_photon_test_exec.png"><img src="https://shamaton.orz.hm/blog/images/posts/2015/03/2015_0329_photon_test_exec.png" alt="photon test exec" width="547" height="358" class="aligncenter wp-image-87" /></a>
</p>

次はコードの修正をします。  
ここまでお疲れ様でした。

 [1]: https://shamaton.orz.hm/blog/images/posts/2015/03/2015_0329_sdk.png