---
title: 初探 React Native
date: 2019-04-14 14:19:09
categories: coding
tags:
 - React
 - React Native
---

## 初始化

### React Native CLI 和 Expo CLI

这里我没有去看两种不同方式生成的项目，都需要哪些工具支持，只是做了一些对比，结论是 

* Expo Cli 更加适合前端开发者，RN Cli 更加适合客户端开发者。
* RN Cli 可以在客户端集成，Expo 生成的项目，难以导入客户端代码（使用 `eject` 方式）

具体可以查看 [react-native | Getting Started](https://facebook.github.io/react-native/docs/getting-started)

<!--more-->

#### React Native init

好处：

* **您可以添加用Java / Objective-C编写的本机模块（可能是唯一但最强大的模块）**

缺点：

* 需要Android Studio和XCode来运行项目
* 没有mac就无法为iOS开发
* 设备必须通过USB连接才能用于测试

#### Expo

好处：

* 共享应用程序很简单（通过QR码或链接），您不必发送整个.apk或.ipa文件
* 设置项目很简单，只需几分钟即可完成
* Expo可以构建.apk和.ipa文件（可以通过Expo分发到商店）

缺点：

* **你不能添加原生模块（可能是一些游戏改造者）**
* 您不能使用在Objective-C / Java中使用本机代码的库
* 标准的Hello World应用程序大约25MB（因为集成了库

### 


## 参考

* [react-native 官方文档](https://facebook.github.io/react-native/docs/getting-started)
