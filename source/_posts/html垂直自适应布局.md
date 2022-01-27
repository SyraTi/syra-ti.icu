---
title: html垂直自适应布局
date: 2017-12-02 10:11:28
tags:
---
### 项目中遇到如下需求
```html
<body>
	<!-- top 盒子内容为内部撑开 -->
	<div class="top">
		<!-- 未知高度的一些元素 -->
	</div>
	<!-- 目标盒子 需要自动填满屏幕剩余高度-->
	<div class="target"></div>
</body>
```
### 解决方案
```css
body,html{
	height:100%;
	width:100%;
}
body{
	display:flex;
	flex-direction: column
}
.target{
	flex:1;
}
```


---