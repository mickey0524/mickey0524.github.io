---
layout:     post
title:      "webpack-learn"
subtitle:   "webpack学习"
date:       2017-09-26 18:30:00
author:     "Mickey"
header-img: "img/post-bg-os-metro.jpg"
tags:
    - 前端开发
---

虽然之前也会用webpack，但是没有系统的过一遍API，这里记录一下

* module中的rules数组 test推荐用正则表达式 include和exclude推荐用绝对地址组成的数组，include优先于exclude使用

```
var path = require('path');
module: {
  rules: [
    {
      test: /\.jsx?$/,
      include: [ path.resolve(__dirname, 'app') ],
      exclude: [ path.resolve(__dirname, 'app/demo-files')]
    }
  ]
}
```

* resolve 用于解决模块请求的选项（其实就是告诉webpack import module的时候去哪里找这个module），可以用于指定模块的查找位置（类似于python中的PYTHONPATH的东西吧），自动扩展后缀名，指定模块名的简写

```js
resolve: {
  module: [
    'node_modules',
    path.resolve(__dirname, 'app')
  ], // 查找模块的位置
  extensions: [ 'jsx', 'js', 'json', 'css' ],
  moduleExtensions: [ '-loader' ], //配置了这个之后，module.rules中的各种loader的引用就不用添加'-loader'后缀了
  alias: {
    'vue$': 'vue/dist/vue.esm.js'
    // $提供一种更加精确的搜索
    // import Vue from vue 这是对的
    // import Vue from vue/vue.js 这就是不行的，会去node_modules里面寻找
  }
}
```

* devServer 使用webpack的时候，可以使用webpack-dev-server起一个80端口的服务器，用来实时观看自己写的页面，devServer这个选项就是来配置一些可选项

```js
  devServer: {
    proxy: { // proxy URLs to backend development server
      '/api': 'http://localhost:3000'
    },
    contentBase: path.join(__dirname, 'public'), // boolean | string | array, static file location
    compress: true, // enable gzip compression
    historyApiFallback: true, // true for index.html upon 404, object for multiple paths
    hot: true, // hot module replacement. Depends on HotModuleReplacementPlugin
    https: false, // true for self-signed, object for cert authority
    noInfo: true, // only errors & warns on hot reload
    // ...
  }
```

* module.output 中最重要也是用的最多的三个属性 filename, path和publicPath，filename 指的是打包后的文件名，path是打包到的绝对路径，然而publicPath项则被许多Webpack的插件用于在生产模式下更新内嵌到css、html文件里的url值。

* devtool 用于控制source map，开发环境下，我个人喜欢使用`#eval-source-map`，生成环境下，将devtool设为false就行了

* webpack一些常用的插件

  1. extract-text-webpack-plugin 单独生成css文件
  2. webpack-stats-plugin 生成一个打包结果的映射
  3. webpack.optimize.CommonsChunkPlugin 提取公共代码
  4. webpack.optimize.UglifyJsPlugin 压缩代码
  5. webpack.LoaderOptionsPlugin 最小化loader

  [plugin-list](https://webpack.js.org/plugins/)

* webpack-dev-server

  多用于起一个express服务器处理静态文件，可以配置热更新，最好起两个express服务器，不然后端代码修改的时候，自动重启进程，webpack-dev-server就丢失缓存了，其实，个人觉得，没有特别大的好处