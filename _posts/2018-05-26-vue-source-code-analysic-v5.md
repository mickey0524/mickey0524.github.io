---
layout:     post
title:      "vue-source-code-analysic（5）"
subtitle:   "阅读Vue源码有感（5）"
date:       2018-05-26 21:30:00
author:     "Mickey"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
tags:
    - 前端开发
    - Vue
---

在上一篇博客中，我们介绍了Vue中initGlobalAPI这个函数，这个函数主要给Vue挂载了一些全局方法，在这篇博客中，我们来介绍一下Vue中异步更新操作以及非常重要的nextTick函数

### Watcher中的update方法

在之前的博客中，我们有介绍，在修改data中的数据时，会触发对应Dep对象的notity方法，Dep对象中有一个subs属性用来存放所有的Watcher实例，notity方法会依次触发所有Watcher实例的update方法，👇我们重温一下update方法的源码

```js
/**
  * Subscriber interface.
  * Will be called when a dependency changes.
  */
update () {
  /* istanbul ignore else */
  if (this.lazy) {
    this.dirty = true
  } else if (this.sync) {
    /*同步则执行run直接渲染视图*/
    this.run()
  } else {
    /*异步推送到观察者队列中，下一个tick时调用。*/
    queueWatcher(this)
  }
}
```

可以看到只有在this.sync为true的时候，Watcher才会立即执行run方法，触发Watcher实例的回调函数，默认是调用queueWatcher方法，queueWatcher方法是定义在observer/scheduler.js中的，下面我们看一下这个函数的源码

### queueWatcher

```js
/**
 * Push a watcher into the watcher queue.
 * Jobs with duplicate IDs will be skipped unless it's
 * pushed when the queue is being flushed.
 */
 /*将一个观察者对象push进观察者队列，在队列中已经存在相同的id则该观察者对象将被跳过，除非它是在队列被刷新时推送*/
export function queueWatcher (watcher: Watcher) {
  /*获取watcher的id*/
  const id = watcher.id
  /*检验id是否存在，已经存在则直接跳过，不存在则标记哈希表has，用于下次检验*/
  if (has[id] == null) {
    has[id] = true
    if (!flushing) {
      /*如果没有flush掉，直接push到队列中即可*/
      queue.push(watcher)
    } else {
      // if already flushing, splice the watcher based on its id
      // if already past its id, it will be run next immediately.
      let i = queue.length - 1
      while (i >= 0 && queue[i].id > watcher.id) {
        i--
      }
      queue.splice(Math.max(i, index) + 1, 0, watcher)
    }
    // queue the flush
    if (!waiting) {
      waiting = true
      nextTick(flushSchedulerQueue)
    }
  }
}
```

可以看到queueWatcher方法其实就是将watcher实例加入queue队列，如果当前不处于等待状态的话，执行nextTick(flushSchedulerQueue)方法，依次调用queue队列中所有watcher实例的run方法，下面我们来看一下nextTick函数

### nextTick

