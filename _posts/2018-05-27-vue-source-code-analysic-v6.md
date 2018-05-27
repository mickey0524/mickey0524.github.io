---
layout:     post
title:      "vue-source-code-analysic（6）"
subtitle:   "阅读Vue源码有感（6）"
date:       2018-05-27 14:30:00
author:     "Mickey"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
tags:
    - 前端开发
    - Vue
---

在前一篇博客中，我们介绍了Vue中的异步更新机制以及相当重要的nextTick函数，这篇博客我们来了解一下core/instance这个文件夹所做的事情

### core/index.js

下面是`core/index.js`的代码，易得，`export default Vue`中的Vue是从instance中引入再挂载全局方法后暴露出去的

```js
import Vue from './instance/index'
import { initGlobalAPI } from './global-api/index'
import { isServerRendering } from 'core/util/env'

initGlobalAPI(Vue)

Object.defineProperty(Vue.prototype, '$isServer', {
  get: isServerRendering
})

Vue.version = '__VERSION__'

export default Vue
```

### instance/index.js

既然Vue是从instance/index.js引入的，那么我们首先来看看这个文件的代码，这个文件定义了Vue类，然后依次调用了initMixin，stateMixin，eventsMixin，lifecycleMixin以及renderMixin五个方法给Vue的原型挂载了各种方法，下面我们依次看一下每个方法

```js
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  /*初始化*/
  this._init(options)
}

initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue
```

### initMixin

从`instance/index.js`中可以看到Vue实例化的时候仅仅调用了一个函数`_init`，initMixin中定义了这个方法

```js
export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    const vm: Component = this
    // a uid
    vm._uid = uid++

    let startTag, endTag
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      startTag = `vue-perf-init:${vm._uid}`
      endTag = `vue-perf-end:${vm._uid}`
      mark(startTag)
    }

    // a flag to avoid this being observed
    /*一个防止vm实例自身被观察的标志位*/
    vm._isVue = true
    // merge options
    if (options && options._isComponent) {
      // optimize internal component instantiation
      // since dynamic options merging is pretty slow, and none of the
      // internal component options needs special treatment.
      initInternalComponent(vm, options)
    } else {
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      )
    }
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      initProxy(vm)
    } else {
      vm._renderProxy = vm
    }
    // expose real self
    vm._self = vm
    // 初始化生命周期
    initLifecycle(vm)
    // 初始化事件
    initEvents(vm)
    // 初始化render
    initRender(vm)
    // 调用beforeCreate钩子函数并且触发beforeCreate钩子事件
    callHook(vm, 'beforeCreate')
    // 初始化inject，沿着组件链跨context获取变量
    initInjections(vm) // resolve injections before data/props
    // 初始化props、methods、data、computed与watch
    initState(vm)
    // 初始化provide
    initProvide(vm) // resolve provide after data/props
    // 调用created钩子函数并且触发created钩子事件
    callHook(vm, 'created')

    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      /*格式化组件名*/
      vm._name = formatComponentName(vm, false)
      mark(endTag)
      measure(`${vm._name} init`, startTag, endTag)
    }

    if (vm.$options.el) {
      /*挂载组件*/
      vm.$mount(vm.$options.el)
    }
  }
}
```

### stateMixin

下面来看看stateMixin函数做的事情，定义了vm.$set和vm.$delete两个方法，其实这两个方法和Vue.set以及Vue.delete的全局方法是一样的，此外，Vue中提供的vm.$watch的方法也是在这里定义的

```js
export function stateMixin (Vue: Class<Component>) {
  // flow somehow has problems with directly declared definition object
  // when using Object.defineProperty, so we have to procedurally build up
  // the object here.
  const dataDef = {}
  dataDef.get = function () { return this._data }
  const propsDef = {}
  propsDef.get = function () { return this._props }
  if (process.env.NODE_ENV !== 'production') {
    dataDef.set = function (newData: Object) {
      warn(
        'Avoid replacing instance root $data. ' +
        'Use nested data properties instead.',
        this
      )
    }
    propsDef.set = function () {
      warn(`$props is readonly.`, this)
    }
  }
  Object.defineProperty(Vue.prototype, '$data', dataDef)
  Object.defineProperty(Vue.prototype, '$props', propsDef)

  /**
   * https://cn.vuejs.org/v2/api/#vm-set
   * 用以将data之外的对象绑定成响应式的
   */
  Vue.prototype.$set = set
  /**
   * https://cn.vuejs.org/v2/api/#vm-delete
   * 与set对立，解除绑定
   */
  Vue.prototype.$delete = del

  /**
   * https://cn.vuejs.org/v2/api/#vm-watch
   * $watch方法
   * 用以为对象建立观察者监视变化
   */
  Vue.prototype.$watch = function (
    expOrFn: string | Function,
    cb: Function,
    options?: Object
  ): Function {
    const vm: Component = this
    options = options || {}
    options.user = true
    const watcher = new Watcher(vm, expOrFn, cb, options)
    // 有immediate参数的时候会立即执行
    if (options.immediate) {
      cb.call(vm, watcher.value)
    }
    // 返回一个取消观察函数，用来停止触发回调
    return function unwatchFn () {
      // 将自身从所有依赖收集订阅列表删除
      watcher.teardown()
    }
  }
}
```

