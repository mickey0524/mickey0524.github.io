---
layout:     post
title:      "移动端video标签踩坑总结"
subtitle:   "mobile-video-control"
date:       2017-09-10 18:00:00
author:     "Mickey"
header-img: "img/post-bg-rwd.jpg"
tags:
    - 前端开发
    - 移动端
---

今天在公司为feed流增加视频文章和视频卡片，调研了一下video.js，觉得过于厚重，便决定使用原生video来实现。。。video的坑实在太多了。。。下面对一些问题记录一下，进行备忘，以便以后查看

### 在html5页面中播放视频

```
<video poster="封面图片" width="300" height="150">
  <source src="x.mp4" type="video/mp4">
  <source src="x.webm" type="video/webm">
  <source src="x.ogg" type="video/ogg">
</video>
```

### IOS中默认全屏播放

可以通过向video添加`webkit-playsinline playsinline`来使视频可以小屏播放

其中webkit-playsinline是为了兼容ios10之前，ios10之后，apple终于松口，使用playsinline就行

### video中心自带播放按钮

其实这倒不是什么大问题，但是为了样式统一，以及方便监控事件，可以在css中取消这个按钮

```
video::-webkit-media-controls-start-playback-button { //video视频不显示默认的中心的play icon
  opacity: 0;
}
```

### android video暂停的时候video中心显示播放按钮

这个无力吐槽，隐藏方法

```
video::-webkit-media-controls-overlay-play-button { //隐藏android端点击control中暂停按钮时视频中心出现的play icon
  display: none;
}
```

### 隐藏controls中的音量控制

```
video::-webkit-media-controls-volume-slider, video::-webkit-media-controls-mute-button { //隐藏android端video标签自带的音量调节按钮
  display: none;
}
```

### 其他关于controls相关的控制

参考[Why do no user-agents implement the CSS cursor style for video elements](https://stackoverflow.com/questions/15126921/why-do-no-user-agents-implement-the-css-cursor-style-for-video-elements/15145555#15145555)

* video::-webkit-media-controls-panel
* video::-webkit-media-controls-play-button
* video::-webkit-media-controls-volume-slider-container
* video::-webkit-media-controls-volume-slider
* video::-webkit-media-controls-mute-button
* video::-webkit-media-controls-timeline
* video::-webkit-media-controls-current-time-display
* video::-webkit-full-page-media::-webkit-media-controls-panel
* video::-webkit-media-controls-timeline-container
* video::-webkit-media-controls-time-remaining-display
* video::-webkit-media-controls-seek-back-button
* video::-webkit-media-controls-seek-forward-button
* video::-webkit-media-controls-fullscreen-button
* video::-webkit-media-controls-rewind-button
* video::-webkit-media-controls-return-to-realtime-button
* video::-webkit-media-controls-toggle-closed-captions-button

### 目前待解决的问题

讲道理来说，全屏的时候，应该另开一个layout来播放，但是在android端发现，全屏退出的时候，会影响当前webview的视口，难受的一匹，我目前的处理办法相当暴力，并且不可取，是在监听到全屏事件的时候，记录位置，然后document.body.scrollTop滚动回去。。

哪位小伙伴知道怎么处理的话，可以邮箱告诉我～ buptbh@163.com