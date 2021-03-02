---
title: assimp2jsonを使えるようにするまで
author: しゃまとん
type: post
date: 2016-01-06T15:34:37+00:00
url: /posts/149
featured_image: /images/posts/2016/01/JSON_logo.png
categories:
  - assimp
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

テストで実装してみたいことがあり、assimp2jsonというツールが必要になったので準備していたのですが、使えるようになるまでの備忘録です。

cmakeというものを知らなかったのですが、当たり前な内容かもしれません。  
cmake便利ですね！<a href="https://ja.wikipedia.org/wiki/CMake" target="_blank">そもそもcmakeとは</a>

<a href="https://github.com/acgessler/assimp2json" target="_blank">assimp2json</a>とは3DのファイルをJSON(assimp)に変換してくれるものです。  
3Dを何かしらやるやつですね。

はい。それではcmakeから使えるようにしていきます。  
<https://cmake.org/download/> からOSにあったものを選びます。（今回はmacのdmg）  
実行して、Applicationsに入れるだけ。

今回はコマンドラインで実行するので、Applicationsのappからリンクをはります。

<pre class="brush: bash; gutter: true">sudo ln -s /Applications/CMake.app/Contents/bin/ccmake /usr/bin/ccmake
sudo ln -s /Applications/CMake.app/Contents/bin/cmake /usr/bin/cmake
sudo ln -s /Applications/CMake.app/Contents/bin/cmake-gui /usr/bin/cmake-gui
sudo ln -s /Applications/CMake.app/Contents/bin/cmakexbuild /usr/bin/cmakexbuild
sudo ln -s /Applications/CMake.app/Contents/bin/cpack /usr/bin/cpack
sudo ln -s /Applications/CMake.app/Contents/bin/ctest /usr/bin/ctest</pre>

次にassimp2jsonをビルドします。まずはgithubからcloneします。  
任意のディレクトリでやればOK。

<pre class="brush: bash; gutter: true">git clone git@github.com:acgessler/assimp2json.git</pre>

cloneした状態だとassimp2json/assimpが空になっています。assimp以下はsubmoduleとして設定されているので、取得します。

<pre class="brush: bash; gutter: true">cd assimp2json
git submodule init
git submodule update</pre>

これを行うと commit : 93bb63fdb40d9682e60ca97b0eda4951a552c742 の状態のassimpがcloneされます。

これでビルドの準備が整ったので、topに移動して下記のコマンドを実行します。

<pre class="brush: bash; gutter: true">cmake CMakeLists.txt -G &#039;Unix Makefiles&#039; # makefile生成</pre>

実行すると下記のように表示されます（はず）

<pre class="brush: text; gutter: true">-- The C compiler identification is AppleClang 7.0.0.7000176
-- The CXX compiler identification is AppleClang 7.0.0.7000176
-- Check for working C compiler: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/cc
-- Check for working C compiler: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/c++
-- Check for working CXX compiler: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Found PkgConfig: /usr/local/bin/pkg-config (found version "0.28")
-- Building a non-boost version of Assimp.
-- Looking for ZLIB...
-- Checking for module &#039;zzip-zlib-config&#039;
--   Package &#039;zzip-zlib-config&#039; not found
-- Found ZLIB: optimized;/usr/lib/libz.dylib;debug;/usr/lib/libz.dylib
-- Checking for module &#039;minizip&#039;
--   Package &#039;minizip&#039; not found
-- Looking for pthread.h
-- Looking for pthread.h - found
-- Looking for pthread_create
-- Looking for pthread_create - found
-- Found Threads: TRUE
-- Configuring done
CMake Warning (dev):
  Policy CMP0042 is not set: MACOSX_RPATH is enabled by default.  Run "cmake
  --help-policy CMP0042" for policy details.  Use the cmake_policy command to
  set the policy and suppress this warning.

  MACOSX_RPATH is not specified for the following targets:

   assimp

This warning is for project developers.  Use -Wno-dev to suppress it.

-- Generating done
-- Build files have been written to: /Users/xxxxxxxx/Downloads/assimp2json</pre>

makeします。

<pre class="brush: bash; gutter: true">$ make
Scanning dependencies of target assimp
[  0%] Building CXX object assimp/code/CMakeFiles/assimp.dir/Assimp.cpp.o
[  0%] Building CXX object assimp/code/CMakeFiles/assimp.dir/BaseImporter.cpp.o
[  1%] Building CXX object assimp/code/CMakeFiles/assimp.dir/BaseProcess.cpp.o

... ビルドが続く ...

[ 99%] Building CXX object assimp/test/CMakeFiles/unit.dir/unit/utNoBoostTest.cpp.o
[100%] Linking CXX executable ../../bin/unit
[100%] Built target unit</pre>

ビルドが通ると bin配下にassimp2jsonが作られます。

実行できるか簡単にテストしてみます。こちら(<https://github.com/golang-samples/gopher-3d>)からデータを取得します。

<pre class="brush: bash; gutter: true">git clone git@github.com:golang-samples/gopher-3d.git
assimp2json gopher.stl &gt; gopher.json</pre>

PATHを通してない場合、bin配下を指定してください。  
エディタ等でひらくと結構長めなjsonが記載されていれば変換されています。

以上です。

モデルデータは下記のサイト等で確認できます。(github上でも見れますが）  
<http://fablabshibuya.org/applications/3dviewer/>

&nbsp;