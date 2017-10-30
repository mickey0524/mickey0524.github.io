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
function observer(value, cb) {
    Object.keys(value).forEach((key) => defineReactive(value, key, value[key] , cb))
}

function defineReactive (obj, key, val, cb) {
    Object.defineProperty(obj, key, {
        enumerable: true,
        configurable: true,
        get: ()=>{
            /*....依赖收集等....*/
	         return obj[key];
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
    notify () { //当data中的数据执行setter方法的时候，调用该方法，通知watch订阅者重绘
        // stabilize the subscriber list first
        const subs = this.subs.slice()
        for (let i = 0, l = subs.length; i < l; i++) {
            subs[i].update()
        }
    }
}
```

可能有同学看到上图代码中的subs数据会产生疑问，其实这就是用来存放watch订阅者对象的(其实就是一个一个包含{{}}的节点)，因为一个data数据可能有多个watch，因此需要使用数组，下面给出watch的实现

```js
class Watcher () {
    constructor (vm, exp, cb) {
        this.cb = cb; //数据改变的时候的回调函数
        this.vm = vm; //vue实例
        this.exp = exp; //data中的key值
        this.value = this.get(); //当前的数值
    }

    update () {
        var value = this.vm.data[this.exp];
        if (value == this.value) {
            return ;
        }
        this.cb.call(this.vm, value, this.value);
        this.value = value;
    }
    
    get () {
        Dep.target = this; 结合observer看，将watcher对象绑定到Dep上
        var value = this.vm.data[this.exp]; 触发一次getter，等于触发依赖收集
        Dep.target = null;
        return value;
    }
}
```

看到这里，同学们可能会问，那什么时候该调用Watcher呢（何时注册观察者），我们知道，实例化Vue对象的时候，需要给出一个el的字段，同时，我们也知道，Vue的模版文件中有很多{{}}，那么，肯定需要一个Compile过程，来将{{}}包裹的字段替换为真实字符串，这里只给出最基本的代码

```js
class Compile {

	 constructor(el, vm) {
		  this.el = document.querySelector(el);
		  this.vm = vm;
		  this.fragment = null;
		  this.init();
	 }
	 
    init: function () {
        if (this.el) {
            this.fragment = this.nodeToFragment(this.el);
            this.compileElement(this.fragment);
            this.el.appendChild(this.fragment);//由于appendChild会删除原先DOM中存在的节点，千万要记得最后要把fragment回填到DOM中去
        } else {
            console.error('Dom元素不存在');
        }
    }
    
    nodeToFragment: function (el) {
        var fragment = document.createDocumentFragment();
        var child = el.firstChild;
        while (child) {
            // 将Dom元素移入fragment中
            fragment.appendChild(child);
            child = el.firstChild; //这里咋一看似乎问题，child似乎一直没有改变，其实不是这样，因为appendChild会删除原先DOM中存在的节点
        }
        return fragment;
    }
    
    compileElement: function (el) {
        var childNodes = el.childNodes;
        var self = this;
        [].slice.call(childNodes).forEach(function(node) {
            var reg = /\{\{(.*)\}\}/;
            var text = node.textContent;

			  if (self.isTextNode(node) && reg.test(text)) {
                self.compileText(node, reg.exec(text)[1]);
            }

            if (node.childNodes && node.childNodes.length) {
                self.compileElement(node);
            }
        });
    }
    
    compileText: function(node, exp) {
        var self = this;
        var initText = this.vm[exp];
        this.updateText(node, initText);
        new Watcher(this.vm, exp, function (value) {
            self.updateText(node, value);
        });
    }
    
    updateText: function (node, value) {
        node.textContent = typeof value == 'undefined' ? '' : value;
    }
    
    isTextNode: function(node) {
        return node.nodeType == 3;
    }
}
```

下面给出加入依赖收集的响应式代码，部分重复的代码不在这里给出

```js
class Vue {
    constructor(options) {
        var self = this;
        this.data = options.data;
        Object.keys(this.data).forEach((key) => {
            self.proxyKey(key);
        })
        observer(this.data);
        new Compile(options.el, this);
        options.mounted.call(this); // 所有事情处理好后执行mounted函数
    }
}
```

### 参考资料

[Vue的视图数据绑定实现](https://www.w3cplus.com/vue/vue-two-way-binding.html)