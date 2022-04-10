---
title: '微信浏览器中iframe srcdoc、src=data:text/html,xxxx 等无法使用情况下防止样式污染的解决方案'
date: 2018-11-07 16:07:46
tags:
---
#### 业务需求
> 从后端取HTML 前端展示
#### 要求
> HTML不得被前台全局样式污染

#### 解决方案
1、使用iframe 的 srcdoc
```html
<iframe srcdoc="<p>这是HTML<p>" frameborder="0"></iframe>
```
2、 src="text/html,xxxx"
```html
<iframe src="data:text/html,<p>这是HTML<p>" frameborder="0"></iframe>
```
3、 在src中使用javascript （ueditor的方案）
```html
<iframe 
	src="javascript:void(function(){
	document.open();document.write('<p>这是HTML<p>');
	document.close();}())" frameborder="0">
</iframe>
```
#### 问题
> 微信浏览器中以上方案都无效 似乎对iframe的src都进行了特殊处理 （惨）
# 最终方案
直接对iframe的contentWindow进行操作 使用document.write 最终解决问题 此方法应该兼容性最好
 ```html
 <iframe src="about:blank;" id="target" frameborder="0"></iframe>
 <script>	
 	document.getElementById('target').contentWindow.document.write('<p>这是HTML<p>') 
 </script>
 ```
*百谷一番都没有找到任何的解决方案 最终自己得出以下方案 记录一下 也为开发者们做一些微小的贡献（笑）


---