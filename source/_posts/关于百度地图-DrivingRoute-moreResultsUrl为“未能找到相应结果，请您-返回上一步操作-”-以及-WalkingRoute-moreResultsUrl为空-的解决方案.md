---
title: >-
  关于百度地图 DrivingRoute moreResultsUrl为“未能找到相应结果，请您:返回上一步操作..” 以及 WalkingRoute
  moreResultsUrl为空 的解决方案
date: 2017-12-07 16:27:37
tags:
---
#### —— 因为百度地图极速版貌似正在关闭并且停止维护了 文档页面和介绍页面都跳转到了正式版的API，然而极速版中非常好用的 routeSearch.routeCall 没了 于是乎开始找代替方案

### **A Few Minutes Later.....**
经过一番Demo和文档的查找，惊喜的发现正式版中TransitRoute，WalkingRoute，以及DrivingRoute 的searchComplete回调中有个“moreResultsUrl” 打开正是百度地图的导航结果页！
&nbsp;
### **事情并没有那么简单  —— 使用DrivingRoute**
当我打开DrivingRoute的moreResultsUrl 度娘很友好的向我展示了下面的界面 并且不管我如何调整参数 emmmmm 毫无变化！

![DrivingRoute未能找到相应结果](https://img-blog.csdnimg.cn/img_convert/d953d2f1c93ba21043d2bb65adfcd32b.png)

&nbsp;
&nbsp;

### **Fine 不开车我走路行了吧  —— 使用WalkingRoute!**

![ops!](https://img-blog.csdnimg.cn/img_convert/0576d2b2c8e66659d39bb9c500dabe9c.png)

![这里写图片描述](https://img-blog.csdnimg.cn/img_convert/0385b0475d2f8f0784130c8f3d045013.png)
..
..................
..........................
........................................ ![这里写图片描述](https://img-blog.csdnimg.cn/img_convert/f052cca3f5e3eee8bb5117452a8b5264.png)

*以上自行感受（崩溃）
&nbsp;
&nbsp;
### **...好吧下下策  坐公交吧\_(:з」∠)_ —— 使用TransitRoute**
那个,出来是出来了.......

![这里写图片描述](https://img-blog.csdnimg.cn/img_convert/5b554afc825dfce4c6cf9f2cc824b8f7.png)

为什么地图还要我切换选项卡才能看到啊！！！！（拿烟的手微微颤抖.jpg

### **OK 冷静 我们来分析一下公交的链接...**

>http://api.map.baidu.com/direction?region=杭州市&origin=30.26348989892,120.21415684874&destination=30.235552,120.205199&output=html&mode=transit&src=baidu_map_jsapi

↓
↓仔细看↓
↓
>?region=city&origin=30.26348989892,120.21415684874&destination=30.235552,120.205199&output=html&mode=transit&src=baidu_map_jsapi

↓
↓仔细看↓
↓
>&mode=transit&src=baidu_map_jsapi

↓
↓仔细看↓
↓
>&mode=transit

是不是明白了什么（滑稽）？
### **Magic！——使用Driving模式和Walking模式的变通方式**
```js
moreResultsUrl.replace('transit', 'driving');
```
```js
moreResultsUrl.replace('transit', 'walking');
```
PS.希望百度早点修复吧、、、 有什么问题在下面问或者说有更好的办法请告诉我 谢谢~☆


#### 2018/5/3 更新

现在似乎三种都无法获取到moreResultsUrl了替代方案为 直接传入region、origin、destination、mode 拼接下面的url然后跳转


```js
window.location.href = 'http://api.map.baidu.com/direction'+
							'?region='+region+
							'&origin='+origin.lat+','+origin.lng+
							'&destination='+dest.lat+','+dest.lng+
							'&output=html'+
							'&mode='+transit/walking/driving+
							'&src=baidu_map_jsapi'

```


---