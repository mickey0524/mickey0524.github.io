---
layout:     post
title:      "初识CSS抛物线动画"
subtitle:   "Get started with CSS parabolic animation"
date:       2017-08-06 14:00:00
author:     "Mickey"
header-img: "img/post-bg-rwd.jpg"
tags:
    - 前端开发
    - CSS
---

上周笔者在完成自己的react-ele项目的时候，需要实现点餐的时候，商品以抛物线的形式落入购物车的动画。咋一看，似乎没有头绪，以前做的动画都是水平垂直运行的，仔细思考之后，其实抛物线运动也能分解成为水平方向的匀速运动和竖直方向的加速度运动，于是我们只用定义两个方向的动画即可，笔者随即去谷歌了一发，大家的思路也和我差不多，下面就给出具体的方法
<br>
<br>
首先要感谢这两篇对我启发比较大的博客文章，分别是<a href="http://blog.csdn.net/boycycyzero/article/details/44088707" target="_blank">CSS3动画之抛物线运动</a>和<a href="http://cloudstone.xin/2015/08/02/%E4%BD%BF%E7%94%A8CSS3%E5%AE%9E%E7%8E%B0%E6%8A%9B%E7%89%A9%E7%BA%BF%E6%95%88%E6%9E%9C--%E8%87%AA%E5%AE%9A%E4%B9%89%E8%B4%9D%E5%A1%9E%E5%B0%94%E6%9B%B2%E7%BA%BF/" target="_blank">使用CSS3实现抛物线效果--自定义贝塞尔曲线</a>，后来在张鑫旭大大的博客里发现了另外一个神奇，可以动态预览贝塞尔曲线的效果<a href="http://cubic-bezier.com/">贝塞尔神器</a>，其实在chrome开发者工具里面，也有动画曲线的预览图，看个人爱好啦。
<br>
<br>
好的，废话不多说，下面开始正式介绍抛物线动画，首先，和上面讲的一样，我们需要将抛物线运动分解成为水平方向的匀速运动和竖直方向上的加速度运动，如下图所示
<br>
<br>
水平方向上是匀速运动的，那么它的运动动画应该也是linear的
![hor](/img/in-post//post-css-bezier/hor.png)
<br>
垂直方向上我的项目里是需要实现一个先向上减速到0，再向下加速的动画，于是它的运动轨迹如图
![hor](/img/in-post/post-css-bezier/vertical.png)
<br>
这里需要注意的是，<b>不要把上面的用于描述“一个维度”的动画完成的过程，与我们这里的抛物线的轨迹混淆！我之前就是卡在了这个误区！</b>
<br>
好的，现在给出一张实际效果图，我们再来分析代码是如何实现的
![ele](/img/in-post/post-css-bezier/ele.gif)
<br>
<br>
大体上，容易想到的思路不外乎使用transition控制left和top的变化，使用transition控制transfrom的translate的变化以及使用animation，其实思路都比较简单
<br>
1.使用transition控制left和top的变化

```js

  .animated-ball {
    position: absolute;
    width: .8rem;
    height: .8rem;
    background: #3190e8;
    border-radius: 50%;
    transition: left .6s linear, top .6s cubic-bezier(0.5, -0.5, 1, 1);
    &.animation {
      left: 1.2rem!important;
      top: 26rem!important;
    }
  }
```
<br>
2.使用transition控制transform的变化，由于translateX和translateY都是transform的，因此需要多加一个节点

```js
section.animate {
  transition: transform .5s linear;
  transform: translateX(200px);
}

.animate-ball {
   transition: transform 0.5s cubic-bezier(0.5, 
   transform: translateY(200px);
}
```
<br>
3.使用animation控制transform的变化

```js
section.animate {
  animation: hor-animation linear 0.5s forwards;
}
.animate-ball {
  animation: vertical-animation cubic-bezier(0.5, -0.5, 1, 1) 0.5s forwards;
}
@keyframes hor-animation {
  0% {
    transform: translateX(0px);
  }
  100% {
    transform: translateX(200px);
  }
}
@keyframes vertical-animation {
  0% {
    transform: translateY(0px);
  }
  100% {
    transform: translateY(200px);
  }
}
```
<br>
<br>
总结一下:

1.尽可能的使用CSS3实现动画，浏览器会帮助我们优化

2.如果不能避免基于javascript的动画，避免使用setTimeout, setInterval，尽量使用requestAnimationFrame，浏览器同样会优化这个过程

3.使用2D transforms代替定位方式（translate能够实现“亚像素”，定位则不能，最小单位为1px），将带来更高的FPS与更少的重绘，使动画更流畅，使用translate会启动GPU加速，有时在动画中，为了得到更好的效果，可以使用translateZ(0)开启硬件加速