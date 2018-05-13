---
layout:     post
title:      "vue-source-code-analysic（4）"
subtitle:   "阅读Vue源码有感（4）"
date:       2018-05-13 21:30:00
author:     "Mickey"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
tags:
    - 前端开发
    - Vue
---

前一篇博客介绍了Vue中的自定义事件，包括$on，$emit，$off以及$once，在这篇博客中，我们来介绍一下core/index.js中执行的initGlobalAPI函数，看看这个函数对Vue执行了什么样的操作，流程图如下所示

![initGlobalAPI函数执行流程图](/img/in-post/vue-source-code-analysic-v4/initGlobalAPI.png)

## initGlobalAPI函数

initGlobalAPI首先调用`Object.defineProperty(Vue, 'config', configDef)`将config挂载到Vue上，然后定义了Vue.set、Vue.delete以及Vue.nextTick等常用的全局函数，定义了Vue.components、Vue.directives以及Vue.filters三个空对象用于存放全局定义的组件、指令和过滤器，最后依次执行了initUse，initMixin，initExtend以及initAssetRegisters函数

```js
function initGlobalAPI (Vue: GlobalAPI) {
  // config
  const configDef = {}
  configDef.get = () => config
  if (process.env.NODE_ENV !== 'production') {
    configDef.set = () => {
      warn(
        'Do not replace the Vue.config object, set individual fields instead.'
      )
    }
  }
  // 将默认的config挂载到Vue上，在代码中可以通过Vue.config.param修改默认配置
  Object.defineProperty(Vue, 'config', configDef)

  // exposed util methods.
  // NOTE: these are not considered part of the public API - avoid relying on
  // them unless you are aware of the risk.
  Vue.util = {
    warn,
    extend,
    mergeOptions,
    defineReactive
  }

  Vue.set = set
  Vue.delete = del
  Vue.nextTick = nextTick

  Vue.options = Object.create(null)
  ASSET_TYPES.forEach(type => {
    Vue.options[type + 's'] = Object.create(null)
  })

  // this is used to identify the "base" constructor to extend all plain-object
  // components with in Weex's multi-instance scenarios.
  /*_base被用来标识基本构造函数（也就是Vue），以便在多场景下添加组件扩展*/
  Vue.options._base = Vue

  extend(Vue.options.components, builtInComponents)

  initUse(Vue) // 定义Vue.use()函数
  initMixin(Vue) // 定义Vue.mixin()函数
  initExtend(Vue) // 定义Vue.extend()构造器函数
  initAssetRegisters(Vue) // 定义Vue.component()，Vue.directive()以及Vue.filter()构造函数
}
```

## Vue.set函数

Vue.set()是非常常用的函数，用于修改数组和对象，强制触发UI更新，这个方法主要用于避开 Vue 不能检测属性被添加的限制

```js
function set (target: Array<any> | Object, key: any, val: any): any {
  /*如果传入数组则在指定位置插入val*/
  if (Array.isArray(target) && typeof key === 'number') {
    target.length = Math.max(target.length, key)
    target.splice(key, 1, val)
    /*因为数组不需要进行响应式处理，数组会修改七个Array原型上的方法来进行响应式处理*/
    return val
  }
  /*如果是一个对象，并且已经存在了这个key则直接返回*/
  if (hasOwn(target, key)) {
    target[key] = val
    return val
  }
  /*获得target的Oberver实例*/
  const ob = (target : any).__ob__;;
  /*
    _isVue 一个防止vm实例自身被观察的标志位 ，_isVue为true则代表vm实例，也就是this
    vmCount判断是否为根节点，存在则代表是data的根节点，Vue 不允许在已经创建的实例上动态添加新的根级响应式属性(root-level reactive property)
  */
  if (target._isVue || (ob && ob.vmCount)) {
    /*  
      Vue 不允许在已经创建的实例上动态添加新的根级响应式属性(root-level reactive property)。
      https://cn.vuejs.org/v2/guide/reactivity.html#变化检测问题
    */
    process.env.NODE_ENV !== 'production' && warn(
      'Avoid adding reactive properties to a Vue instance or its root $data ' +
      'at runtime - declare it upfront in the data option.'
    )
    return val;
  }
  if (!ob) {
    target[key] = val;
    return val;
  }
  /*为对象defineProperty上在变化时通知的属性*/
  defineReactive(ob.value, key, val);
  ob.dep.notify();
  return val;
}
```

