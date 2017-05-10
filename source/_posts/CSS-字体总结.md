---
title: CSS 字体总结
date: 2017-05-09 20:28:39
categories: coding
tags:
  - CSS
  - Font
---

本文总结了CSS字体中常见的概念，字体的注意事项。

<!--more-->

## 字体的分类

就 Web 常用的一些字体而言，经常听说的字体类型，大致可以分为这几种：

* serif(衬线)
* sans-serif(无衬线)
* monospace(等宽)
* fantasy(梦幻)
* cursive(草体)

其实大体上分为衬线字体和无衬线字体，等宽字体中也有衬线等宽和无衬线等宽字体，这 5 个分类是 `font-family` 的 5 个可用字体系列取值。

下面是这五种字体中、英文的显示效果，我们可以看到，其实有的字体中文显示效果相同，有的字体英文显示效果相同，下面我们来分别看一下这几种字体的区别。

![](http://ojt6zsxg2.bkt.clouddn.com/eb66c44ff11117e0b08896b414c8ade5.png)

### serif-衬线字体

特点：

* 字符笔画末端有叫做衬线的小细节的额外装饰
* 笔画的粗细会有所不同

![](http://ojt6zsxg2.bkt.clouddn.com/bd102a06040a1a53e39a8009dc22e059.png)

属于衬线字体的字体有：

#### 宋体（SimSun）

Windows 下大部分浏览器的默认中文字体，是为适应印刷术而出现的一种汉字字体。笔画有粗细变化，是一种衬线字体，宋体在小字号下的显示效果还可以接受，但是字号一大体验就很差了，所以使用的时候要注意，不建议做标题字体使用。

#### Times New Roman

Mac 平台 Safari 下默认的英文字体，是最常见且广为人知的西文衬线字体之一，众多网页浏览器和文字处理软件都是用它作为默认字体。

### sans-serif-无衬线字体

特点：

* 没有衬线
* 字体线条的宽度相同

![](http://ojt6zsxg2.bkt.clouddn.com/fb47aafd02223d0ee65f69de0aee7a72.png)

中文下，无衬线字体就是**黑体**，黑体字也就是又称方体或等线体，没有衬线装饰，字形端庄，笔画横平竖直，笔迹全部一样粗细。

#### 微软雅黑（Microsoft Yahei）

大名鼎鼎的微软雅黑相信都不陌生，从 windows Vista 开始，微软提供了这款新的字体，一款无衬线的黑体类字体，显著提高了字体的显示效果。现在这款字体已经成为 windows 浏览器最值得使用的中文字体。

#### 华文黑体（STHeiti）、华文细黑（STXihei）

属于同一字体家族系列，MAC OS X 10.6 之前的简体中文系统界面的默认中文字体，正常粗细就是华文细黑，粗体下则是华文黑体。

#### 黑体-简（Heiti SC）

从 MAC OS X 10.6 开始，黑体-简代替华文黑体用作简体中文系统界面默认字体，苹果生态最常用的字体之一，包括 iPhone、iPad 等设备用的也是这款字体。

#### 冬青黑体（Hiragino Sans GB）

又叫苹果丽黑，Hiragino 是字游工房设计的系列字体名称。是一款清新的专业印刷字体，小字号时足够清晰，Mac OS X 10.6 开始自带有 W3 和 W6 。

#### Helvetica、Helvetica Neue

被广泛用于全世界使用拉丁字母和西里尔字母的国家。Helvetica 是苹果电脑的默认字体，微软常用的 Arial 字体也来自于它。

#### Arial

Windows 平台上默认的无衬线西文字体，有多种变体，比例及字重（weight）和 Helvetica 极为相近。

#### Tahoma

十分常见的无衬线字体，字体结构和 Verdana 很相似，其字元间距较小，而且对 Unicode 字集的支持范围较大。许多不喜欢 Arial 字体的人常常会改用 Tahoma 来代替，除了是因为 Tahoma 很容易取得之外，也是因为 Tahoma 没有一些 Arial 为人诟病的缺点，例如大写“i”与小写“L”难以分辨等。。

下面是各种字体的显示效果：

![](http://ojt6zsxg2.bkt.clouddn.com/9db133a46a80f502e971311b54ceed37.png)

### monospace-等宽字体

特点：

* 字符宽度相同，通常用于计算机相关书籍中排版代码块

#### Consolas

这是一套等宽的字体，属无衬线字体。这个字体使用了微软的 ClearType 字型平滑技术，主要是设计做为代码的显示字型之用，特别之处是它的“0”字加入了一斜撇，以方便与字母“O”分辨。

```
ClearType：由微软在其操作系统中提供的屏幕亚像素微调字体平滑工具，让 Windows 字体更加漂亮。在 Windows XP 平台上，这项技术默认是关闭，到了Windows Vista 才默认为开启。
```

#### Courier

Courier是一种常见的计算机字体，由IBM公司的Bud Ketler设计，应用于IBM和其它打字机。

以上两种字体的显示效果如下：

![](http://ojt6zsxg2.bkt.clouddn.com/43865a491ee9892f73bbf2d37ad6747e.png)

### fantasy 、cuisive
fantasy和 cuisive 字体在浏览器中不常用，在各个浏览器中有明显的差异。


## 字体的使用

### 字体的引号

在 [CSS Validation Service (CSS 校验器)](http://jigsaw.w3.org/css-validator/)，中，提供了一种对网站 CSS 使用格式进行校验的工具，大家不妨将自己的链接贴到校验器中，查看自己的 CSS 语法是否有错误。

我们在使用字体的时候，有时会遇到一些困惑的现象，就是在 `font-family` 中，有的字体需要添加引号，有的字体中的符号需要进行转义，为什么会出现这些情况，我们又应该怎么处理呢？

首先放出结论，**需要判断空格包裹字体的情况大致有以下几种：**

1. 如果字体族的名字中，空格分隔的非关键字部分都是合格的标识符，则该整体可以不使用引号进行包裹。如果在字体族的名字中存在非法的 CSS 标识符，则该字体族的名字需要用字符串的形式进行替代，或者将非法的标识符进行转义
2. 如果字体族的名字和关键字一样，则需要加引号

下面我们针对以上一点进行分析：

**CSS 中的标识符**：在CSS里，标识符(包括元素名称，类名，选择器里的ID)只能包含字符`[a-zA-Z0-9]`，ISO 10646里比`U+00A0`大的字符，还有连字符（`-`）和下划线（`_`）

#### 情况一

```
字体族的名字要么作为字符串用引号包含起来，要么作为标识符，不需要引号。
```

上面一句话表示，如果是没有引号包裹的字体族名称，则各个空格分隔的部分都必须满足标识符的定义。如果其中有不满足标识符的部分，则该字体族需要用引号包裹。

```
如果font family的名字是一系列标识符。那么计算机识别的最终值是单个空格分隔的标识符转换成字符串后的值。标识符前后的空格会被忽略，内部的其他多个空格会被转成一个空格。
```

这一句话也表示出，如果一系列标识符都是合法的，则该字体族的含义和用字符串包裹的含义一致，即 `font-family: Comic Sans MS`是合法的，和`font-family: 'Comic Sans MS'`等价

#### 情况二

规范定义了如下的字体族关键字：`serif`, `sans-serif`, `cursive`, `fantasy`, 和 `monospace`。这些关键字可以作为普通的回退机制，以防期望的字体不可以用的时候。作者被鼓励在字体定义的最后面加上一种上面这些字体之一作为最后的可选项，来提高代码的鲁棒性。作为关键字，他们不需要引号。

如果自己设计的字体族的名字和已有的关键字相同，则该字体族的名字需要加上引号。

换句话说，`font-family: sans-serif`代表的是一个普通的 sans-serif 字体族。而`font-family: 'sans-serif'`则代表了一个名字叫 sans-serif 的字体。这个区别很重要。

### 中文字体的写法

一些中文字体，例如font-family: '宋体'，由于字符编码的问题，少部分浏览器解释这个代码的时候，中文出现乱码，这个时候设定的字体无法正常显示。

所以通常会转化成对应的英文写法或者是对应的 unicode 编码，`font-family:'宋体' -> font-family: '\5b8b\4f53'`。

`\5b8b\4f53` 是宋体两个中文字的 unicode 编码表示。类似的写法还有：

* 黑体：`\9ED1\4F53`
* 微软雅黑：`\5FAE\8F6F\96C5\9ED1`
* 华文细黑：`\534E\6587\7EC6\9ED1`
* 华文黑体：`\534E\6587\9ED1\4F53`

### 字体定义的顺序

字体定义顺序是一门学问，通常而言，我们定义字体的时候，会定义多个字体或字体系列：

```
body {
    font: 12px/1.5 tahoma,arial,'Hiragino Sans GB','\5b8b\4f53',sans-serif;
}
```

别看短短 5 个字体名，其实其中门道很深。解释一下：

1. 使用 `tahoma` 作为首选的西文字体，小字号下结构清晰端整、阅读辨识容易；
2. 用户电脑未预装 `tohoma`，则选择 `arial` 作为替代的西文字体，覆盖 windows 和 MAC OS；
3. `Hiragino Sans GB` 为冬青黑体，首选的中文字体，保证了 MAC 用户的观看体验；
4. Windows 下没有预装冬青黑体，则使用 `'\5b8b\4f53'` 宋体为替代的中文字体方案，小字号下有着不错的效果；
5. 最后使用无衬线系列字体 `sans-serif` 结尾，保证旧版本操作系统用户能选中一款电脑预装的无衬线字体，向下兼容。

### 书写规则

字体 font-family 定义的原则大概遵循：

1、兼顾中西

中文或者西文（英文）都要考虑到。

2、西文在前，中文在后

由于大部分中文字体也是带有英文部分的，但是英文部分又不怎么好看，同理英文字体中大多不包含中文。

所以通常会先进行英文字体的声明，选择最优的英文字体，这样不会影响到中文字体的选择，中文字体声明则紧随其次。

3、兼顾多操作系统

选择字体的时候要考虑多操作系统。例如 MAC OS 下的很多中文字体在 Windows 都没有预装，为了保证 MAC 用户的体验，在定义中文字体的时候，先定义 MAC 用户的中文字体，再定义 Windows 用户的中文字体。其次很多人都不知道 Android 下没有预装微软雅黑和宋体，那么 Android 机型下的中文字体设置又是很考究的。

4、兼顾旧操作系统，以字体族系列 `serif` 和 `sans-serif` 结尾

当使用一些非常新的字体时，要考虑向下兼容，兼顾到一些极旧的操作系统，使用字体族系列 `serif` 和 `sans-serif` 结尾总归是不错的选择。

### 现有的一些网站字体属性

**淘宝首页**：

```
body {
    font: 12px/1.5 tahoma,arial,'Hiragino Sans GB','\5b8b\4f53',sans-serif;
}
```

**知乎首页**：

```
body {
    font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif;
}
```

**知乎文章页面**：

```
body {
    font-family: -apple-system, "Helvetica Neue", "Arial", "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", "WenQuanYi Micro Hei", sans-serif;
    font0-weight: 400;
}
```

**GitHub README 页面**：

```
body {
    font-family : -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
    
}
```

**GitHub README 页面 `<code>` 标签**：

```
code {
    font-family : "SFMono-Regular", Consolas, "Liberation Mono", Menlo, Courier, monospace;
}
```

## 参考

本文参考的文章有：

* [谈谈一些有趣的CSS题目（十二）-- 你该知道的字体 font-family](http://www.cnblogs.com/coco1s/p/6253154.html)
* [CSS的font family什么时候需要引号](http://www.lixuejiang.me/2017/01/02/css%E7%9A%84font-family%E4%BB%80%E4%B9%88%E6%97%B6%E5%80%99%E9%9C%80%E8%A6%81%E5%BC%95%E5%8F%B7/)


