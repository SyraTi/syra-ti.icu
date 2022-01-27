---
title: 'ios中 input[readonly] 仍然可聚焦出现光标并弹出“完成”面板问题'
date: 2018-02-05 17:47:24
tags:
---
除readonly外 还需要添加

	unselectable="on" onfocus="this.blur()" 

---