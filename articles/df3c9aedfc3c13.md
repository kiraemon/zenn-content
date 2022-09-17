---
title: "VSCodeでモダン風C++開発環境構築 #1"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Cpp", "docker", "VSCode", "clang", "gxx"]
published: true
---

# 概要

初投稿です。普段 C++案件に関わっているので、
VSCode(Visual Studio Code) + Remote Containers でモダン風 C++開発環境構築してみます。

# 目標

M1 Mac 上に以下イメージ図のような環境を構築します。
既知の情報ですので詳細は省きますが、こうすることでホストの環境を汚さずに済む、環境の共有やスクラップ&ビルドが用意になる等様々なメリットがあります。

将来的には本環境をベースとして、

- CMake によるプロジェクト作成
- clang-format によるフォーマット設定
- gtest/gmock によるテスト環境構築

などなど説明していければと思います。
C++バージョンは C++17 を予定しています。

個人環境が Mac なので MacOS 上の説明になりますが、コマンドを適宜読み替えていただければ Windows 環境でも可能です。

## イメージ図

![](/images/df3c9aedfc3c13/1.jpg)

# 環境構築

## 各種ツールのインストール

以下構築の土台となるツールのインストールを行います。

- [VS Code](https://azure.microsoft.com/ja-jp/products/visual-studio-code/)
- [Docker Hub](https://hub.docker.com/)
  - Docker CE でも可能です。

## コンテナ立ち上げ

### 拡張機能(Extension)のインストール

インストールが終わりましたら VS Code を起動します。

1. 起動したら左端タブから Extensions を選択
2. 左上の検索窓から"Remote"と入力
3. "Remode - Containers" をインストール

![](/images/df3c9aedfc3c13/2.png)

### Dockerfile の作成

以下のように適当なフォルダ(ここでは cpp_test)を作って VS Code で Open Folder します。
フォルダが開いたら、以下のようにフォルダ直下へ Dockerfile を作成します。
![](/images/df3c9aedfc3c13/3.png)

以下の Dockerfile からビルド環境を生成します。

```yml:Dockerfile
# ベースとなるOSはubuntu 20.04を選択
FROM ubuntu:20.04

# パッケージの一覧更新
RUN apt-get update

# タイムゾーンの設定
RUN apt install -y tzdata
ENV TZ=Asia/Tokyo

# 開発環境のシステムインストール
RUN apt install -y wget \
  g++ \
  cmake \
  git \
  clang-format
```

ここでは、C++開発に必要な以下ツールをインストールします。

- コンパイラ：g++
- ビルド支援：cmake
- コード管理：git
- フォーマッタ：clang-format
- (一応)ダウンローダー：wget

タイムゾーンの設定は以下を参考にさせて頂きました。ありがとうございます。
[[Docker] build tzdata タイムゾーン選択回避方法(ubuntu)](https://sleepless-se.net/2018/07/31/docker-build-tzdata-ubuntu/)

### コンテナ立ち上げ

それでは、先ほど作った Dockerfile からコンテナ生成します。
左下の緑色><マークをクリックすると選択肢のリストが開くので、
Reopen in Container を選択してください。
(command + P から Remote Container を選択しても OK です)

![](/images/df3c9aedfc3c13/4.png)

先ほど作成した Dockerfile から生成したいので、`From Dockerfile`を選択します。

![](/images/df3c9aedfc3c13/5.png)

初回のコンテナ立ち上げが始まりますので少し待ちます。
立ち上げが終わりましたら、既にコンテナへの接続が行われておりますので左下が`Dev Container: Existing Dockerfile`となっていることを確認します。

![](/images/df3c9aedfc3c13/6.png)

ターミナルから各ツールがインストールされているかバージョン確認してみましょう。

```
root@326af17361a7:/workspaces/cpp_test# g++ --version
g++ (Ubuntu 9.4.0-1ubuntu1~20.04.1) 9.4.0
Copyright (C) 2019 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

root@326af17361a7:/workspaces/cpp_test# cmake --version
cmake version 3.16.3

CMake suite maintained and supported by Kitware (kitware.com/cmake).
root@326af17361a7:/workspaces/cpp_test# git --version
git version 2.25.1

root@326af17361a7:/workspaces/cpp_test# clang-format --version
clang-format version 10.0.0-4ubuntu1
```

バージョン確認が終わりましたら、Microsoft が公式で用意してくれている VS Code での C++開発で有用な拡張機能パックをコンテナ側にインストールします。

![](/images/df3c9aedfc3c13/7.png)

## 動作確認

ここまでで一旦簡単に動作確認してみましょう。
同階層に`main.cc`を作成し、Hello, world してみます。

![](/images/df3c9aedfc3c13/8.png)

```
root@326af17361a7:/workspaces/cpp_test# g++ main.cc
root@326af17361a7:/workspaces/cpp_test# ./a.out
Hello, world
```

# まとめ

ここまででイメージ図の環境は構築できました。
次回、本環境をベースとして CMake によるプロジェクト作成、clang-format によるフォーマット設定を行います。
