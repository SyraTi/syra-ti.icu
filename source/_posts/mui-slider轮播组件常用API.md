---
title: mui slider轮播组件常用API
date: 2017-08-09 11:41:12
tags:
---
原文:[http://www.bcty365.com/content-146-5588-1.html](http://www.bcty365.com/content-146-5588-1.html)

## 组件介绍
轮播组件是mui提供的一个核心组件，在该核心组件基础上，衍生出了图片轮播、可拖动式图文表格、可拖动式选项卡、左右滑动9宫格等组件。

## 示例

获取slide组件对象

	var slider = mui('#slider').slider();
## 常用API

### gotoItem(index)
切换到指定轮播

	slider.gotoItem(1); //切换至第二个轮播
### prevItem()
上一个轮播

	slider.prevItem();
### nextItem()
下一个轮播

	slider.nextItem();
### getSlideNumber()
当前所处位置

	slider.getSlideNumber();
### stopped
禁用/开启滑动切换

	slider.stopped = false; //开启滑动切换
	slider.stopped = true; //关闭滑动切换

---