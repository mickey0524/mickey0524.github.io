---
layout:     post
title:      "页面滚动时自动显示隐藏导航效果"
subtitle:   "scroll-auto-show-hide"
date:       2017-09-23 20:00:00
author:     "Mickey"
header-img: "img/post-bg-digital-native.jpg"
tags:
    - 前端开发
    - CSS
    - javaScript
---

页面在用户滚动的时候自动显示和隐藏，这种交互方式已经出现很久了，甚至有很多大公司都在使用，这种交互希望导航在需要的时候很方便的显示，当用户向下滚动页面时，导航会自动隐藏，为内容留出更多的空间。在用户向上滚动页面时，将他的行为理解为想要访问导航，所以把导航自动自动显示出来，在本篇博客中会带来简要的实现方式

## CSS实现流畅的目录打开，关闭样式切换

如图所示，下面的图片切换可以用于侧栏的打开与关闭，看起来就很舒服

![1](/img/in-post/scroll-auto-show-hide/1.gif)

下面给出实现的代码，清晰易懂，就不做过多的解释了

```
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>css-directory-animation</title>
  <style type="text/css">
    body {
      padding-top: 20px;
    }
    .nd-animation {
      display: block;
      position: relative;
      width: 22px;
      height: 2px;
      background-color: #25283D;
      transition: background-color .2s linear;
    }
    .nd-animation::before {
      position: absolute;
      content: '';
      width: 22px;
      height: 2px;
      background: #25283D;
      transform: translateY(-6px);
      transition: transform .2s linear;
    }
    .nd-animation::after {
      position: absolute;
      content: '';
      width: 22px;
      height: 2px;
      background: #25283D;
      transform: translateY(6px);
      transition: transform .2s linear;
    }
    .is-open {
      background-color: rgba(255, 255, 255, 0);
    }
    .is-open::before {
      transform: rotate(-45deg);
    }
    .is-open::after {
      transform: rotate(45deg);
    }
  </style>
</head>
<body>
  <span class="nd-animation"></span>
  <script type="text/javascript">
    var node = document.getElementsByClassName('nd-animation')[0];
    window.addEventListener('click', function() {
      if (node.classList.contains('is-open')) {
        node.classList.remove('is-open');
      }
      else {
        node.classList.add('is-open');
      }
    }, false);
  </script>
</body>
</html>
```

## 页面滚动时自动显示隐藏导航效果

这类导航栏自动隐藏和显示常见的有三种形式

* 只有一个导航栏，下滑显示，上滑隐藏
* 有两个导航栏，下滑的时候，主导航栏显示，上滑的时候，主导航栏隐藏，子导航栏吸顶
* 有两个导航栏和一张大图，下滑的时候，主导航栏显示，上滑的时候，当大图消失在视口，主导航栏隐藏，子导航栏吸顶

下面我们依次介绍这三种情况

### 主导航

这种情况非常简单，利用js监听window的scroll事件，比较本次和上次滚动的距离，判断用户滑动的方向，控制导航栏显示和隐藏即可

```
<header class="cd-auto-hide-header">
	一级header
</header>

header {
	position: fixed;
	left: 0;
	right: 0;
	top: 0;
	height: 60px;
	background: #25283D;
	color: #FFF;
	line-height: 60px;
	transition: transform .5s;
	transform: translateZ(0);
}
.is-hide {
	transform: translateY(-100%);
}
main {
	background: #ECF0F1;
	color: #A5A8A9;
	padding: 20px 100px 20px;
}

var header = $('header');
var scrollDelta = 10;
var scrollOffset = 200;
var isScroll = false;
var previousTop = 0;
var currentTop = 0;


$(window).scroll(function() {
	if (!isScroll) {
		isScroll = true;
		(window.requestAnimationFrame)
			? requestAnimationFrame(autoHideHeader)
			: setTimeout(autoHideHeader, 250);
	}
});

function autoHideHeader() {
	currentTop = $(window).scrollTop();

	var distance = main.offset().top - header.height() - sub.height();
	if (previousTop >= currentTop) {
		if (previousTop - currentTop >= scrollDelta) {
			header.removeClass('is-hide');
		}
	}
	else {
		if (currentTop - previousTop >= scrollDelta && currentTop > scrollOffset) {
			header.addClass('is-hide');
		}
	}

	previousTop = currentTop;
	isScroll = false;
}
```

![nav-1](/img/in-post/scroll-auto-show-hide/nav-1.gif)

### 主导航 + 子导航

这个场景和仅有主导航几乎完全一样，这里有一个黑魔法，就是将子导航写到主导航里面，这样处理起来就非常方便

