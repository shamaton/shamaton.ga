---
title: 'Cocos2dx@WinでサンプルのJSゲームをビルドする#2'
author: しゃまとん
date: 2013-08-29T05:02:41+00:00
url: /posts/25
categories:
  - cocos2dx
  - プログラミング関連

---
お世話になります。  
しゃまとんです。

ビルドする#1の続きです。  
JSを読みに行くようにCppを修正し、実機転送できるように各所を修正していきます。

以下手順です。

<!--more-->

■Cppファイルの整理  
プロジェクト作成時に、サンプルができているがJSに切り替えるので、HelloWorldScene.*を削除します。  
AppDelegate.cppを既存サンプルのCrystalCrazeからコピーして上書きます。  
場所は`cocos2d-x-2.1.4\samples\Javascript\CrystalCraze\Classes`

■Android.mkの修正  
いろいろとビルド時に必要なものがあるので下記のように修正します。  
これで、プロジェクトのビルドまでは通ります。

```text
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE := game_shared

LOCAL_MODULE_FILENAME := libgame

LOCAL_SRC_FILES := hellocpp/main.cpp \
                   ../../Classes/AppDelegate.cpp

LOCAL_C_INCLUDES := $(LOCAL_PATH)/../../Classes                   

LOCAL_WHOLE_STATIC_LIBRARIES := cocos2dx_static
LOCAL_WHOLE_STATIC_LIBRARIES += cocosdenshion_static
LOCAL_WHOLE_STATIC_LIBRARIES += cocos_extension_static
LOCAL_WHOLE_STATIC_LIBRARIES += chipmunk_static
LOCAL_WHOLE_STATIC_LIBRARIES += spidermonkey_static
LOCAL_WHOLE_STATIC_LIBRARIES += scriptingcore-spidermonkey

LOCAL_EXPORT_CFLAGS := -DCOCOS2D_DEBUG=2 -DCOCOS2D_JAVASCRIPT

include $(BUILD_SHARED_LIBRARY)

$(call import-module,CocosDenshion/android)
$(call import-module,cocos2dx)
$(call import-module,extensions)

$(call import-module,scripting/javascript/spidermonkey-android)
$(call import-module,scripting/javascript/bindings)
```

■ライブラリの追加  
プロジェクト右クリック → プロパティ → C/C++一般 → パスおよびシンボル

タブ：インクルード  
追加時「すべての言語に追加」にチェック  
下記2つを追加する。

```text
${NDK_ROOT}/platforms/android-8/arch-arm/usr/include
${ProjDirPath}/../../cocos2dx/include
```

タブ：ソースロケーション  
リンク・フォルダーを押す。変数ボタンを押して、変数をに新しく追加する。

```text
変数：COCOS2DX_ROOT
値C:\adt-bundle-windows\cocos2d-x-2.1.4\cocos2d-x-2.1.4

変数：PROJECT_ROOT
値：C:\adt-bundle-windows\cocos2d-x-2.1.4\cocos2d-x-2.1.4\(プロジェクトの名前)</pre>

追加したら、ファイル・システム内のフォルダーにリンクをチェックし、下記に通りに2つ入力する。

<pre class="brush: text; gutter: false">PROJECT_ROOT/Classes
COCOS2DX_ROOT/cocos2dx
```

すると、  
Project/Classes  
Project/cocos2dx

というフォルダーがプロジェクト内に作成されている。

■確認  
最後に、jni/hellocpp/main.cppを開いて、エラーが何も無ければOK

これで実行を押すと、実機へのアプリ転送ができます(　´∀｀)bｸﾞｯ!