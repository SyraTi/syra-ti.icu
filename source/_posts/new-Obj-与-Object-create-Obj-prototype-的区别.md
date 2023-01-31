---
title: new Obj() 与 Object.create(Obj.prototype)的区别
date: 2018-06-07 13:03:42
tags:
---
## 先看 Object.create 和new 的区别
###### 以下引用自[blueblueskyhua的博客](https://blog.csdn.net/blueblueskyhua/article/details/73135938)

>创建对象的方式，我以我碰到的两种创建方式，Object.create 和new来说明

	var Base = function () {}
	var o1 = new Base()
	var o2 = Object.create(Base)

>那这样到底有什么不一样呢？
<!-- more -->

>我先来一段Object.create的实现方式

	Object.create =  function (o) {
	    var F = function () {}
	    F.prototype = o
	    return new F()
	}
>可以看出来。Object.create是内部定义一个对象，并且让F.prototype对象 赋值为引进的对象/函数 o，并return出一个新的对象。
>再看看var o1 = new Base()的时候new做了什么。

>JavaScript 实际上执行的是：

	var o1 = new Object()
	o1.[[Prototype]] = Base.prototype
	Base.call(o1)
>new做法是新建一个obj对象o1，并且让o1的__proto__指向了Base.prototype对象。并且使用call 进行强转作用环境。从而实现了实例的创建。

### 那么new Obj() 与 Object.create(Obj.prototype)有什么区别呢

###### 以下为原创内容

	var Obj = function(){
		console.log('obj\'s constructor')
	}
	
	> new Obj()
	< obj's constructor
	< Obj {}
	> Object.create(Obj.prototype)
	< Obj {}

## —— 得出结论 Object.create(Obj.prototype)将不会触发Obj的构造函数


---