```js
/**
 * Defer a task to execute it asynchronously.
 */
 /*
    延迟一个任务使其异步执行，在下一个tick时执行，一个立即执行函数，返回一个function
    这个函数的作用是在task或者microtask中推入一个timerFunc，在当前调用栈执行完以后以此执行直到执行到timerFunc
    目的是延迟到当前调用栈执行完以后执行
*/
export const nextTick = (function () {
  /*存放异步执行的回调*/
  const callbacks = []
  /*一个标记位，如果已经有timerFunc被推送到任务队列中去则不需要重复推送*/
  let pending = false
  /*一个函数指针，指向函数将被推送到任务队列中，等到主线程任务执行完时，任务队列中的timerFunc被调用*/
  let timerFunc

  /*下一个tick时的回调*/
  function nextTickHandler () {
    /*一个标记位，标记等待状态（即函数已经被推入任务队列或者主线程，已经在等待当前栈执行完毕去执行），这样就不需要在push多个回调到callbacks时将timerFunc多次推入任务队列或者主线程*/
    pending = false
    /*执行所有callback*/
    const copies = callbacks.slice(0)
    callbacks.length = 0
    for (let i = 0; i < copies.length; i++) {
      copies[i]()
    }
  }

  // the nextTick behavior leverages the microtask queue, which can be accessed
  // via either native Promise.then or MutationObserver.
  // MutationObserver has wider support, however it is seriously bugged in
  // UIWebView in iOS >= 9.3.3 when triggered in touch event handlers. It
  // completely stops working after triggering a few times... so, if native
  // Promise is available, we will use it:
  /* istanbul ignore if */

  /*
    这里解释一下，一共有Promise、MutationObserver以及setTimeout三种尝试得到timerFunc的方法。
    优先使用Promise，在Promise不存在的情况下使用MutationObserver，这两个方法的回调函数都会在microtask中执行，它们会比setTimeout更早执行，所以优先使用。
    如果上述两种方法都不支持的环境则会使用setTimeout，在task尾部推入这个函数，等待调用执行。
    为啥要用 microtask？我在顾轶灵在知乎的回答中学习到：
    根据 HTML Standard，在每个 task 运行完以后，UI 都会重渲染，那么在 microtask 中就完成数据更新，
    当前 task 结束就可以得到最新的 UI 了。反之如果新建一个 task 来做数据更新，那么渲染就会进行两次。
    参考：https://www.zhihu.com/question/55364497/answer/144215284
  */
  if (typeof Promise !== 'undefined' && isNative(Promise)) {
    /*使用Promise*/
    var p = Promise.resolve()
    var logError = err => { console.error(err) }
    timerFunc = () => {
      p.then(nextTickHandler).catch(logError)
      // in problematic UIWebViews, Promise.then doesn't completely break, but
      // it can get stuck in a weird state where callbacks are pushed into the
      // microtask queue but the queue isn't being flushed, until the browser
      // needs to do some other work, e.g. handle a timer. Therefore we can
      // "force" the microtask queue to be flushed by adding an empty timer.
      if (isIOS) setTimeout(noop)
    }
  } else if (typeof MutationObserver !== 'undefined' && (
    isNative(MutationObserver) ||
    // PhantomJS and iOS 7.x
    MutationObserver.toString() === '[object MutationObserverConstructor]'
  )) {
    // use MutationObserver where native Promise is not available,
    // e.g. PhantomJS IE11, iOS7, Android 4.4
    /*新建一个textNode的DOM对象，用MutationObserver绑定该DOM并指定回调函数，在DOM变化的时候则会触发回调,该回调会进入主线程（比任务队列优先执行），即textNode.data = String(counter)时便会加入该回调*/
    var counter = 1
    var observer = new MutationObserver(nextTickHandler)
    var textNode = document.createTextNode(String(counter))
    observer.observe(textNode, {
      characterData: true //抽象节点依然观察改变
    })
    timerFunc = () => {
      counter = (counter + 1) % 2
      textNode.data = String(counter)
    }
  } else {
    // fallback to setTimeout
    /* istanbul ignore next */
    /*使用setTimeout将回调推入任务队列尾部*/
    timerFunc = () => {
      setTimeout(nextTickHandler, 0)
    }
  }

  /*
    推送到队列中下一个tick时执行
    cb 回调函数
    ctx 上下文
  */
  return function queueNextTick (cb?: Function, ctx?: Object) {
    let _resolve
    /*cb存到callbacks中*/
    callbacks.push(() => {
      if (cb) {
        try {
          cb.call(ctx)
        } catch (e) {
          handleError(e, ctx, 'nextTick')
        }
      } else if (_resolve) {
        _resolve(ctx)
      }
    })
    if (!pending) {
      pending = true
      timerFunc()
    }
    if (!cb && typeof Promise !== 'undefined') {
      return new Promise((resolve, reject) => {
        _resolve = resolve
      })
    }
  }
})()
```

这段代码就是Vue.nextTick以及this.$nextTick的原理，其实就是用一个callback数组存放需要执行的函数，然后在pending为false的时候，调用timerFunc函数依次执行cb数组中的全部函数，这段代码中出现了Promise，MutationObserver以及setTimeout三种控制函数，其中Promise和MutationObserver是jobs，而setTimeout是tasks，综合速度和复杂度，Promise 优于 MutationObserver 优于 setTimeout

### Vue为什么要使用异步更新机制

我们来看下面一段代码

```html
<template>
  <div class="container">
    <div>{{ num }}</div>
  </div>
</template>
```

```js
export default {
  data() {
    return {
      num: 0,
    }
  },
  mounted() {
    for (let i = 0; i < 1000; i++) {
      this.num += 1;
    }
  },
};
```

如果不采用异步更新机制的话，那么上面这段代码就会存在1000次`setter => dep.notity() => watcher.update() => watcher.run()`的过程，然后频繁的去修改DOM，这将造成极大的性能损失

另外，在Vue中，存在如下的代码检测是否存在死循环，在开发环境下，会提示开发者

```js
// in dev build, check and stop circular updates.
/**
 * 在测试环境中，检测watch是否在死循环中
 * 比如这样一种情况
 * watch: {
 *    num() {
 *      this.num++;
 *    }
 * }
 * 持续执行了一百次watch代表可能存在死循环
 */
if (process.env.NODE_ENV !== 'production' && has[id] != null) {
  circular[id] = (circular[id] || 0) + 1
  if (circular[id] > MAX_UPDATE_COUNT) {
    warn(
      'You may have an infinite update loop ' + (
        watcher.user
          ? `in watcher with expression "${watcher.expression}"`
          : `in a component render function.`
      ),
      watcher.vm
    )
    break
  }
}
```

### 如何获取对应节点的真实innerText

依旧是上面那个例子，将获取真实节点的代码放在`this.$nextTick`或者`Vue.nextTick`中，即能正常取数，因为queueWatcher中，同样也是调用
nextTick(flushSchedulerQueue)进行异步更新，那么我们放在this.$nextTick中的取数函数同样会进行nextTick内部的callback中，最后会依次执行，这就是原因

```html
<template>
  <div class="container">
    <div ref="num">{{ num }}</div>
    <button @click="onClickBtn">点我</button>
  </div>
</template>
```

```js
export default {
  data() {
    return {
      num: 0,
    }
  },
  methods() {
    onClickBtn() {
      this.num += 1;
      this.$nextTick(() => {
        console.log(this.$refs.num.innerText);
      });
    },
  },
};
```