```
<header class="cd-auto-hide-header">
	一级header
	<div class="sub-nav">二级header</div>
</header>

header {
	position: fixed;
	left: 0;
	right: 0;
	top: 0;
	height: 60px;
	background: #25283D;
	color: #FFF;
	line-height: 60px;
	transition: transform .5s;
	transform: translateZ(0);
}
.sub-nav {
	line-height: 50px;
	background: pink;
	color: #FFF;
	height: 50px;
}
.is-hide {
	transform: translateY(-100%);
}
main {
	background: #ECF0F1;
	color: #A5A8A9;
	padding: 20px 100px 20px;
}

var header = $('header');
var scrollDelta = 10;
var scrollOffset = 200;
var isScroll = false;
var previousTop = 0;
var currentTop = 0;


$(window).scroll(function() {
	if (!isScroll) {
		isScroll = true;
		(window.requestAnimationFrame)
			? requestAnimationFrame(autoHideHeader)
			: setTimeout(autoHideHeader, 250);
	}
});

function autoHideHeader() {
	currentTop = $(window).scrollTop();

	var distance = main.offset().top - header.height() - sub.height();
	if (previousTop >= currentTop) {
		if (previousTop - currentTop >= scrollDelta) {
			header.removeClass('is-hide');
		}
	}
	else {
		if (currentTop - previousTop >= scrollDelta && currentTop > scrollOffset) {
			header.addClass('is-hide');
		}
	}

	previousTop = currentTop;
	isScroll = false;
}
```

![nav-1](/img/in-post/scroll-auto-show-hide/nav-2.gif)

### 主导航 + 子导航 + 图片

这种情况比前两种复杂一些，但也还好，主要就是要根据滚动的距离，修改子导航的样式～

```
<header class="cd-auto-hide-header">
	一级header
</header> <!-- .cd-auto-hide-header -->

<div class="big-img"></div>

<div class="sub-nav">二级header</div>

header {
	position: fixed;
	left: 0;
	right: 0;
	top: 0;
	height: 60px;
	background: #25283D;
	color: #FFF;
	line-height: 60px;
	transition: transform .5s;
	transform: translateZ(0);
}
.big-img {
	height: 400px;
	width: 100%;
	background: no-repeat center center;
	background-image: url('./2.jpg');
	background-size: cover;
	margin-top: 60px;
}
.sub-nav {
	line-height: 50px;
	background: pink;
	color: #FFF;
	height: 50px;
}
.sub-nav-fixed {
	position: fixed;
	top: 60px;
	left: 0;
	right: 0;
	transition: transform .5s;
	transform: translateZ(0);
}
.sub-nav-slide {
	transform: translateY(-60px);
}
.is-hide {
	transform: translateY(-100%);
}
main {
	background: #ECF0F1;
	color: #A5A8A9;
	padding: 20px 100px 20px;
}
.main-sub-fixed {
	margin-top: 50px;
}

var header = $('header');
var sub = $('.sub-nav');
var main = $('main');
var scrollDelta = 10;
var scrollOffset = 200;
var isScroll = false;
var previousTop = 0;
var currentTop = 0;


$(window).scroll(function() {
	if (!isScroll) {
		isScroll = true;
		(window.requestAnimationFrame)
			? requestAnimationFrame(autoHideHeader)
			: setTimeout(autoHideHeader, 250);
	}
});

function autoHideHeader() {
	currentTop = $(window).scrollTop();

	var distance = main.offset().top - header.height() - sub.height();

	if (previousTop >= currentTop) {
		if (currentTop < distance) {
			header.removeClass('is-hide');
			sub.removeClass('sub-nav-slide sub-nav-fixed');
			main.removeClass('main-sub-fixed');
		}
		else if (currentTop >= distance && previousTop - currentTop >= scrollDelta) {
			header.removeClass('is-hide');
			sub.addClass('sub-nav-fixed').removeClass('sub-nav-slide');
			main.addClass('main-sub-fixed');
		}
	}
	else {
		if (currentTop > distance + scrollOffset) {
			sub.addClass('sub-nav-slide');
			header.addClass('is-hide');
		}
		else if (currentTop > distance) {
			sub.addClass('sub-nav-fixed');
			main.addClass('main-sub-fixed');
		}
	}

	previousTop = currentTop;
	isScroll = false;
}
```

![nav-1](/img/in-post/scroll-auto-show-hide/nav-3.gif)

### 参考

[滚动自动显示和隐藏](http://www.css88.com/archives/7883)



