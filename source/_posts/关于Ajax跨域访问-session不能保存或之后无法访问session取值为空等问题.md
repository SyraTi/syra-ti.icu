---
title: 关于Ajax跨域访问 session不能保存或之后无法访问session取值为空等问题
date: 2017-09-27 00:27:35
tags:
---
#### 根据浏览器的保护规则，**跨域的时候我们创建的sessionId是不会被浏览器保存下来的**，这样，当我们在进行跨域访问的时候，我们的sessionId就不会被保存下来，也就是说，每一次的请求，服务器就会以为是一个新的人，而不是同一个人，为了解决这样的办法，下面这种方法可以解决这种跨域的办法。

<!-- more -->
在ajax 请求要加配置
```javascript
	$.ajax({
		url:url,
		//加上 xhrFields及crossDomain
		xhrFields: {
			//允许带上凭据
	        withCredentials: true
		},
        crossDomain: true,
        //以上
		success:function(result){
			alert("test");
		},
		error:function(){
		}
	});
```
### 关于 withCredentials
>**withCredentials：**
默认情况下，跨源请求不提供凭据(cookie、HTTP认证及客户端SSL证明等)。通过将withCredentials属性设置为true，可以指定某个请求应该发送凭据。如果服务器接收带凭据的请求，会用下面的HTTP头部来响应。

>*"Access-Control-Allow-Credentials: true"*

>如果发送的是带凭据的请求，但服务器的相应中没有包含上面这个头部，那么浏览器就不会把相应交给JavaScript(于是，responseText中将是空字符串，status的值为0，而且会调用onerror()事件处理程序)。另外，服务器还可以在Preflight响应中发送这个HTTP头部，表示允许源发送带凭据的请求。

>支持withCredentials属性的浏览器有Firefox 3.5+、Safari 4+和Chrome。IE10及更早版本都不支持。
同时

### 注意
在添加基本的允许跨域响应头之后
需要添加 Access-Allow-Credentials:true
另外由于谷歌的安全策略
当withCredentials 为 true 时
ResponseHeader中的 Access-Allow-Origin 不能使用通配符 '*'
否则会提示
>A wildcard '*' cannot be used in the 'Access-Control-Allow-Origin' header when the credentials flag is true. Origin 'http://url' is therefore not allowed access.

其他浏览器待测试
 
---
