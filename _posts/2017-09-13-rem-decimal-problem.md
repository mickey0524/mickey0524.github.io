---
layout:     post
title:      "rem-decimal-problem"
subtitle:   "rem产生的小数问题"
date:       2017-09-13 20:00:00
author:     "Mickey"
header-img: "img/post-bg-e2e-ux.jpg"
tags:
    - 前端开发
---

今天，正撒欢写代码的时候，产品拿着手机走了过来，对我说道，你这按钮样式怎么不一样啊。。emmmmm，我说，不可能，代码用的都是一套，怎么可能不一样。。emmmm，拗不过PM爸爸，于是开始调试～

<b>rem</b>

rem是用来解决移动端网页的适配问题的，大大减少了移动端web页面的开发量，这篇博客是记录rem小数带来的像素问题，有关rem的概念，请大家自行百度

<b>小数像素</b>

1rem在width=320px的分辨率下的真实尺寸为32px，在width=360px的分辨率下的真实尺寸为36px，都是整数，这样计算就不会出现问题，如果一个DOM元素的width为1.75rem的话，情况会是怎么样的呢

| 代表机型           | 浏览器宽     | 对应尺寸  |
| -------------     |:----------:| --------:|
| iPhone 4/4s/5/5s  | 320px      | 56px     |
| Samsung Note 3    | 360px      | 63px     |
| iPhone6           | 375px      | 65.625px |
| Nexus 6           | 412px      | 72.1px   |
| iphone6 plus      | 414px      | 72.45px  |

可以看到，1.75rem对应的真实物理尺寸有很多有小数点，那么浏览器是如何识别小数点的呢

```js
<div class="container">
	<div class="block"></div>
	<div class="block"></div>
	<div class="block"></div>
	<div class="block"></div>
	<div class="block"></div>
</div>

.block {
	display: inline-block;
	width: 1.75rem;
	height: 1.75rem;
}
```

讲道理，上面的五个.block渲染的结果应该是一样的，均为65.625px，但是请看，实际渲染出来的数值既有65px，也有66px，最关键的是毫无规律，emmmm

![1](/img/in-post/rem-decimal/1.jpg)

我参考淘宝FE[mobile-rem-problem](http://taobaofed.org/blog/2015/11/04/mobile-rem-problem/)，得出如下结论

浏览器在渲染时所做的舍入处理只是应用在元素的渲染尺寸上，其真实占据的空间依旧是原始大小。

也就是说如果一个元素尺寸是 0.625px，那么其渲染尺寸应该是 1px，空出的 0.375px 空间由其临近的元素填充；同样道理，如果一个元素尺寸是 0.375px，其渲染尺寸就应该是 0，但是其会占据临近元素 0.375px 的空间。于是就顺着这个思路验证了以下：

1. 第一个色块的宽度为 65.625px，根据四舍五入的原则其最终渲染尺寸为 66px，空出的 0.375px 由第二个色块补上；
2. 第二个色块向左补进 0.375px，相当于减少了 0.375px，余下 65.25px，根据四舍五入的原则其最终渲染尺寸为 65px，多出的 0.25px 会占用第三个色块的空间；
3. 第三个色块被占用了 0.25px，相当于增加了 0.25px，等于 65.875px，根据四舍五入的原则其最终渲染尺寸为 66px，空出的 0.125px 由第四个色块补上；
4. 第四个色块向左补进 0.125px，相当于减少了 0.125px，余下 65.5px，根据四舍五入的原则其最终渲染尺寸为 66px，空出的 0.5px 由第五个色块补上；
5. 第五个色块向左补进 0.5px，相当于减少了 0.5px，余下 65.125px，根据四舍五入的原则其最终渲染尺寸为 65px，多出 0.125px；

所以怎么办呢，尽量少出现小数呗～

<b>问题</b>

同一组 icon 在不同机型下显示的不一样，有些手机或多或少的会被裁掉一部分，原因就是由于小数像素导致的，这点可以从元素的 Computed Style 上看出。

要避免这种问题的话，在设置background-image的时候，记得给元素多留一些空隙，这样可以有效的防止因为小数点导致的渲染差异

![3](/img/in-post/rem-decimal/3.png)



