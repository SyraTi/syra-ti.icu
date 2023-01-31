---
title: 关于CSS选择器书写正确浏览器却不认的一个原因
date: 2020-03-26 11:05:37
tags:
---


>不想看原因的，可以直接跳到页尾看解决方案

**今天在写样式的时候出现不生效的问题，选择器是正确的，使用jquery也能正常选择到，但是浏览器就是不识别这一段样式。**


*问题样式 ↓*
![问题样式](https://img-blog.csdnimg.cn/20200326102440376.png)
*jQuery能选择到 ↓*
![jQuery能选择到](https://img-blog.csdnimg.cn/20200326102815730.png)

*然而Chrome的开发者工具里面根本没有出现这一行 ↓*
![Chrome不识别](https://img-blog.csdnimg.cn/20200326102544543.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM0OTMwNjQw,size_16,color_FFFFFF,t_70)

------------------
<!-- more -->
**排查到最后尝试删除了选择器末尾的空格，发现居然神奇的被识别了。。。。**



*删除末尾空格 ↓*
![删除空格](https://img-blog.csdnimg.cn/20200326103031581.png)
*识别成功 ↓*
![识别成功](https://img-blog.csdnimg.cn/20200326103217265.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM0OTMwNjQw,size_16,color_FFFFFF,t_70)

----------------
问题是解决了，但是奇了怪了，以前好像从来没有被末尾空格影响过啊，同文件里其他末尾空格也没影响到样式识别啊？

直觉告诉我是不是BOM头类似的问题，是其他的不可见字符导致的？

于是乎原样复制了问题样式末尾的的空格，转成unicode查看，果然。。。。

![空格转Unicode](https://img-blog.csdnimg.cn/20200326105101412.png)
可以看到 `两个空格+一个大括号` 一共三个字符，unicode分别为 160、32、123，果然两个空格不是一伙的！

查阅了Unicode表，发现
- Unicode 32，是普通空格 —— 就是在末尾会被忽略的那种；
- Unicode 160，是No-break Space —— 也就是我们熟知的&amp;nbsp; 这个放在选择器末尾是会被浏览器识别的！

这就导致了`.mini-buttonedit-border`变成了`.mini-buttonedit-border&nbsp;` 当然就没法对应到元素上了！

---
事出有因，这段css是在Chrome的开发者工具调整好选中直接复制出来的，相信很多人这么干过↓
![在开发者工具选中复制](https://img-blog.csdnimg.cn/20200326105853418.png)
有可能复制的时候会带有&amp;nbsp;字符，现在要注意了。。。

---


## 所以 解决方案就是：→删掉选择器末尾空格←
##### PS：不一定是花括号前面那个，理论上每个空格都有可能出现这个问题，如果删去花括号前面的还有问题，可以尝试把前面的空格都重新打一遍。

<br><br><br>


以上，如果有什么问题欢迎在下面评论，我会及时回复www

----
