---
title: Android实战记录
date: 2018-05-13 15:36:10
tags:
  - Android
---

本文是学习 组件化封装思想实战Android App，慕课网教程的记录

<!--more-->

## 经验总结

### Application

Application 类是整个程序的入口：

1. 会做第三方组件的初始化
2. 为整个应用的其他模块提供上下文环境，因为 Application 类的 `onCreate` 只会被执行一次，所以通过在 `onCreate` 来实现单例

在创建 Application 之后，如果要使用 Application，则需要在 Manifest.xml 中加入该 Application

### 提供 BaseFragment

项目中的所有 Fragment 都没有直接继承 Fragment 类，而是继承了 BaseFragment，而 BaseFragment 继承的 Fragment。

使用 BaseFragment 的好处

1. 封装所有 Fragment 的公共行为，如在 Fragment 中进行数据统计，统计停留时间等，可以封装在 BaseFragment 中

### 日志中使用 Tag

同样，创建 BaseAcitvity，在 BaseActivity 中封装统一的行为，如生成日志的 Tag，Tag 的生成可以在 onCreate 中实现

```
public String TAG;

TAG = getComponentName().getShortClassName();
```

### 初始化 View

在开发中，最好是在 `onCreate` 的时候就初始化所有的 View，而不是在使用到的时候，才 `findViewById`。

### Bitmap和Drawable的区别

可以简单地理解为 Bitmap 储存的是 像素信息，Drawable 储存的是 对 Canvas 的一系列操作。

而 BitmapDrawable 储存的是「把 Bitmap 渲染到 Canvas 上」这个操作。

### android关于几个drawable文件夹的区别 

mdpi、hdpi、xdpi、xxdpi用来修饰Android中的drawable文件夹及values文件夹，用来区分不同像素密度下的图片和dimen值。

mdpi    120dpi~160dpi    
hdpi    160dpi~240dpi    
xhdpi    240dpi~320dpi    
xxhdpi    320dpi~480dpi    
xxxhdpi    480dpi~640dpi 

在设计图标时，对于五种主流的像素密度（MDPI、HDPI、XHDPI、XXHDPI 和 XXXHDPI）应按照 2:3:4:6:8 的比例进行缩放。例如，一个启动图标的尺寸为48x48 dp，这表示在 MDPI 的屏幕上其实际尺寸应为 48x48 px，在 HDPI 的屏幕上其实际大小是 MDPI 的 1.5 倍 (72x72 px)，在 XDPI 的屏幕上其实际大小是 MDPI 的 2 倍 (96x96 px)，依此类推。

图标尺寸

mdpi    48x48px    
hdpi    72x72px    
xhdpi    96x96px    
xxhdpi    144x144px    
xxxhdpi    192x192px

