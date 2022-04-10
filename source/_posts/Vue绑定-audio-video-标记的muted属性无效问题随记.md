---
title: Vue绑定<audio>/<video>标记的muted属性无效问题随记
date: 2021-01-28 14:59:47
tags:
---
## 问题代码
```html
<!--这里的muted绑定不起作用-->
<audio :muted="isMuted" ref="audioPlayer"></audio>
```

## 援引Github上的issue回复
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127220113689.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM0OTMwNjQw,size_16,color_FFFFFF,t_70)

翻译：muted属性类似于`<input>`的`value`属性，如果他在元素初始化时是内联的，那么属性值将作为初始值；但是如果在一开始并没有内联，只是后续更改了attribute [^1] 值，那么将不会起任何效果，正确的做法是设置相对应的property [^1] 值。


[^1]: 关于attribute和property值的区别，可以看我往期的[文章](https://blog.csdn.net/sinat_34930640/article/details/76310396)。

### 解决方式：使用watch来手动设置muted
*html*
```html
<audio ref="audioPlayer"></audio>
```
*javascript*
```javascript
watch:{
	isMuted(val){
		if(val){
			this.$refs.audioPlayer.muted = 'muted'
		}
		else{
			this.$refs.audioPlayer.muted = ''
		}
	}
}
```
