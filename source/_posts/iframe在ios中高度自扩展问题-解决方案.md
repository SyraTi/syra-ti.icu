---
title: iframe在ios中高度自扩展问题 解决方案
date: 2018-10-11 14:52:31
tags:
---

##### 开发需求
在页面中加入一个定高的iframe
##### 问题
IOS safari中iframe会根据内容高度进行iframe的内容自扩展以适应内容高度 设置height无效

###### 百谷之后
>在 iOS Safari 上，对于一个 scrollable 的 iframe 元素，iOS Safari 会选择扩展 iframe 的高度来自适应其中 web 页面内容的高度。所以当页面内容超过 iframe 设置的高度时，iOS Safari 并不会像在 Android 浏览器中那样维持 iframe 的高度并在右侧显示一个拖动条，而是直接扩展 iframe 的高度。

<!-- more -->
##### 解决方案
① 在iframe上添加 scrollable=no
此方案会使safari中iframe的设置高度生效不再自扩展，但是页面会直接被裁剪 产生类似于overflow:hidden的效果 不是想要的效果
②在iframe外增加一个定高div 并overflow-y:hidden，至此IOS满足需求，发现安卓显示高度不正确，令iframe高度100% 至此双端表现一致 满足需求 采用该方案

##### 代码
###### 问题代码
```html
<iframe src="url" style="height:300px"></iframe>
```
###### 方案①代码
```html
<iframe src="url" scrollable="no" style="height:300px"></iframe>
```
###### 方案②代码（最终代码）
```html
<div class="wrap" style="height:300px;overflow-y:hidden">
	<iframe src="url" style="height:100%"></iframe>
</div>
```

### --- 以上 问题解决
### 有什么不明白的可以在评论里留言(￣▽￣)／

---
