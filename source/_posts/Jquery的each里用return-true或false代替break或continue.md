---
title: Jquery的each里用return true或false代替break或continue
date: 2017-11-30 10:10:52
tags:
---

## 在each代码块内不能使用break和continue,要实现break和continue的功能的话，要使用return true/false

```js
$(DOM).each(function(index,el){
	return false; //break 
	return true; //continue 
}
```

---