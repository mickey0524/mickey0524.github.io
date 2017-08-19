---
layout:     post
title:      "vue-component"
subtitle:   "Vue组件的创建，注册和使用"
date:       2017-08-19 18:30:00
author:     "Mickey"
header-img: "img/post-bg-js-module.jpg"
tags:
    - 前端开发
    - Vue
---

Vue是博主学习的第一个MVVM框架，当时学习的时候为了快速上手，加之有vue-cli十分的好用，对vue中组件相关的操作没有理解的非常透彻，最近公司用vue写一些后台管理界面，深刻的发现自己的基础不是很扎实，于是温习了一下，在此记录一下～

<h3>组件简介</h3>

组件系统是Vue.js其中一个重要的概念，它提供了一种抽象，让我们可以使用独立可复用的小组件来构建大型应用，任意类型的应用界面都可以抽象为一个组件树

![component](/img/in-post/post-vue-component/1.png)

<h3>组件的创建和注册</h3>

Vue组件的使用有三个基本的步骤：创建组件构造器，注册组件和使用组件

![component](/img/in-post/post-vue-component/2.png)

这是一段使用Vue的代码

```js
<!DOCTYPE html>
<html>
    <body>
        <div id="app">
            <!-- 3. #app是Vue实例挂载的元素，应该在挂载元素范围内使用组件-->
            <my-component></my-component>
        </div>
    </body>
    <script src="js/vue.js"></script>
    <script>
    
        // 1.创建一个组件构造器
        var myComponent = Vue.extend({
            template: '<div>This is my first component!</div>'
        })
        
        // 2.注册组件，并指定组件的标签，组件的HTML标签为<my-component>
        Vue.component('my-component', myComponent)
        
        new Vue({
            el: '#app'
        });
        
    </script>
</html>
```

我们用以下几个步骤来理解组件的创建和注册：

1. Vue.extend()是Vue构造器的扩展，调用Vue.extend()创建的是一个组件构造器。 

2. Vue.extend()构造器有一个选项对象，选项对象的template属性用于定义组件要渲染的HTML。 

3. 使用Vue.component()注册组件时，需要提供2个参数，第1个参数时组件的标签，第2个参数是组件构造器。 

4. 组件应该挂载到某个Vue实例下，否则它不会生效。

<h3>单页Vue应用和多页Vue应用的区别</h3>

单页的话，非常easy，首先，有vue-cli的存在，使用vue-cli初始化的项目，通过App.vue和main.js已经把组件App挂载到了全局的#app元素下，基本上只用去写相应的.vue组件就好，单页的.vue组件由template，script，style三部分组成，直接export default {}就行，十分easy，再次膜拜yyx大大。

```js
import Vue from 'vue'
import App from './components/App.vue'
import router from './router/index'
import store from './store/index'
import VueResource from 'vue-resource' 

Vue.use(VueResource);

new Vue({
  router,
  store,
  el: '#app',
  render: h => h(App) // template: <App></App>
})
```

多页的话，相对来说较为复杂一点，首先，共用的组件可以放到common目录下，使用Vue.extend()和Vue.component()全局组册好，对应路由的展示页面可以放到pagelet目录下，这里需要注意的是，由于不是单页应用，每个页面都需要使用new Vue()来初始化Vue对象

```js
import template from './index.html';
List = Vue.extend({
  template
})
new Vue({
  el: '#app',
  components: {
    List
  },
  template: '<list></list>'
});
```

<h3>总结</h3>

博主觉得Vue算是比较好入门的一个框架，但是也有它自己的特点，上手虽然容易，但是依然需要理解原理，不然换个环境就一窍不通了，希望看完这篇博客，看官能够有收获～