## Vue.delete

用于删除数组或者对象元素，这个方法主要用于避开 Vue 不能检测到属性被删除的限制，应该很少会使用它

```js
function del (target: Array<any> | Object, key: any) {
  if (Array.isArray(target) && typeof key === 'number') {
    target.splice(key, 1) // 不需要执行notify函数，理由同上
    return
  }
  const ob = (target : any).__ob__
  if (target._isVue || (ob && ob.vmCount)) {
    process.env.NODE_ENV !== 'production' && warn(
      'Avoid deleting properties on a Vue instance or its root $data ' +
      '- just set it to null.'
    )
    return
  }
  if (!hasOwn(target, key)) {
    return
  }
  delete target[key]
  if (!ob) {
    return
  }
  ob.dep.notify()
}
```

## Vue.nextTick

延迟一个任务使其异步执行，在下一个tick时执行，一个立即执行函数，返回一个function，这个函数的作用是在task或者microtask(/jobs/微任务)中推入一个timerFunc，在当前调用栈执行完以后以此执行直到执行到timerFunc，目的是延迟到当前调用栈执行完以后执行

```js
nextTick = (function () {
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

## initUse函数

在Vue应用中，`Vue.use`函数随处可见，用于加载第三方的Vue扩展插件，插件需要提供一个install函数，否则Vue会把plugin当作install函数执行

```js
function initUse (Vue: GlobalAPI) {
  /*https://cn.vuejs.org/v2/api/#Vue-use*/
  Vue.use = function (plugin: Function | Object) {
    /* istanbul ignore if */
    /*标识位检测该插件是否已经被安装*/
    if (plugin.installed) {
      return
    }
    // additional parameters
    const args = toArray(arguments, 1)
    /*a*/
    args.unshift(this) // install函数需要传入一个Vue作为参数，因此这里unshift(this)
    if (typeof plugin.install === 'function') {
      /*install执行插件安装*/
      plugin.install.apply(plugin, args)
    } else if (typeof plugin === 'function') {
      plugin.apply(null, args)
    }
    plugin.installed = true
    return this
  }
}
```

## initMixin函数

在Vue中，可以通过initMixin函数加载抽象出来的Vue公共处理代码(和Vue的options参数相同)，然后通过mergeOption函数挂到Vue的options上，因为是全局作用的，因此需要小心，官方建议的如下

```js
// 为自定义的选项 'myOption' 注入一个处理器。
Vue.mixin({
  created: function () {
    var myOption = this.$options.myOption
    if (myOption) {
      console.log(myOption)
    }
  }
})

