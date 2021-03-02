---
title: 'Cocos2dx@WinでサンプルのJSゲームをビルドする#1'
author: しゃまとん
type: post
date: 2013-08-23T16:56:33+00:00
url: /posts/24
categories:
  - cocos2dx
  - プログラミング関連

---
お世話になります。  
しゃまとんです。

この前コチラにて開発環境を作成を備忘録として記載しました。  
[Cocos2d-xのAndroidアプリ開発環境をWindowsに作成する][1]  
とあるところで、JavaScript bindingな開発ができるようになったほうがいいよ！との意見をいただきました。

その意見を踏まえ、やってみるか！となったら、create-project.batを実行してもCpp用のファイルしか生成できない…  
ということで、サンプルで入っているJSプロジェクトを使って何とか出来ないかと模索しました。

いろいろと試行錯誤したので、その時の備忘録です。  
もしかしたら技術書にのっててサクッできることかも(´・ω・｀)

以下、自分の作ったプロジェクトでJSゲーム制作環境を用意する手順です。

<!--more-->

■下準備

今回はサンプルJSゲームとしてはいっているCrystalCrazeを使います。(cocos2dxのバージョンは2.1.4)  
cocos2d-x-2.1.4/samples/Javascript/Shared/games/CrystalCraze/Published-Androidを  
cocos2d-x-2.1.4/MyScript/TestGameとしてコピーします。

■プロジェクト作成・インポート  
create-android-project.batを実行し、通常通りプロジェクトを作成。  
作成したら、adb bundleを立ち上げ、Androidプロジェクトとしてインポートします。  
メニュー　→　ファイル　→　インポート　→　Existing Android&#8230;

■C/C++プロジェクト変換  
ビルドできるように変換します。  
エクスプローラーウインドウのプロジェクトを右クリック　→　新規　→　C/C++プロジェクトへ変換

■ビルド設定  
プロジェクト右クリック　→プロパティ　→　C/C++ビルド  
「デフォルトビルドコマンドを使用する」のチェックを外し、ビルド・コマンドを下記に設定。

<pre class="brush: text; gutter: false">bash ${ProjDirPath}/build_native.sh</pre>

ビルドに必要な変数を追加します。  
プロジェクト右クリック　→プロパティ　→　C/C++ビルド　→　環境

<pre class="brush: text; gutter: false">変数：NDK_ROOT
値：C:\adt-bundle-windows\android-ndk-r8e</pre>

■icon.pngの追加(エラー対処)  
iconがないよ！とエラーができるので、アイコンを用意して、res/drawable-xxx辺りに追加しておく。  
コンソールからエラー表示が消えるのを確認。

■Application.mk(NDKエラー対処)  
下記を最下行に追記する。

<pre class="brush: text; gutter: true">APP_PLATFORM := android-8</pre>

■build_native.shの編集  
以下の用に変更。JSファイルをビルド時にassetsに持ってくるようにする。  
デフォルトのbuild_native.shに加筆なので、もっと整理できるかも。

<pre class="brush: text; gutter: true">APPNAME="アプリ名(おそらくプロジェクト名)"

# options

buildexternalsfromsource=
PARALLEL_BUILD_FLAG=

usage(){
cat &lt;&lt; EOF
usage: $0 [options]

Build C/C++ code for $APPNAME using Android NDK

OPTIONS:
-s	Build externals from source
-h	this help
EOF
}

while getopts "sh" OPTION; do
case "$OPTION" in
s)
buildexternalsfromsource=1
;;
p)
PARALLEL_BUILD_FLAG=\-j8
;;
h)
usage
exit 0
;;
esac
done

# exit this script if any commmand fails
set -e

# paths

if [ -z "${NDK_ROOT+aaa}" ];then
echo "please define NDK_ROOT"
exit 1
fi

DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
# ... use paths relative to current directory
COCOS2DX_ROOT="$DIR/../.."
APP_ROOT="$DIR/.."
APP_ANDROID_ROOT="$DIR"
RESROUCE_ROOT="$APP_ROOT/../MyScript/TestGame"
BINDINGS_JS_ROOT="$APP_ROOT/../scripting/javascript/bindings/js"

echo "----------- Paths -----------------"
echo "NDK_ROOT         = $NDK_ROOT"
echo "COCOS2DX_ROOT    = $COCOS2DX_ROOT"
echo "APP_ROOT         = $APP_ROOT"
echo "APP_ANDROID_ROOT = $APP_ANDROID_ROOT"
echo "RESROUCE_ROOT    = $RESROUCE_ROOT"
echo "BINDINGS_JS_ROOT = $BINDINGS_JS_ROOT"
echo "-----------------------------------"

# Debug
# set -x

# make sure assets is exist
if [ -d "$APP_ANDROID_ROOT"/assets ]; then
    rm -rf "$APP_ANDROID_ROOT"/assets
fi

mkdir "$APP_ANDROID_ROOT"/assets

# copy resources
for file in "$APP_ROOT"/Resources/*
do
if [ -d "$file" ]; then
    cp -rf "$file" "$APP_ANDROID_ROOT"/assets
fi

if [ -f "$file" ]; then
    cp "$file" "$APP_ANDROID_ROOT"/assets
fi
done

# copy project js files
cp -rf "$RESROUCE_ROOT"/* "$APP_ANDROID_ROOT"/assets

# copy bindings/*.js into assets&#039; root
cp -f "$BINDINGS_JS_ROOT"/*.js "$APP_ANDROID_ROOT"/assets

# copy icons (if they exist)
file="$APP_ANDROID_ROOT"/assets/Icon-72.png
if [ -f "$file" ]; then
	cp "$file" "$APP_ANDROID_ROOT"/res/drawable-hdpi/icon.png
fi
file="$APP_ANDROID_ROOT"/assets/Icon-48.png
if [ -f "$file" ]; then
	cp "$file" "$APP_ANDROID_ROOT"/res/drawable-mdpi/icon.png
fi
file="$APP_ANDROID_ROOT"/assets/Icon-32.png
if [ -f "$file" ]; then
	cp "$file" "$APP_ANDROID_ROOT"/res/drawable-ldpi/icon.png
fi

if [[ "$buildexternalsfromsource" ]]; then
    echo "Building external dependencies from source"
    "$NDK_ROOT"/ndk-build -C "$APP_ANDROID_ROOT" $* \
        "NDK_MODULE_PATH=${COCOS2DX_ROOT}:${COCOS2DX_ROOT}/cocos2dx/platform/third_party/android/source"
else
    echo "Using prebuilt externals"
    "$NDK_ROOT"/ndk-build -C "$APP_ANDROID_ROOT" $* \
        "NDK_MODULE_PATH=${COCOS2DX_ROOT}:${COCOS2DX_ROOT}/cocos2dx/platform/third_party/android/prebuilt"
fi</pre>

ここまでで、既存のCppファイルでビルドが通るようになる。そのときに必要なJSファイルをassetsに持ってくるようになる。  
次はAppDelegate.cppからJSファイルを読み込んで、実機転送できるようにする。

 [1]: http://shamaton.orz.hm/blog/posts/21