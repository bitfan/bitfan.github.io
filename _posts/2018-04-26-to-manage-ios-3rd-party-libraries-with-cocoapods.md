---
layout: post
comment: true
title: 使用CocoaPods管理iOS项目第三方库
key: 2018042601
tags:
  - iOS
  - CocoaPods
lang: zh
---
CocoaPods是iOS项目最常用的类库管理工具，绝大部分有名的开源类库，都支持CocoaPods。

## 安装CocoaPods
在安装CocoaPods之前，首先要在本地安装好Ruby环境。至于如何在Mac中安装好Ruby环境，请google一下，本文不再涉及。

假如你在本地已经安装好Ruby环境，那么下载和安装CocoaPods将十分简单，只需要一行命令：
```
 sudo gem install cocoapods
```

## 使用CocoaPods
在iOS项目目录下创建文件`Podfile`[^Podfile]
```
platform :ios, '11.0'
target 'XXXXX' do
        pod 'AFNetworking', '~> 3.2.0'
        pod 'OpenCV2', '~> 3.4.1'
end
```
然后安装第三方库：
```
pod install
```
如果不清楚库名称或者版本号，可以使用`pod search`查询相关库：
```
pod search OpenCV
```
在修改`Podfile`之后，可以使用`pod update`命令进行更新。

[^Podfile]: [The Podfile](https://guides.cocoapods.org/using/the-podfile.html)
