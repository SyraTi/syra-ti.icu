---
title: '关于ajaxFileUpload造成 input[type=file] change事件只能触发一次的问题'
date: 2017-08-18 13:26:09
tags:
---
#### 现在发现的问题
通过js绑定的`input[type=file]` `change`事件只能触发一次
#### 原因
该问题并不是由change事件失效造成的，而是ajaxFileUpload插件造成的，它会把原来的file元素替换成新的file元素，所以之前绑定的change事件就失效了，需要重新绑定一下

除了重新绑定以外，如果是使用`$fileInput.trigger('click')` 方式，失效原因是`$fileInput`仍指向旧元素，替代方案分为以下两步
1. 首先将ajaxFileUpload源码中`$(oldElement).clone()`（有些版本中为`jQuery(oldElement).clone()`）改为`$(oldElement).clone(true)` 这样可以在复制元素的同时复制事件
2. 将`$fileInput.trigger('click')` 改为 `$('#id').trigger('click')`对保证新元素进行trigger而不是旧元素

如果你想要知道具体原因，请往下看

```javascript
/**
 * ajaxFileUpload源码
 */
 // 原input[type=file] 
 var oldElement = $('#' + fileElementId);
 // clone一份新元素 此处需要添加true为参数 即上文第一步所提到的 复制元素的同时复制事件
 var newElement = $(oldElement).clone(true);
 // 修改旧元素的id
 $(oldElement).attr('id', fileId);
 // 将新元素放在旧元素的位置
 $(oldElement).before(newElement);
 // 将旧元素移动到目标表单
 $(oldElement).appendTo(form);
 // 设置表单样式
 $(form).css('position', 'absolute');
 $(form).css('top', '-1200px');
 $(form).css('left', '-1200px');
 $(form).appendTo('body');
```

所以提交完后 原来上传的位置实际已经被newElement替代 但是如果在上传之前就指定了变量指向原input[type=file] ，那么上传之后将依旧指向oldElement（其实已经在表单里）,此时trigger会生效，但是该元素实际已经被ajaxFileUpload所删除 ,只存在于内存中，change事件似乎不会生效了（此处原因不详，如果知晓请告诉我）

#### 解决方案
### 1.（治标）将change事件写为内联`onchange="func()"`形式
### 2.（治标）将change事件重新绑定
### 3.（推荐 治本）将ajaxFileUpload内`$(oldElement).clone()`添加true为参数后 将`$fileInput.trigger('click')` 改为 `$('#id').trigger('click')`


---