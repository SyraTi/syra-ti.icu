---
title: 关于mui Div模式选项卡 JS主动触发切换指定选项卡
date: 2017-11-07 23:20:22
tags:
---
### 需要先触发touchstart事件（旧版本需要先触发mousedown事件）
```javascript
	mui.trigger($('.mui-tab-item').eq(1)[0],'touchstart');
	mui.trigger($('.mui-tab-item').eq(1)[0],'tap')
```
### 源代码尚未研究 以上为解决方案

#### 2018-3-30 PS: 发现好像只要改一下mui-active就行了？（扶额）
#### 2018-4-25 PS: 最简单的方式是改变item和切换目标的mui-active


---