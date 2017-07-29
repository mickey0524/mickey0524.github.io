---
layout:     post
title:      "linear-gradient"
subtitle:   "CSS之线性变化"
date:       2017-07-29 17:00:00
author:     "Mickey"
header-img: "img/post-bg-2015.jpg"
tags:
    - 前端开发
    - CSS
---

近期懒癌又犯了，完全没有更博的动力QAQ。好吧，言归正传，最近在做项目的时候，有学习到linear-gradient这个属性，觉得很有意思，于是来记录一下～
<br>
<br>------------
<br>
<br><b>linear-gradient</b>
<br>
<br>CSS linear-gradient()函数创建一个表示颜色线性变化的image，该函数的结果是CSS gradient 数据类型的对象。像任何其他渐变，CSS线性渐变不是一个CSS  color ，而是一个没有内在尺寸的图像; 也就是说，它既不具有 固有的或首选的尺寸，也不具有比率。它的具体尺寸将与其适用的元素的尺寸匹配，一般来说，该属性会用在background-image中。
<br>
<br>linear-gradient()的语法为（angle, color (length | percent | ''))，angle表示该线性变化的方向，该属性以
竖直向上为0deg，沿顺时针角度增加，水平向右为90deg，也可以使用to的语法，to top为0deg, to bottom right为135deg，
以此类推
<br>
<br>linear-gradient( 45deg, blue, red );           /* A gradient on 45deg axis starting blue and finishing red */
linear-gradient( to left top, blue, red);      /* A gradient going from the bottom right to the top left starting blue and 
                                                  finishing red */
linear-gradient( 0deg, blue, green 40%, red ); /* A gradient going from the bottom to top, starting blue, being green after 40% 
                                                  and finishing red */
<br>
<br>这里需要注意的是，color后面可以跟百分数，也可以跟数字，也可以啥都不跟，百分数，就是相对起点的比例，啥都不跟的话，自动会取本颜色位置和前一个
颜色位置的中间数，数字的话只能为0，其他的会自动剔除该属性，0的意思就是起点紧贴前一个颜色，没有过渡渐变的意思
<br>如果要实现下面这个图的效果
<br>linear-gradient(135deg/to bottom right, #26ce61, #26ce61 50%, transparent 0/50%)都阔以

![example]('../img/in-post/linear-gradient.png')

<br>
<br>------------
<br>下面给出参考资料
<ul>
    <li><a href="https://developer.mozilla.org/zh-CN/docs/Web/CSS/linear-gradient" class=" wrap external" target="_blank" rel="nofollow noreferrer">MDN linear-gradient<i class="icon-external"></i>
        <br>
    </li>
    <li><a href="http://lea.verou.me/css3patterns/" class=" wrap external" target="_blank" rel="nofollow noreferrer">CSS Gradients Patterns Gallery, by Lea Verou<i class="icon-external"></i>
        <br>
    </li>
</ul>






