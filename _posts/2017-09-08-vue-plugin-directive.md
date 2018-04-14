---
layout:     post
title:      "Vue-plugin-directive"
subtitle:   "Vue中的自定义插件和指令"
date:       2017-09-08 20:00:00
author:     "Mickey"
header-img: "img/post-bg-ios9-web.jpg"
tags:
    - 前端开发
    - Vue
---

博主最近在写Vue项目的时候，觉得自己对Vue的理解仅仅停留在使用的地步，远远达不到熟练掌握Vue的地步，于是，决定在之前的基础上重新学习一遍Vue，前不久写了一篇关于Vue组件的博客，而在这篇博客中，我将简要介绍一下Vue的一些扩展，比如自定义指令以及类似于Element UI的组件库如何开发～

主要讲以下三个知识点，至于最基础的也是最常见的使用v-model或者.sync显式控制组件在本篇博客中不会涉及，需要学习这部分知识的小伙伴可以移步[Vue官方文档](https://cn.vuejs.org/v2/guide/)，另外，本篇博客中的很多写法，灵感均来自于Element UI，不得不说，饿了么的前端团队确实有他们的独到之处，感谢🙏[Element ui](https://github.com/ElemeFE/element/)

### Vue元指令的开发

虽然Vue官方是推荐使用v-model或者.sync显式调用组件，但不得不说，在js中使用this.$message()的方式调用一个组件在某些时候十分的方便，在element ui中，也有类似的写法～

之前的做法都是写一个Vue文件，然后通过components属性引入页面，显式写入标签调用的。那么如何将组件通过js的方法去调用呢？

这里的关键是Vue的extend方法。

文档里并没有详细给出extend能这么用，只是作为需要手动mount的一个Vue的组件构造器说明了一下而已，这里给出具体的操作方式

首先依然是创建一个Notice.vue的文件

```java
<template>
  <div class="notice hide">
    <div class="content">
      {{ content }}
    </div>
  </div>
</template>

<script>
  export default {
    name: 'notice',
    data() {
      return {
        content: '',
        duration: 3000,
        transitionDuration: 1000,
      }
    },
    methods: {
      setTimer() {
        setTimeout(() => {
          this.close(); // 3000ms之后调用关闭方法
        }, this.duration);
      },
      close() {
        this.$el.classList.add('hide');
        this.$destroy(true); // 出发beforeDestory 和 destoryed 
        setTimeout(() => {
          this.$el.parentNode.removeChild(this.$el) // 从DOM里将这个组件移除
        }, this.transitionDuration);
      }
    },
    mounted() {
      this.setTimer(); // 挂载的时候就开始计时，3000ms后消失
    }
  }
</script>

<style>
  .notice {
    opacity: 1;
    background: #21252B;
    color: #FFF;
    position: fixed;
    width: 100px;
    line-height: 40px;
    text-align: center;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    border-radius: 4px;
    transition: all 1s ease-in-out;
  }
  .hide {
    opacity: 0;
  }
  .notice .content {
    display: inline-block;
  }
</style>
```

接下来，需要创建一个index.js文件，将该组件注册到Vue的prototype上去，如下所示

```js
import Vue from 'vue';
import NoticeComponent from './Notice.vue';

const noticeConstructor = Vue.extend(NoticeComponent);

const Notice = (content, duration) => {
  let noticeInstance = new noticeConstructor({
    data: {
      content: content,
      transitionDuration: duration,
    }
  });
  noticeInstance.vm = noticeInstance.$mount();
  noticeInstance.dom = noticeInstance.vm.$el;
  noticeInstance.dom.style.zIndex = '1000';
  noticeInstance.dom.style.transitionDuration = `${duration / 1000}s`;
  document.body.appendChild(noticeInstance.dom);
  setTimeout(() => {
    noticeInstance.dom.classList.remove('hide');
  }, 0);
  return noticeInstance.vm;
}

export default {
  install: Vue => {
    Vue.prototype.$notice = Notice;
  }
}
```

上述代码中，首先使用Vue.extend()生成了一个Notice组件的构建器，可以理解为一个class，然后实例化了一个noticeInstance，可以理解为一个Vue实例，通过手动$mount()方法挂载实例，这个时候，实例就拥有了自己的$el属性，也即要添加到DOM中的元素

最后，到main.js中，使用Vue提供的Vue.use()方法注册该插件，然后就能愉快的时候了，这里需要注意以下，Vue.use()接受一个install函数，用于将该Vue实例注册到原型链上去

```js
import Notice from './components/notice';
Vue.use(Notice);
```

### 自定义指令的开发

Vue中，自定义指令的开发主要涉及Vue.directive()这个方法，其实跟元指令的思路差不多，不过因为涉及到directive，所以在逻辑上会相对复杂一点。

平时如果不涉及Vue的directive的开发，可能是不会接触到modifiers、binding等概念, 这里可以参考Vue官方文档[Vue自定义指令](https://cn.vuejs.org/v2/guide/custom-directive.html)

简单说下，形如:v-loading.fullscreen="true"这句话，v-loading就是directive，fullscreen就是它的modifier，true就是binding的value值。

其实loading也是一个实际的DOM节点，只不过要把它做成一个方便的指令还不是特别容易。

首先我们需要写一下loading的Vue组件。新建一个Loading.vue文件

```java
<template>
    <div
      v-show="visible"
      :class="['loading-mask', fullscreen && 'fullscreen']">
      <div class="loading-text" v-if="text">
        {{ text }}
      </div>
    </div>
</template>

<script>
export default {
  name: 'loading',
  data () {
    return {
      visible: true,
      fullscreen: true,
      text: '等待中',
    }
  }
}
</script>

<style>
  .loading-mask {
    position: absolute; /*非全屏模式下，position是absolute*/
    z-index: 10000;
    background-color: rgba(255,235,215, .8);
    margin: 0;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    transition: opacity .3s;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .loading-mask.fullscreen {
    position: fixed;  /* 全屏模式下，position是fixed*/
  }
</style>
```
Loading关键是实现两个效果：

1. 全屏loading，此时可以通过插入body下，然后将Loading的position改为fixed，插入body实现。
2. 对所在的元素进行loading，此时需要对当前这个元素的的position修改：如果不是absolute的话，就将其修改为relatvie，并插入当前元素下。此时Loading的position就会相对于当前元素进行绝对定位了。

所以在当前目录下创建一个index.js的文件，用来声明我们的directive的逻辑。

```js
import Vue from 'vue';
import LoadingComponent from './Loading.vue';

const LoadingConstructor = Vue.extend(LoadingComponent);

export default {
  install: Vue => {
    Vue.directive('loading', { // 指令的关键
      bind: (el, binding) => {
        const loading = new LoadingConstructor({ // 实例化一个loading
          el: document.createElement('div'),
          data: {
            text: el.getAttribute('loading-text'), // 通过loading-text属性获取loading的文字
            fullscreen: !!binding.modifiers.fullscreen,
          }
        })
        el.instance = loading; // el.instance是个Vue实例
        el.loading = loading.$el; // el.loading的DOM元素是loading.$el，其实就是el.instance.$el
        toggleLoading(el, binding);
      },
      update: (el, binding) => {
        if (binding.oldValue !== binding.value) {
          toggleLoading(el, binding)
        }
      },
      unbind: (el, binding) => { // 解绑
        if (el.domInserted) {
          if (binding.modifiers.fullscreen) {
            document.body.removeChild(el.loading);
          }
          else {
            el.loading &&
            el.loading.parentNode &&
            el.loading.parentNode.removeChild(el.loading);
          }
        }
      }
    })
    const toggleLoading = (el, binding) => { // 用于控制Loading的出现与消失
      if (binding.value) {
        Vue.nextTick(() => {
          if (binding.modifiers.fullscreen) { // 如果是全屏
            el.originalPosition = document.body.style.position;
            el.originalOverflow = document.body.style.overflow;
            insertDom(document.body, el, binding); // 插入dom
          } else {
            el.originalPosition = el.style.position;
            insertDom(el, el, binding); // 如果非全屏，插入元素自身
          }
        });
      } else {
        if (el.domVisible) {
          el.domVisible = false;
          if (binding.modifiers.fullscreen && el.originalOverflow !== 'hidden') {
            document.body.style.overflow = el.originalOverflow;
          }
          if (binding.modifiers.fullscreen) {
            document.body.style.position = el.originalPosition;
          } else {
            el.style.position = el.originalPosition;
          }
          Vue.nextTick(() => {
            el.instance.visible = false;
          });
        }
      }
    }
    const insertDom = (parent, el, binding) => { // 插入dom的逻辑
      if (!el.domVisible) {
        if (el.originalPosition !== 'absolute') {
          parent.style.position = 'relative'
        }
        if (binding.modifiers.fullscreen) {
          parent.style.overflow = 'hidden';
        }
        el.domVisible = true;
        if (!el.domInserted) {
          parent.appendChild(el.loading) // 插入的是el.loading而不是el本身
          el.domInserted = true;
        }
        Vue.nextTick(() => {
          el.instance.visible = true;
        });
      }
    }
  }
}
```

同样，写完整个逻辑，我们需要将其注册到项目里的Vue下：

```js
// main.js
// ...
import Loading from 'loading/index.js'
Vue.use(Loading)
// ...
```
至此我们已经可以使用形如

`<div v-loading.fullscreen="loading" loading-text="正在加载中">`

这样的方式来实现调用一个loading组件了。

### 一个组件库的开发

回想一下，Vue创建一个组件的流程，首先是一个Vue组件的编写，编写完成使用Vue.extend()进行创建，然后，该组件通过Vue.component()的方法进行注册，然后就能在其他组件中使用了，一个组件库的编写同样是这个道理，让我们来一探究竟～

首先，编写两个简单到不能再简单的Vue组件，Hello.vue和World.vue，prefix只是一个前缀，就和element-ui中的el一样

```java
<template>
  <div class="hello">
    hello
  </div>
</template>

<script>
  import { prefix } from '../base.js';
  export default {
    name: `${prefix}hello`
  }
</script>
```

```java
<template>
  <div class="world">
    world
  </div>
</template>

<script>
  import { prefix } from '../base.js';
  export default {
    name: `${prefix}world`
  }
</script>
```

然后给Hello.vue和World.vue两个组件增加install方法

```js
import Hello from './Hello.vue';

Hello.install = function(Vue) {
  Vue.component(Hello.name, Hello);
}

export default Hello;
```

```js
import World from './World.vue';

World.install = function(Vue) {
  Vue.component(World.name, World);
}

export default World;
```

最后，用一个整体的组件库入口文件index.js将所有组件进行注册

```js
import Hello from './lib/components/hello/index.js';
import World from './lib/components/world/index.js';

var components = [
  Hello,
  World
];

function install(Vue) {
  components.map((component) => {
    component.install(Vue);
  })
}

export default {
  install
}

export {
  Hello,  //这是为了单独引用Hello组件，减少代码大小
  World
};
```

最后同样在main.js中使用Vue.use()注册，就可以愉快的使用了

```js
import MickeyUI from 'mickey-ui';

Vue.use(MickeyUI);  //整体注册

import {
  Hello,
  World
} from 'mickey-ui';

Vue.component(Hello.name, Hello);  //按需注册
Vue.component(World.name, World);
```