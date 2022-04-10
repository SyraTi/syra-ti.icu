---
title: 拒绝假空心！纯css实现 真·空心带边框直角三角形和空心带边框直角梯形！
date: 2019-06-07 23:17:06
tags:
---
## 需求
![要实现的直角梯形](https://img-blog.csdnimg.cn/20190607225358253.png)
**↑需要实现一个背景为半透明且带边框的直角梯形**
<br>


## 百谷解决方案
碰到问题，程序猿的优良传统，先百谷一下（？） 得到的解决方案总结一下有这些：
1.利用border交界处为斜边来制作，即border-top:none;border-right:transparent
**→问题：使用border来做背景，没法在border上再做边框，只能得到一个半透明的直角梯形**
2.在用border制作的梯形外侧套上一个上下左右大出1像素，以边框色为背景的梯形，作为边框
**→问题：中间小梯形无法做成半透明，会叠加上大梯形的背景色**
3.利用矩形和三角形拼接
**→问题：右边直角三角形的边框依旧无法解决**
4.边框使用矩形旋转来做
**→问题：至少要用到4个div，过于复杂**
<br>

## 没有条件就自己上！
就在要放弃打算导出png来做的时候，导出界面给了我灵感！
![导出图片界面](https://img-blog.csdnimg.cn/20190607230522372.png)
**↑这不就是一个矩形overflow:hidden套上一个平行四边形吗！给力！**

不废话立马做了出来
#### html
```html
<!-- 真·空心直角三角形 -->
<div class="tri"></div>
<!-- 真·空心直角梯形 -->
<div class="trapezoid"></div>
```

#### css
```css
/* 直角三角形 */
.tri{
  position:relative;
  height: 80px;
  width: 80px;
  border-left:5px solid red;
  overflow:hidden;
}
.tri:before{
  content:'';
  display:block;
  position:absolute;
  top:0;
  bottom:0;
  right:0;
  width: 80px;
  border:5px solid blue;
  transform-origin:0 100%;
  transform:skewX(45deg);
}
/* 直角梯形 */
.trapezoid{
  margin-top: 10px;
  position:relative;
  height: 80px;
  width: 120px;
  border-left:5px solid red;
  overflow:hidden;
}
.trapezoid:before{
  content:'';
  display:block;
  position:absolute;
  top:0;
  bottom:0;
  right:0;
  width: 120px;
  border:5px solid blue;
  transform-origin:0 100%;
  transform:skewX(45deg);
}
```
<br>

## 效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190607231305645.png)
方便你们看，红色的是矩形的边框，蓝色的是平行四边形的边框！
是空心的哦！真是太满意了！

不多说了，上DEMO！
[Demo点这里](https://codepen.io/KissekiAkaru/pen/argrVe)

有什么问题欢迎在下面评论，我会及时回复www

----