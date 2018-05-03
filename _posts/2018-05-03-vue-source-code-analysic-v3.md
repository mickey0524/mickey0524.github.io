---
layout:     post
title:      "vue-source-code-analysic（3）"
subtitle:   "阅读Vue源码有感（3）"
date:       2018-05-03 18:00:00
author:     "Mickey"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
tags:
    - 前端开发
    - Vue
---

前一篇博客从源码的角度讲述了vue作为一个MVVM框架，data是如何和视图进行绑定的，这篇博客将讲述vue2中的4个事件API

vue2和vue1有很多的不同，总的来说，框架变得更加严格，鲁棒性较好，其中非常重要的一个改变就是移除了$broadcast和$dispatch两个方法，优化了性能

vue2中提供了4个事件API挂在vm上，分别是$emit, $on, $off, $once

* $emit: 触发一个自定义事件，本组件和父亲组件能够监听
* $on: 用于在vm实例上监听一个自定义事件，该事件可以被$emit触发
* $off: 移除vm实例上监听的事件或者事件的回调函数
* $once: 用于在vm实例上监听一次自定义事件，事件触发之后，移除该事件

[vue事件处理源码地址](https://github.com/vuejs/vue/blob/dev/src/core/instance/events.js)

## 初始化事件

vm实例初始化的时候，在vm上绑定了一个_events对象，用于存放自定义对象，对象的key值为自定义事件名，value为一个数组，存放了该事件所有的回调函数

```js
vm._events = {
  eventName: [cb1, cb2, cb3],
};

/**
 * 初始化事件
 */
export function initEvents (vm: Component) {
  /**
   * 在vm上创建一个_events对象，用来存放事件。
   */
  vm._events = Object.create(null)
  /**
   * 这个bool标志位来表明是否存在钩子，而不需要通过哈希表的方法来查找是否有钩子
   * 这样做可以减少不必要的开销，优化性能
   */
  vm._hasHookEvent = false
  // init parent attached events
  /**
   * 初始化父组件attach的事件
   */
  const listeners = vm.$options._parentListeners
  if (listeners) {
    updateComponentListeners(vm, listeners)
  }
}

let target: any

/**
 * once为true的时候，执行$once注册事件，否则调用$on注册事件
 */
function add (event, fn, once) {
  if (once) {
    target.$once(event, fn)
  } else {
    target.$on(event, fn)
  }
}

/**
 * 销毁事件
 */
function remove (event, fn) {
  target.$off(event, fn)
}

/**
 * 更新组件的监听事件
 */
export function updateComponentListeners (
  vm: Component,
  listeners: Object,
  oldListeners: ?Object
) {
  target = vm
  updateListeners(listeners, oldListeners || {}, add, remove, vm)
  target = undefined
}

/**
 * 更新监听事件（在这里就是将父组件的事件capture函数放入vm实例的callback数组里）
 */
export function updateListeners (
  on: Object,
  oldOn: Object,
  add: Function,
  remove: Function,
  vm: Component
) {
  let name, def, cur, old, event
  for (name in on) {
    def = cur = on[name]
    old = oldOn[name]
    event = normalizeEvent(name)
    /* istanbul ignore if */
    if (__WEEX__ && isPlainObject(def)) {
      cur = def.handler
      event.params = def.params
    }
    if (isUndef(cur)) {
      process.env.NODE_ENV !== 'production' && warn(
        `Invalid handler for event "${event.name}": got ` + String(cur),
        vm
      )
    } else if (isUndef(old)) {
      if (isUndef(cur.fns)) {
        cur = on[name] = createFnInvoker(cur)
      }
      add(event.name, cur, event.once, event.capture, event.passive, event.params)
    } else if (cur !== old) {
      old.fns = cur
      on[name] = old
    }
  }
  for (name in oldOn) {
    if (isUndef(on[name])) {
      event = normalizeEvent(name)
      remove(event.name, oldOn[name], event.capture)
    }
  }
}
```

## $on方法

$on方法用来在vm实例上监听一个自定义事件，该事件可用$emit触发

```js
Vue.prototype.$on = function (event: string | Array<string>, fn: Function): Component {
  const vm: Component = this
  /**
   * 如果传入的事件不是string而是一个数组的话，则调用自身，为数组的每一个成员绑上fn的cb函数 
   */
  if (Array.isArray(event)) {
    for (let i = 0, l = event.length; i < l; i++) {
      this.$on(event[i], fn)
    }
  } else {
    (vm._events[event] || (vm._events[event] = [])).push(fn)
    // optimize hook:event cost by using a boolean flag marked at registration
    // instead of a hash lookup
    /**
     * 这里在注册事件的时候标记bool值也就是个标志位来表明存在钩子，而不需要通过哈希表的方法
     * 来查找是否有钩子，这样做可以减少不必要的开销，优化性能
     */
    const hookRE = /^hook:/ // vue的生命周期的钩子事件都是hook:开头的
    if (hookRE.test(event)) {
      vm._hasHookEvent = true
    }
  }
  return vm
}
```

## $once方法

$once监听一个只能触发一次的事件，在触发以后会自动移除该事件

```js
Vue.prototype.$once = function (event: string, fn: Function): Component {
  const vm: Component = this
  function on () {
    /*在第一次执行的时候将该事件销毁*/
    vm.$off(event, on)
    /*执行注册的方法*/
    fn.apply(vm, arguments)
  }
  on.fn = fn
  vm.$on(event, on)
  return vm
}
```

## $off方法

$off方法用于移除自定义事件

```js
Vue.prototype.$off = function (event?: string | Array<string>, fn?: Function): Component {
  const vm: Component = this
  // all
  /*如果不传参数则注销所有事件*/
  if (!arguments.length) {
    vm._events = Object.create(null)
    return vm
  }
  // array of events
  /*如果event是数组则递归注销事件*/
  if (Array.isArray(event)) {
    for (let i = 0, l = event.length; i < l; i++) {
      this.$off(event[i], fn)
    }
    return vm
  }
  // specific event
  const cbs = vm._events[event]
  /*Github:https://github.com/answershuto*/
  /*本身不存在该事件则直接返回*/
  if (!cbs) {
    return vm
  }
  /*如果只传了event参数则注销该event方法下的所有方法*/
  if (arguments.length === 1) {
    vm._events[event] = null
    return vm
  }
  // specific handler
  /*遍历寻找对应方法并删除*/
  let cb
  let i = cbs.length
  while (i--) {
    cb = cbs[i]
    if (cb === fn || cb.fn === fn) {
      cbs.splice(i, 1)
      break
    }
  }
  return vm
}
```

## $emit方法

$emit用来触发指定的自定义事件

```js
export function toArray (list: any, start?: number): Array<any> {
  start = start || 0
  let i = list.length - start
  const ret: Array<any> = new Array(i)
  while (i--) {
    ret[i] = list[i + start]
  }
  return ret
}

Vue.prototype.$emit = function (event: string): Component {
  const vm: Component = this
  if (process.env.NODE_ENV !== 'production') {
    const lowerCaseEvent = event.toLowerCase()
    if (lowerCaseEvent !== event && vm._events[lowerCaseEvent]) {
      tip(
        `Event "${lowerCaseEvent}" is emitted in component ` +
        `${formatComponentName(vm)} but the handler is registered for "${event}". ` +
        `Note that HTML attributes are case-insensitive and you cannot use ` +
        `v-on to listen to camelCase events when using in-DOM templates. ` +
        `You should probably use "${hyphenate(event)}" instead of "${event}".`
      )
    }
  }
  let cbs = vm._events[event]
  if (cbs) {
    /*将类数组的对象转换成数组*/
    cbs = cbs.length > 1 ? toArray(cbs) : cbs
    const args = toArray(arguments, 1)
    /*遍历执行*/
    for (let i = 0, l = cbs.length; i < l; i++) {
      cbs[i].apply(vm, args)
    }
  }
  return vm
}
```