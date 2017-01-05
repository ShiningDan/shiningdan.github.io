---
title: CSS-注意事项
date: 2017-01-05 20:26:33
categories:
tags:
  - CSS
---

## float 和 display:inline-block 的使用

**float 和 display:inline-block 不能同时使用，在有的浏览器(chrome)中，浏览器对 css 代码解释的过程中，有时候按照 float 来布局，有时候按照 inline-block 来布局，会使得布局产生变化。**

设了float:left的元素允许它的右边存在任何元素同行显示，不论是内联元素还是块元素。但它的左边还是不允许存在任何元素与之同行显示，哪怕其它的元素的代码在前，除非也给前面的元素加上float:left后，才允许同行显示。

设了display:inline的元素，允许它的前后存在其它的内联元素同行显示。关于代码在其前面的块元素之同行显示，则要让前面的元素浮动（不管是左还是右浮动）或设为display:inline，还有代码在后面的是块元素（管它有没有浮动，是左浮动还是右浮动），均不能与之同行，除非设为display:inline。

## 使用 overflow:hidden 清除浮动

出现情况：**父块没有设置指定的高宽，当子块设置为浮动后，原本包裹子块的父块的高度塌陷消失，这时给父块设置overflow:hidden就能清楚浮动造成的影响，使父块重新包裹子块。**

<!--more-->

[知乎 overflow:hidden 能够清除浮动的解释](https://www.zhihu.com/question/30938856)

相关知识：[BFC 的原理及其应用](http://www.cnblogs.com/lhb25/p/inside-block-formatting-ontext.html)

## 阴影

可以对块添加 box-shadow 属性向框添加一个或多个阴影。

## 全字母大写，放大，加下划线

如果想实现全字母大写，放大，加下划线，如图下效果：

![](http://7xs2ra.com1.z0.glb.clouddn.com/22.PNG)

直观想到的是全字母大写，然后首字母的 font-size 进行放大来进行是实现。

	p {
		font-variant:small-caps;
		text-decoration: underline;
	}

	p:first-letter {
	    font-size: 30px;
	}

但是这种实现效果如下图，首字母放大会影响到下划线的显示效果：

![](http://7xs2ra.com1.z0.glb.clouddn.com/23.PNG)

-------------

所以我们的实现方法，是使用 font-variant:small-caps; 来代替 :first-letter 设置首字母大小，这样的显示效果就不会影响下划线。

	p {
		font-variant:small-caps;
		text-decoration: underline;
		text-transform: capitalize;
	}

显示效果如下：

![](http://7xs2ra.com1.z0.glb.clouddn.com/22.PNG)


















