---
title: window.location.replace和window.location.href区别
date: 2017-12-07 15:09:52
tags:
---
###有3个页面 a,b,c
如果当前页面是c页面，并且c页面是这样跳转过来的：a->b->c

1: b->c 是通过window.location.replace("..xx/c")   此时b页面的url会被c页面代替，并且点击后退按钮时会回退到a页面（最开始的页面）

2: b->c是通过window.location.href("..xx/c")    此时b页面的路径会被c页面代替，但是点击回按钮后页面回退的是b页面

两者的区别: 两者后退时所回退的页面不一样