---
title: Chrome DevTool 行号错位问题
date: 2018-12-20 10:43:16
tags:
---
随手记录

原因：
当你在一个js里面写document.write('\<script>多行内容xxxx\</script>')后
页面上的script断点调试会错位

表现就是断点打在某一行 实际停在上面几行
相差的行数取决于你写进文档的多行内容的行数


---