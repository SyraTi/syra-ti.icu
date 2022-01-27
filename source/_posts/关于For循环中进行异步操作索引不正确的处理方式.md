---
title: 关于For循环中进行异步操作索引不正确的处理方式
date: 2017-11-28 10:14:18
tags:
---
### code 问题代码
```js
function test() {
    for (var i = 0; i < 5; ++i) {
        setTimeout(function() {
            console.log("index is :", i);
        }, 1000);
    }
}
test();
```
### output 问题输出
```js
index is : 5
index is : 5
index is : 5
index is : 5
index is : 5
```

### solution 使用自执行函数
```js
function test() {
    for (var i = 0; i < 5; ++i) {
        (function(i){
	        setTimeout(function() {
	            console.log("index is :", i);
	        }, 1000)
        })(i);
    }
}
test();
```

### output 正确输出
```js
index is : 0
index is : 1
index is : 2
index is : 3
index is : 4
```


---