new Vue({
  myOption: 'hello!'
})
// => "hello!"
```

下面是initMixin函数的代码

```js
function initMixin (Vue: GlobalAPI) {
    /*https://cn.vuejs.org/v2/api/#Vue-mixin*/
  Vue.mixin = function (mixin: Object) {
    /*mergeOptions合并optiuons*/
    this.options = mergeOptions(this.options, mixin)
  }
}
```

## initExtend函数

我们可以通过Vue.extend()函数得到一个Vue的组件构造器，通过new这个构造器可以得到Vue实例，在生产模式中也非常常见

```js
function initExtend (Vue: GlobalAPI) {
  /**
   * Each instance constructor, including Vue, has a unique
   * cid. This enables us to create wrapped "child
   * constructors" for prototypal inheritance and cache them.
   */
  /*
    每个构造函数实例（包括Vue本身）都会有一个唯一的cid
    它为我们能够创造继承创建自构造函数并进行缓存创造了可能
  */
  Vue.cid = 0
  let cid = 1

  /**
   * Class inheritance
   */
   /*
   使用基础 Vue 构造器，创建一个“子类”。
   其实就是扩展了基础构造器，形成了一个可复用的有指定选项功能的子构造器。
   参数是一个包含组件option的对象。  https://cn.vuejs.org/v2/api/#Vue-extend-options
   */
  Vue.extend = function (extendOptions: Object): Function {
    extendOptions = extendOptions || {}
    /*父类的构造*/
    const Super = this
    /*父类的cid*/
    const SuperId = Super.cid
    const cachedCtors = extendOptions._Ctor || (extendOptions._Ctor = {})
    /*如果构造函数中已经存在了该cid，则代表已经extend过了，直接返回*/
    if (cachedCtors[SuperId]) {
      return cachedCtors[SuperId]
    }

    const name = extendOptions.name || Super.options.name
    if (process.env.NODE_ENV !== 'production') {
      /*name只能包含字母与连字符*/
      if (!/^[a-zA-Z][\w-]*$/.test(name)) {
        warn(
          'Invalid component name: "' + name + '". Component names ' +
          'can only contain alphanumeric characters and the hyphen, ' +
          'and must start with a letter.'
        )
      }
    }

    /*
      Sub构造函数其实就一个_init方法，这跟Vue的构造方法是一致的，在_init中处理各种数据初始化、生命周期等。
      因为Sub作为一个Vue的扩展构造器，所以基础的功能还是需要保持一致，跟Vue构造器一样在构造函数中初始化_init。
    */
    const Sub = function VueComponent (options) {
      this._init(options)
    }
    /*继承父类*/
    Sub.prototype = Object.create(Super.prototype)
    /*构造函数*/
    Sub.prototype.constructor = Sub
    /*创建一个新的cid*/
    Sub.cid = cid++
    /*将父组件的option与子组件的合并到一起(Vue有一个cid为0的基类，即Vue本身，会将一些默认初始化的option何入)*/
    Sub.options = mergeOptions(
      Super.options,
      extendOptions
    )
    /*es6语法，super为父类构造*/
    Sub['super'] = Super

    // For props and computed properties, we define the proxy getters on
    // the Vue instances at extension time, on the extended prototype. This
    // avoids Object.defineProperty calls for each instance created.
    /*在扩展时，我们将计算属性以及props通过代理绑定在Vue实例上（也就是vm），这也避免了Object.defineProperty被每一个实例调用*/
    if (Sub.options.props) {
      /*初始化props，将option中的_props代理到vm上*/
      initProps(Sub)
    }
    if (Sub.options.computed) {
      /*处理计算属性，给计算属性设置defineProperty并绑定在vm上*/
      initComputed(Sub)
    }

    // allow further extension/mixin/plugin usage
    /*加入extend、mixin以及use方法，允许将来继续为该组件提供扩展、混合或者插件*/
    Sub.extend = Super.extend
    Sub.mixin = Super.mixin
    Sub.use = Super.use

    // create asset registers, so extended classes
    // can have their private assets too.
    /*使得Sub也会拥有父类的私有选项（directives、filters、components）*/
    ASSET_TYPES.forEach(function (type) {
      Sub[type] = Super[type]
    })
    // enable recursive self-lookup
    /*把组件自身也加入components中，为递归自身提供可能（递归组件也会查找components是否存在当前组件，也就是自身）*/
    if (name) {
      Sub.options.components[name] = Sub
    }

    // keep a reference to the super options at extension time.
    // later at instantiation we can check if Super's options have
    // been updated.
    /*保存一个父类的options，此后我们可以用来检测父类的options是否已经被更新*/
    Sub.superOptions = Super.options，
    /*extendOptions存储起来*/
    Sub.extendOptions = extendOptions
    /*保存一份option，extend的作用是将Sub.options中的所有属性放入{}中*/
    Sub.sealedOptions = extend({}, Sub.options)

    // cache constructor
    /*缓存构造函数（用cid），防止重复extend*/
    cachedCtors[SuperId] = Sub
    return Sub
  }
}
```

## initAssetRegisters函数

在Vue应用中，全局的filter和directive比比皆是，组册方法如下所示

```js
function initAssetRegisters (Vue: GlobalAPI) {
  /**
   * Create asset registration methods.
   */
  ASSET_TYPES.forEach(type => {
    Vue[type] = function (
      id: string,
      definition: Function | Object
    ): Function | Object | void {
      if (!definition) {
        return this.options[type + 's'][id]
      } else {
        /* istanbul ignore if */
        if (process.env.NODE_ENV !== 'production') {
          if (type === 'component' && config.isReservedTag(id)) {
            warn(
              'Do not use built-in or reserved HTML elements as component ' +
              'id: ' + id
            )
          }
        }
        if (type === 'component' && isPlainObject(definition)) {
          definition.name = definition.name || id
          definition = this.options._base.extend(definition)
        }
        if (type === 'directive' && typeof definition === 'function') {
          definition = { bind: definition, update: definition }
        }
        this.options[type + 's'][id] = definition
        return definition
      }
    }
  })
}
```