### eventsMixin

在本系列第三篇博客中，我们详细介绍了$on, $once, $off以及$emit四个Vue中操作事件相关的API，这里就不多重复，详细可以查看[vue源码分析(3)](https://mickey0524.github.io/2018/05/03/vue-source-code-analysic-v3/)

```js
/*为Vue原型加入操作事件的方法*/
export function eventsMixin (Vue: Class<Component>) {
  const hookRE = /^hook:/

  /*在vm实例上绑定事件方法*/
  Vue.prototype.$on = function (event: string | Array<string>, fn: Function): Component {
    const vm: Component = this

    /*如果是数组的时候，则递归$on，为每一个成员都绑定上方法*/
    if (Array.isArray(event)) {
      for (let i = 0, l = event.length; i < l; i++) {
        this.$on(event[i], fn)
      }
    } else {
      (vm._events[event] || (vm._events[event] = [])).push(fn)
      // optimize hook:event cost by using a boolean flag marked at registration
      // instead of a hash lookup
      /*这里在注册事件的时候标记bool值也就是个标志位来表明存在钩子，而不需要通过哈希表的方法来查找是否有钩子，这样做可以减少不必要的开销，优化性能。*/
      if (hookRE.test(event)) {
        vm._hasHookEvent = true
      }
    }
    return vm
  }

  /*注册一个只执行一次的事件方法*/
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

  /*注销一个事件，如果不传参则注销所有事件，如果只传event名则注销该event下的所有方法*/
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

  /*触发一个事件方法*/
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
}
```

### lifecycleMixin

lifecycleMixin中给Vue的原型挂载了_update，$destory以及$forceUpdate三个方法，其中vm._update是一个内部方法，用于数据更改的时候调用vm.\_\_patch\_\_方法更新视图，vm.$forceUpdate调用vm._watcher的update方法，可以在数据修改而视图未更新的时候强制更新视图(注意这种方法不推荐使用)，vm.$destory方法销毁自身实例，包括清除监听者，从父亲组件的$children数组中移除，同时将vm对应的vnode赋为null，这里感觉可以将vm赋为null，提速垃圾回收

```js
export function lifecycleMixin (Vue: Class<Component>) {
  /*更新节点*/
  Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
    const vm: Component = this
    /*如果已经该组件已经挂载过了则代表进入这个步骤是个更新的过程，触发beforeUpdate钩子*/
    if (vm._isMounted) {
      callHook(vm, 'beforeUpdate')
    }
    const prevEl = vm.$el
    const prevVnode = vm._vnode
    const prevActiveInstance = activeInstance
    activeInstance = vm
    vm._vnode = vnode
    // Vue.prototype.__patch__ is injected in entry points
    // based on the rendering backend used.
    /*基于后端渲染Vue.prototype.__patch__被用来作为一个入口*/
    if (!prevVnode) {
      // initial render
      vm.$el = vm.__patch__(
        vm.$el, vnode, hydrating, false /* removeOnly */,
        vm.$options._parentElm,
        vm.$options._refElm
      )
    } else {
      // updates
      vm.$el = vm.__patch__(prevVnode, vnode)
    }
    activeInstance = prevActiveInstance
    // update __vue__ reference
    /*更新新的实例对象的__vue__*/
    if (prevEl) {
      prevEl.__vue__ = null
    }
    if (vm.$el) {
      vm.$el.__vue__ = vm
    }
    // if parent is an HOC, update its $el as well
    if (vm.$vnode && vm.$parent && vm.$vnode === vm.$parent._vnode) {
      vm.$parent.$el = vm.$el
    }
    // updated hook is called by the scheduler to ensure that children are
    // updated in a parent's updated hook.
  }

  Vue.prototype.$forceUpdate = function () {
    const vm: Component = this
    if (vm._watcher) {
      vm._watcher.update()
    }
  }

  Vue.prototype.$destroy = function () {
    const vm: Component = this
    if (vm._isBeingDestroyed) {
      return
    }
    /* 调用beforeDestroy钩子 */
    callHook(vm, 'beforeDestroy')
    /* 标志位 */
    vm._isBeingDestroyed = true
    // remove self from parent
    const parent = vm.$parent
    if (parent && !parent._isBeingDestroyed && !vm.$options.abstract) {
      remove(parent.$children, vm)
    }
    // teardown watchers
    /* 该组件下的所有Watcher从其所在的Dep中释放 */
    if (vm._watcher) {
      vm._watcher.teardown()
    }
    let i = vm._watchers.length
    while (i--) {
      vm._watchers[i].teardown()
    }
    // remove reference from data ob
    // frozen object may not have observer.
    if (vm._data.__ob__) {
      vm._data.__ob__.vmCount--
    }
    // call the last hook...
    vm._isDestroyed = true
    // invoke destroy hooks on current rendered tree
    vm.__patch__(vm._vnode, null)
    // fire destroyed hook
    /* 调用destroyed钩子 */
    callHook(vm, 'destroyed')
    // turn off all instance listeners.
    /* 移除所有事件监听 */
    vm.$off()
    // remove __vue__ reference
    if (vm.$el) {
      vm.$el.__vue__ = null
    }
    // remove reference to DOM nodes (prevents leak)
    vm.$options._parentElm = vm.$options._refElm = null
  }
}
```

### renderMixin

renderMixin函数定义了vm.$nextTick，和delete和set一样，这里vm.$nextTick和Vue.nextTick也是一样的用法，vm._render同样是个内部函数，用于将options渲染成一个vnode节点，剩下的方法是render内部使用的函数，之所以挂载在原型上，是为了减少render的大小

```js
export function renderMixin (Vue: Class<Component>) {
  Vue.prototype.$nextTick = function (fn: Function) {
    return nextTick(fn, this)
  }

  /*_render渲染函数，返回一个VNode节点*/
  Vue.prototype._render = function (): VNode {
    const vm: Component = this
    const {
      render,
      staticRenderFns,
      _parentVnode
    } = vm.$options

    if (vm._isMounted) {
      // clone slot nodes on re-renders
      /*在重新渲染时会克隆槽位节点 不知道是不是因为Vnode必须必须唯一的原因，网上也没找到答案，此处存疑。*/
      for (const key in vm.$slots) {
        vm.$slots[key] = cloneVNodes(vm.$slots[key])
      }
    }

    /*作用域slot*/
    vm.$scopedSlots = (_parentVnode && _parentVnode.data.scopedSlots) || emptyObject

    if (staticRenderFns && !vm._staticTrees) {
      /*用来存放static节点，已经被渲染的并且不存在v-for中的static节点不需要重新渲染，只需要进行浅拷贝*/
      vm._staticTrees = []
    }
    // set parent vnode. this allows render functions to have access
    // to the data on the placeholder node.
    vm.$vnode = _parentVnode
    // render self
    /*渲染*/
    let vnode
    try {
      /*调用render函数，返回一个VNode节点*/
      vnode = render.call(vm._renderProxy, vm.$createElement)
    } catch (e) {
      handleError(e, vm, `render function`)
      // return error render result,
      // or previous vnode to prevent render error causing blank component
      /* istanbul ignore else */
      if (process.env.NODE_ENV !== 'production') {
        vnode = vm.$options.renderError
          ? vm.$options.renderError.call(vm._renderProxy, vm.$createElement, e)
          : vm._vnode
      } else {
        vnode = vm._vnode
      }
    }
    // return empty vnode in case the render function errored out
    /*如果VNode节点没有创建成功则创建一个空节点*/
    if (!(vnode instanceof VNode)) {
      if (process.env.NODE_ENV !== 'production' && Array.isArray(vnode)) {
        warn(
          'Multiple root nodes returned from render function. Render function ' +
          'should return a single root node.',
          vm
        )
      }
      vnode = createEmptyVNode()
    }
    // set parent
    vnode.parent = _parentVnode
    return vnode
  }

  // internal render helpers.
  // these are exposed on the instance prototype to reduce generated render
  // code size.
  /*
    内部处理render的函数
    这些函数会暴露在Vue原型上以减小渲染函数大小
  */
  /*处理v-once的渲染函数*/
  Vue.prototype._o = markOnce
  /*将字符串转化为数字，如果转换失败会返回原字符串*/
  Vue.prototype._n = toNumber
  /*将val转化成字符串*/
  Vue.prototype._s = toString
  /*处理v-for列表渲染*/
  Vue.prototype._l = renderList
  /*处理slot的渲染*/
  Vue.prototype._t = renderSlot
  /*检测两个变量是否相等*/
  Vue.prototype._q = looseEqual
  /*检测arr数组中是否包含与val变量相等的项*/
  Vue.prototype._i = looseIndexOf
  /*处理static树的渲染*/
  Vue.prototype._m = renderStatic
  /*处理filters*/
  Vue.prototype._f = resolveFilter
  /*从config配置中检查eventKeyCode是否存在*/
  Vue.prototype._k = checkKeyCodes
  /*合并v-bind指令到VNode中*/
  Vue.prototype._b = bindObjectProps
  /*创建一个文本节点*/
  Vue.prototype._v = createTextVNode
  /*创建一个空VNode节点*/
  Vue.prototype._e = createEmptyVNode
  /*处理ScopedSlots*/
  Vue.prototype._u = resolveScopedSlots
}
```