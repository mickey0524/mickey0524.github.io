---
layout:     post
title:      "vue-source-code-analysic（1）"
subtitle:   "阅读Vue源码有感（1）"
date:       2017-09-12 23:00:00
author:     "Mickey"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
tags:
    - 前端开发
    - Vue
---

最近，正在有条不紊的对Vue进行第二次学习，前不久，比较细致的学习了Vue组件相关的知识，包括Vue.extend()生成一个Vue组件的构造器，Vue.component()对Vue的组件进行全局组册...随后，对照Vue的官方文档，学习了平时不常用的一些API，对Vue的生命周期有了更加深刻的理解，最近几天，阅读了各位大大对于Vue源码的总结，在此记录一下，作为自己阅读Vue源码的基础吧～

<b>Vue的响应式原理</b>

Vue实现响应式最重要的函数是Object.defineProperty()，这也是Vue为什么不支持IE8以下的原因，Vue通过设定对象属性的setter/getter方法来监听数据的变化，通过getter进行依赖收集，而每个setter方法就是一个观察者，在数据变更的时候通知订阅者更新视图。

下面就是Vue将data变为可观察的方法

```js
function observer(value) {
    Object.keys(value).forEach((key) => defineReactive(value, key, value[key] , cb))
}

function defineReactive (obj, key, val, cb) {
    Object.defineProperty(obj, key, {
        enumerable: true,
        configurable: true,
        get: ()=>{
            /*....依赖收集等....*/
            /*Github:https://github.com/answershuto*/
        },
        set:newVal=> {
            cb();/*订阅者收到消息的回调*/
        }
    })
}

class Vue {
    constructor(options) {
        this._data = options.data;
        observer(this._data, options.render)
    }
}

let app = new Vue({
    el: '#app',
    data: {
        text: 'text',
        text2: 'text2'
    },
    render(){
        console.log("render");
    }
})
```

这是最简单的情况，但是言简意赅，容易理解，现在我们就可以通过修改app._data.text修改data数据对视图进行重绘，但是Vue中都是把data挂载在VM实例上的，是如何实现的呢？

答案是调用一个proxy()代理函数，将app._data中的数据代理到app上

```js
_proxy(options.data);/*构造函数中*/

/*代理*/
function _proxy (data) {
    const that = this;
    Object.keys(data).forEach(key => {
        Object.defineProperty(that, key, {
            configurable: true,
            enumerable: true,
            get: function proxyGetter () {
                return that._data[key];
            },
            set: function proxySetter (val) {
                that._data[key] = val;
            }
        })
    });
}
```

这样，我们就可以使用app.text代替app._data.text了

<b>Vue的依赖收集</b>

可能有同学会问，上面不是已经将Vue的响应式原理讲的很清楚了嘛，为什么还需要依赖收集呢，这不是多此一举吗，可是，仔细想想，我们可以在Vue的data中定义无穷多的数据，然后在render过程中，真正用到的数据寥寥无几，却因为修改了useless的数据而频繁重绘，这是不能忍受的，因此，Vue使用一个Dep依赖收集类来判断发生变化的数据在DOM中是否被用到，被用到的话，才进行这次重绘，反之，则忽略数据的修改

下面说说Dep类，当对data上的对象进行修改值的时候会触发它的setter，那么取值的时候自然就会触发getter事件，所以我们只要在最开始进行一次render，那么所有被渲染所依赖的data中的数据就会被getter收集到Dep的subs中去。在对data中的数据进行修改的时候setter只会触发Dep的subs的函数，每一个存在于data对象中的数据都会包含一个dep类～

```js
class Dep () {
    constructor () {
        this.subs = []; // 订阅者
    }

    addSub (sub: Watcher) {
        this.subs.push(sub)
    }

    removeSub (sub: Watcher) {
        remove(this.subs, sub)
    }
    /*Github:https://github.com/answershuto*/
    notify () { //当data中的数据执行setter方法的时候，调用该方法，通知watch订阅者重绘
        // stabilize the subscriber list first
        const subs = this.subs.slice()
        for (let i = 0, l = subs.length; i < l; i++) {
            subs[i].update()
        }
    }
}
```

可能有同学看到上图代码中的subs数据会产生疑问，其实这就是用来存放watch订阅者对象的，因为一个data数据可能有多个watch，因此需要使用数组，下面给出watch的实现

```js
class Watcher () {
    constructor (vm, expOrFn, cb, options) {
        this.cb = cb;
        this.vm = vm;

        /*在这里将观察者本身赋值给全局的target，只有被target标记过的才会进行依赖收集*/
        Dep.target = this;
        /*Github:https://github.com/answershuto*/
        /*触发渲染操作进行依赖收集*/
        this.cb.call(this.vm);
    }

    update () {
        this.cb.call(this.vm);
    }
}
```

下面给出加入依赖收集的响应式代码，部分重复的代码不在这里给出

```js
class Vue {
    constructor(options) {
        this._data = options.data;
        observer(this._data, options.render);
        let watcher = new Watcher(this, );
    }
}

function defineReactive (obj, key, val, cb) {
    ／*在闭包内存储一个Dep对象*／
    const dep = new Dep();

    Object.defineProperty(obj, key, {
        enumerable: true,
        configurable: true,
        get: ()=>{
            if (Dep.target) {
                /*Watcher对象存在全局的Dep.target中*/
                dep.addSub(Dep.target);
            }
        },
        set:newVal=> {
            /*只有之前addSub中的函数才会触发*/
            dep.notify();
        }
    })
}

Dep.target = null;
```

将观察者Watcher实例赋值给全局的Dep.target，然后触发render操作只有被Dep.target标记过的才会进行依赖收集�。有Dep.target的对象会将Watcher的实例push到subs中，在对象被修改出发setter操作的时候dep会调用subs中的Watcher实例的update方法进行渲染