---
title: JS实现点击按钮，复制文本框中的的内容
date: 2017-07-28 09:04:03
tags:
---
### Html
***
	<textarea cols="20" rows="10" id="anyInput">用户定义的代码区域</textarea>
	<button type="button" onClick="copy()" value="点击复制代码" />

### JS
***
	function copyUrl2()
	{
		var target=document.getElementById("anyInput");
		target.select(); // 选择文本
		document.execCommand("Copy"); // 执行浏览器复制命令
		alert("已复制好，可贴粘。");
	}
	
---