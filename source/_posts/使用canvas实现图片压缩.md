---
title: 使用canvas实现图片压缩
date: 2017-11-09 10:10:05
tags:
---
## 原文链接 https://segmentfault.com/a/1190000009308553
那，首先通过`URL.createObjectURL(file)`从file对象直接取得了图片的地址

前面就不详细说了，开始压缩咯 ( ´ ▽ ` )ﾉ

噢，有个注意点:

每次调用`createObjectURL`的时候，一个新的URL对象就被创建了，即使是同一个file对象，也会创建一个新对URL对象，所以，为了最佳性能和内存使用，当不再需要这个对象的时候要`URL.revokeObjectURL()`释放它。

开始压缩

<!-- more -->
创建一个`compressImage`函数，将之图片的地址url作参数传入：
```js
	compressImage (url) {
		let cvs = document.createElement('canvas')
		let ctx = cvs.getContext('2d')
		let img = new window.Image()
		img.src = url
		img.onload = () => {
			cvs.width = img.width
			cvs.height = img.height
			setTimeout(() => {
				ctx.drawImage(img, 0, 0, cvs.width, cvs.height)
				this.newImageData = cvs.toDataURL('image/jpeg', 0.1)
		    }, 0)
		    this.showPreviewer = true
		}
	}
```
这里说说

`canvas.toDataURL(type, encoderOptions)`
`HTMLCanvasElement.toDataURL()`方法接受两个参数，`type`和`encoderOptions`

`type`是可选的，图片格式，默认是 `image/png`，`encoderOptions`表示图片质量， 在`type`为`image/jpeg` 或 `image/webp`时可以从 0 到 1 的区间内选择图片的质量。如果超出取值范围，将会使用默认值 `0.92`。其他参数会被忽略。

`toDataURL()`返回的是`base64`字符串，如果要转成2进制
```js
	convertToBinary (dataURI) {
		let byteString = window.atob(dataURI.split(',')[1])
		let ab = new ArrayBuffer(byteString.length)
		let ia = new Uint8Array(ab)
		for (let i = 0; i < byteString.length; i++) {
			ia[i] = byteString.charCodeAt(i)
		}
		let bb = new window.Blob([ ab ])
		return bb
	}
```
测试结果：由iphone6所拍摄的图片上传，由平均1.9M左右压缩至170k

*･゜ﾟ･*:.｡..｡.:*･'(*ﾟ▽ﾟ*)'･*:.｡. .｡.:*･゜ﾟ･*

再见
5月5日发布


---
