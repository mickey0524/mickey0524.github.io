---
layout:     post
title:      "tmux-introduction"
subtitle:   "tmux介绍"
date:       2017-11-14 20:00:00
author:     "Mickey"
header-img: "img/post-bg-e2e-ux.jpg"
tags:
    - 实用开发工具
---

### Tmux终端复用

你是否曾经开过一大堆的Terminal？有没有把它们都保存下来的冲动？Tmux 的Session就是做这件事情的！你可以随时退出或者进入任何一个Session。每个Session有若干个Window，每个Window又可以分成多个窗格（Pane）。

Tmux是一个终端复用软件，BSD协议发布。一般用于在一个命令行窗口中访问多个命令行会话，或者在一个命令行终端中同时使用多个程序。

* iTerm的窗格和Tmux有什么区别？

	我也很喜欢用iTerm的窗格，本地开发也很好用，但是在服务器上，要保持一个服务一直run，Tmux就有他的过人之处了，Tmux在一个命令行窗口中不仅可以显示多个Shell的内容，而且可以保持多个会话，最重要的是：Tmux和Vim一样属于字符终端软件，不需要任何GUI的支持，在远程登录的时候尤其有用。
	
* 如何在服务器上下载Tmux

	[Tmux的github地址](https://github.com/tmux/tmux)
	
	```
	$ git clone https://github.com/tmux/tmux.git
	$ cd tmux
	$ sh autogen.sh
	$ ./configure && make
	```

	过程可能会比较慢，但等一下就好啦，毕竟是源代码编译～
	
* 基本使用

	* tmux 创建一个默认名字的session，并且进入该session
	* tmux ls 查看当前tmux-server中有多少session
	* tmux kill-server 清空tmux-server中的所有session
	* tmux new -s name 创建一个以name为名字的session，并且进入该session
	* tmux a -t name 进入名字为name的session

* 基本配置

	* 默认的`<prefix>`是`Ctrl + b`，`<prefix>`是你在tmux中执行一些操作的前缀按键，如果你觉得不好按可以调整为`Ctrl + a`，只需要在配置文件`~/.tmux.conf`中加入:

		```
		unbind ^b
		set -g prefix 'C-a'
		```
	
	* 为了能让Tmux动态载入配置而不是重启，我们设一个快捷键<prefix>r来重新载入配置：
	
		```
		bind r source-file ~/.tmux.conf \; display-message "Config reloaded"
		```
		
		> 注意，通过`<prefix>`r重新载入配置并不等同于重启，只是增量地执行了配置文件中的所有命令而已。如果配置未生效，可以通过tmux kill-server来强行关闭Tmux。
		
	* 窗格切换
	
		可以把`hjkl`设置为切换窗格的快捷键
	
		```
		bind h select-pane -L
		bind j select-pane -D
		bind k select-pane -U
		bind l select-pane -R
		```
	
	* 设置打开新窗格的目录为当前目录

		```
		bind '"' split-window -c '#{pane_current_path}'
		bind '%' split-window -h -c '#{pane_current_path}'
		```
	
	* Tmux快捷键

		```
		<prefix>$ 重命名当前Session
		<prefix>c 创建新的窗口
		<prefix>% 水平分割窗口
		<prefix>" 垂直分割窗口
		<prefix>d 退出当前session
		<prefix>& kill当前session
		```
	
	* 我的tmux.conf

		```
		unbind ^b
		set -g prefix 'C-a'
		
		bind r source-file ~/.tmux.conf \; display-message "Config reloaded"
		
		bind h select-pane -L
		bind j select-pane -D
		bind k select-pane -U
		bind l select-pane -R
		
		unbind '"'
		bind - splitw -v
		unbind %
		bind | splitw -h
		
		bind '-' split-window -c '#{pane_current_path}'
		bind '|' split-window -h -c '#{pane_current_path}'
		```
	
### 参考

* [优雅地使用命令行：Tmux 终端复用](http://harttle.com/2015/11/06/tmux-startup.html)
* [Tmux 快捷键 & 速查表](https://gist.github.com/ryerh/14b7c24dfd623ef8edc7)
