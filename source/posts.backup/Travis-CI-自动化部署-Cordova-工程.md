---
title: Travis CI 自动化部署 Cordova 工程
date: 2018-10-10 22:50:34
categories: 技术相关
tags: [travis-ci, cordava, android]
reward: true
---
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/others/26.png" width="45%" height="45%">

前些时间心血来潮，开发了一款针对武汉大学图书馆预约系统的桌面端抢座软件(使用 Electron + Vue 搭建)，发布之后，有很多同学向我反映说想要 Android 版本的。

虽然利用 Cordova/PhoneGap 将网页打包为 Android 安装包并不是一件很复杂的事，但在笔记本上安装 Android 的开发环境实在让人崩溃。

但出于对用户需求的尊重，我还是开始了将网页打包为 Apk 的挖坑之旅，项目链接：[https://github.com/CS-Tao/whu-library-seat-mobile](https://github.com/CS-Tao/whu-library-seat-mobile)。

本文用于记录在本项目中利用 Travis CI 持续集成和部署的配置代码。
<!-- more -->

### 以 node.js 为主环境的配置方法

本方法的构建和部署耗时 580s 左右，构建日志：[https://travis-ci.com/CS-Tao/whu-library-seat-mobile/builds/88561677](https://travis-ci.com/CS-Tao/whu-library-seat-mobile/builds/88561677)

```yml
sudo: required

language: node_js
node_js: stable

cache:
  directories:
    - node_modules
    - cordova/node_modules

addons:
  apt:
    sources:
      - ubuntu-toolchain-r-test
    packages:
      - g++-4.8
      - openjdk-7-jdk
      - lib32stdc++6
      - lib32z1
env:
  CXX=g++-4.8

install:
  - yarn
  - yarn --cwd cordova

before_script:
  # 添加 android 环境
  - mkdir android-sdk
  - wget -P android-sdk -q https://dl.google.com/android/android-sdk_r24.4.1-linux.tgz
  - tar -xf ./android-sdk/android-sdk_r24.4.1-linux.tgz -C ./android-sdk
  - echo y | ./android-sdk/android-sdk-linux/tools/android update sdk --no-ui --all --filter platform-tools
  - echo y | ./android-sdk/android-sdk-linux/tools/android update sdk --no-ui --all --filter build-tools-26.0.2
  - echo y | ./android-sdk/android-sdk-linux/tools/android update sdk --no-ui --all --filter android-27
  - echo y | ./android-sdk/android-sdk-linux/tools/android update sdk --no-ui --all --filter extra-android-support
  - echo y | ./android-sdk/android-sdk-linux/tools/android update sdk --no-ui --all --filter extra-android-m2repository
  - echo y | ./android-sdk/android-sdk-linux/tools/android update sdk --no-ui --all --filter extra-google-m2repository
  - export ANDROID_HOME=$PWD/android-sdk/android-sdk-linux
  - export PATH=${PATH}:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools:$ANDROID_HOME/build-tools/26.0.2

script:
  - yarn build
  - cd cordova/
  - npm install -g cordova
  - cordova platform add android
  - cordova prepare android
  - cordova build android --release -- --keystore="release-key.keystore" --alias=whu-library-seat-mobile --storePassword=${STORE_PASSWORD} --password=${PASSWORD}
  - mv ./platforms/android/app/build/outputs/apk/release/app-release.apk whu-library-seat-mobile_${TRAVIS_TAG}.apk

deploy:
  provider: releases
  skip-cleanup: true
  overwrite: true
  api_key: $GH_TOKEN
  file: 
    - "whu-library-seat-mobile_${TRAVIS_TAG}.apk"
  on:
    tags: true
```
源文件链接：[.travis.yml](https://github.com/CS-Tao/whu-library-seat-mobile/blob/3c67e5889c7fd1d3a05db23928cbfb0f17a20fc6/.travis.yml)

### 以 android 为主环境的配置方法

本方法的构建和部署耗时 450s 左右，构建日志：[https://travis-ci.com/CS-Tao/whu-library-seat-mobile/builds/89426551](https://travis-ci.com/CS-Tao/whu-library-seat-mobile/builds/89426551)

```yml
sudo: required

language: android

android:
  components:
    - tools
    - build-tools-26.0.2
    - android-27
    - extra-android-m2repository
    - extra-android-support
    - extra-google-m2repository
  licenses:
    - 'android-sdk-license.*'

cache:
  directories:
    - node_modules
    - cordova/node_modules

before_install:
  # 添加 node.js 环境
  - curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
  - sudo apt-get install nodejs
  - curl -o- -L https://yarnpkg.com/install.sh | bash
  - source ~/.bashrc
  - node -v && npm -v && yarn -v

install:
  - yarn
  - yarn --cwd cordova

script:
  - yarn build
  - cd cordova/
  - sudo npm install -g cordova
  - cordova -v
  - cordova platform add android
  - cordova prepare android
  - cordova build android --release -- --keystore="release-key.keystore" --alias=whu-library-seat-mobile --storePassword=${STORE_PASSWORD} --password=${PASSWORD}
  - mv ./platforms/android/app/build/outputs/apk/release/app-release.apk whu-library-seat-mobile_${TRAVIS_TAG}.apk
  # - zipalign -v 4 ./platforms/android/app/build/outputs/apk/release/app-release.apk whu-library-seat-mobile_${TRAVIS_TAG}.apk

deploy:
  provider: releases
  skip-cleanup: true
  overwrite: true
  api_key: $GH_TOKEN
  file: 
    - "whu-library-seat-mobile_${TRAVIS_TAG}.apk"
  on:
    tags: true
```
源文件链接：[.travis.yml](https://github.com/CS-Tao/whu-library-seat-mobile/blob/c55bf899c486b527ca621a21ed150b14019eb6f5/.travis.yml)

### 说明

> 请在 Travis CI 的网站中设置`$GH_TOKEN`环境变量并设置为不可见(默认便是不可见)，该环境变量是 GitHub 的`Personal access token`。

> 代码中的`yarn build`命令用于将生产版本的网页放入 cordova 项目的`www`文件夹中。代码中的`${STORE_PASSWORD}`和`${PASSWORD}`环境变量为下面这条命令需要输入的两个密码，也应该添加到 Travis CI 的网站中，并设置为不可见。

```bash
# 在 cordova 项目的根目录中生成 release-key.keystore 文件
keytool -genkey -v -keystore release-key.keystore -alias whu-library-seat-mobile -keyalg RSA -keysize 2048 -validity 10000
```
