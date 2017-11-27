---
layout:     post
title:      "fe-download"
subtitle:   "web前端下载"
date:       2017-11-27 21:30:00
author:     "Mickey"
header-img: "img/post-bg-alitrip.jpg"
tags:
    - 前端开发
---

前端的很多项目中，都有文件下载的需求，特别是JS生成文件内容，然后让浏览器执行下载操作，很多情况下，我们只能给出一个链接，让用户点击打开-》另存为

`<a href=”file.js”>file.js</a>`

用户点击这个链接的时候，浏览器会打开并显示链接指向的文件内容，显然，这并没有实现我们的需求

HTML5中给a标签增加了一个download属性，只要有这个属性，点击这个链接时浏览器就不在打开链接指向的文件，而是改为下载，下载时会直接使用链接的名字来作为文件名，但是是可以改的，只要给download加上想要的文件名即可，如：download=“not-a-file.js”，通过这种方法，可以下载图片，静态js文件，mp3格式音频，一切放置在服务器上有访问路径的资源都可以下载

### 前端中的Blog对象

> 一个 Blob对象表示一个不可变的, 原始数据的类似文件对象。Blob表示的数据不一定是一个JavaScript原生格式。 File 接口基于Blob，继承 blob功能并将其扩展为支持用户系统上的文件。

以下载图片为例，我们都知道，python爬虫可以很轻易的通过urllib.urlopen来获取图片的二进制数据，然后写入文件完成下载，前端的Blog对象实现的其实是相似的功能

Blog对象是二进制大数据对象，详细说明可以参考这里[这里](https://developer.mozilla.org/en-US/docs/Web/API/Blob)，可以用于创建dataurl，进而可以作为a标签的href数值，完成上述相似的下载功能

通过XHR请求

```
const xhr = new XMLHttpRequest();
xhr.open('GET', 'http://www.baidu.com/img/bd_logo1.png');
xhr.responseType = 'blob';
xhr.onload = () => {
  console.log(xhr);
  const blob = xhr.response;
  var a = document.createElement('a');
  var url = window.URL.createObjectURL(blob);
  var filename = 'logo.png';
  a.href = url;
  a.download = filename;
  a.click();
  window.URL.revokeObjectURL(url);
}
xhr.send();
```

通过Fetch请求

```
fetch('http://dl.stream.qqmusic.qq.com/C400002oJLcZ1c7Cyf.m4a?vkey=7A8ED97F90BB5508D853E37DF27D5400476DC00AFDA796EFAC475FEEB748B834920BBFB824DCC50DB743FB45344D04CF45C86CE1D5678D42&guid=3731340815&uin=0&fromtag=66').then(res => res.blob()).then(blob => {
  var a = document.createElement('a');
  var url = window.URL.createObjectURL(blob);
  var filename = 'dayu.mp3';
  a.href = url;
  a.download = filename;
  a.click();
  window.URL.revokeObjectURL(url);
});
```

上面的`window.URL.createObjectURL`方法接收blog对象作为参数，生成一个dataurl，使用完毕之后，调用`window.URL.revokeObjectURL`方法释放对该dataurl的应用引用

