---
title: '利用Overflow:Hidden实现自适应左右两列布局/填充父级元素剩余宽度'
date: 2022-01-27 19:41:34
tags:
---
### HTML
***
	<div class="parent">
		<div class="Left"></div>
		<div class="Right"></div>
	</div>

### 左侧CSS
***
	.Left{
		width:auto;/* 左侧宽度任意 */
		float:left;
	}

### 右侧CSS
***
	.Right{
		overflow:hidden;
	}
	
---



以上！まいど～

---