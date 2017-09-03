---
layout:     post
title:      "js-css-slider"
subtitle:   "js和css的轮播实现方式"
date:       2017-09-03 12:00:00
author:     "Mickey"
header-img: "img/post-bg-js-module.jpg"
tags:
    - 前端开发
    - CSS
    - javaScript
---

博主最近使用react和python对豆瓣电影进行爬取，需要实现一个无限轮播的功能，即滚动到最后一个slider-item的时候，需要立即滑动到第一个item，在之前写`Cloud-Sing`项目的时候，博主使用的是纯CSS实现的方法，但是在`Douban-Movie`项目中，这个方法并不适用，我使用的是JS实现轮播的方法，下面博主就来介绍一下这两种实现无限轮播的方法

<h3>CSS实现轮播方法</h3>

下图就是博主用CSS实现无限轮播的Demo

![CSS-Slider](/img/in-post/js-css-slider/css-slider.gif)

这是CSS实现的源码，具体源码可见[Cloud-Sing](https://github.com/mickey0524/douban-movie)

```js
.banner{
	position: relative;
	width: 100%;
	height: 20rem;
	overflow: hidden;
	.swipe-photos {
		position: absolute;
		width: calc(100% * 4);
		.swipe-item {
			position: absolute;
			width: 25%;
			height: 20rem;
			img {
				width: 100%;
				height: 20rem;
			}
		}
		.swipe-item:first-of-type {
			animation: swipe_1 10s linear infinite;
		}
		.swipe-item:nth-of-type(2) {
			animation: swipe_2 10s linear infinite;
		}
		.swipe-item:nth-of-type(3) {
			animation: swipe_3 10s linear infinite;
		}
		.swipe-item:last-of-type {
			animation: swipe_4 10s linear infinite;
		}
	}
}

@keyframes swipe_1 {
    0%, 15% { transform: translate(0);  z-index: 1000; }
    25%, 40% { transform: translate(-100%); }
   	50%, 65% { transform: translate(0%); z-index: -1000; }
    75%, 90% { transform: translate(100%); z-index: -1000; }
    100% { transform: translate(0); z-index: 1000; }
}

@keyframes swipe_2 {
    0%, 15% { transform: translate(100%); }
    25%, 40% { transform: translate(0); z-index: 1000; }
   	50%, 65% { transform: translate(-100%); z-index: 1000; }
    75%, 90% { transform: translate(0); z-index: -1000; }
    100% { transform: translate(100%); }
}

@keyframes swipe_3 {
    0%, 15% { transform: translate(0); }
    25%, 40% { transform: translate(100%); }
   	50%, 65% { transform: translate(0); z-index: 1000; }
    75%, 90% { transform: translate(-100%); z-index: 1000; }
    100% { transform: translate(0); }
}

@keyframes swipe_4 {
    0%, 15% { transform: translate(-100%); }
    25%, 40% { transform: translate(0); }
   	50%, 65% { transform: translate(100%); z-index: -1000; }
    75%, 90% {  transform: translate(0); z-index: 1000; }
    100% { transform: translate(-100%); z-index: 1000; }
}
```

大体上思路就是，用CSS动画去控制每一个时间点的图片位置，造成一种无限轮播的错觉，在图片数量固定且不轻易更改的情况下，使用这种方式不失为可行之举，但是代码量过大，且不利于管理，在`Douban-Movie`项目中可行性几乎为0～

<h3>JS实现轮播方法</h3>

下图就是博主用js实现无限轮播的Demo

![JS-Slider](/img/in-post/js-css-slider/js-slider.gif)

这是js实现的大致源码，具体可见[Douman-movie](https://github.com/mickey0524/douban-movie)

```js
    direction *= SCROLL_SPEED;   //滚动速度
    this.isScroll[type] = true;
    let curLeft = Number(el.style.left.replace(/[^0-9]/g, ''));
    let proNum = 0;
    let step = () => {
      if (proNum < 100 / SCROLL_SPEED) {
        proNum += 1;
        let nowPrecent = -(curLeft + proNum * direction);
        el.style.left = nowPrecent + '%';
        if (nowPrecent <= -cycleNum) {
          el.style.left = '-100%';
        }
        if (nowPrecent >= 0) {
          el.style.left = -(cycleNum - 100) + '%';
        }
        requestAnimationFrame(step);
      }
      else {
        this.isScroll[type] = false;
      }
    }
    requestAnimationFrame(step);
```

JS源码过多，不好全部贴出，这里讲解一下大概思路，首先，第一页的图片肯定是在大数组的首部，最后一页的图片在大数组的尾部，那么按照常理来说，从最后一页滚动到第一页肯定会有长时间的动画，用户体验很差，思考一下，如果从一开始fetch数据的时候，就将第一页的图片复制一份放到最后，最后一页的图片复制一份放到最前，然后在banner滚动到相应复制块的时候，将left数值替换为对应的真实块的位置，这样实现的无限轮播就天衣无缝了～



