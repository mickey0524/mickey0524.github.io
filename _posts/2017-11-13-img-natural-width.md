---
layout:     post
title:      "img-natural-width"
subtitle:   "用JavaScript获取图片的原始宽高"
date:       2017-11-13 12:00:00
author:     "Mickey"
header-img: "img/post-bg-rwd.jpg"
tags:
    - 前端开发
    - javaScript
---

前几天写vue的瀑布流插件的时候，思考了一个问题，如何才能让大小不一的图片在瀑布流中自适应的摆放。我写的插件是让瀑布流每一列保持宽度一致，高度通过缩放自适应，那么问题来了，如何判断一个图片的原始大小呢。

* 现在很多公司都有图片裁剪自适应的服务，可以通过在url上添加参数，用以自适应，这个就不需要获取原始宽高，直接前端显示的时候补充url参数即可
* HTML5提供了一个新属性naturalWidth/naturalHeight可以直接获取图片的原始宽高。这两个属性在Firefox/Chrome/Safari/Opera及IE9里已经实现。改造下获取图片尺寸的方法

	```js
	if (img.naturalWidth) { // 现代浏览器
	    nWidth = img.naturalWidth
	    nHeight = img.naturalHeight
	}
	```
* IE8以下没有naturalWidth和naturalHeight，只能通过创建一个空的Image类，然后设置src属性，图片onload之后，获取宽高；同理，如果一个图片还没有被加载进DOM文档，比如我的vue瀑布流插件，那么也只能通过这种方式，获取宽高，再进行对应的缩放适应。
* 相关

	<a href="http://www.cnblogs.com/snandy/p/3704218.html" target="_blank">http://www.cnblogs.com/snandy/p/3704218.html</a>
	<a href="http://www.cnblogs.com/snandy/archive/2011/09/06/2167440.html" target="_blank">http://www.cnblogs.com/snandy/archive/2011/09/06/2167440.html</a>
	