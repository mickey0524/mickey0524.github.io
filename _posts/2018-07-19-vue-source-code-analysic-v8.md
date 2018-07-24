---
layout:     post
title:      "vue-source-code-analysic（8）"
subtitle:   "阅读Vue源码有感（8）"
date:       2018-07-19 23:30:00
author:     "Mickey"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
tags:
    - 前端开发
    - Vue
---

上一篇博客中，我们介绍了在vue中非常重要VNode类，这是AST的基础，在这篇博客中，我们来介绍一下位于`vnode`目录下的`create-element.js`文件以及`create-component.js`两个文件，在这两个文件中，我们能够知道VNode节点是如何生成的

### create-element.js文件

在`render.js`中有如下的一段代码

```js
export function initRender() {
  vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false)
  vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true)
}
```

将`createElement`方法挂载到vm实例上，在`render`函数中相应的执行，下面我们就来看看该方法的源码

```js
export function createElement (
  context: Component,
  tag: any,
  data: any,
  children: any,
  normalizationType: any,
  alwaysNormalize: boolean
): VNode {
  /*兼容不传data的情况*/
  if (Array.isArray(data) || isPrimitive(data)) {
    normalizationType = children
    children = data
    data = undefined
  }
  /*如果alwaysNormalize为true，则normalizationType标记为ALWAYS_NORMALIZE*/
  if (isTrue(alwaysNormalize)) {
    normalizationType = ALWAYS_NORMALIZE
  }
  /*创建虚拟节点*/
  return _createElement(context, tag, data, children, normalizationType)
}
```

从上面的代码可以看到，`createElement`返回了一个`_createElement`函数，其实，核心的代码都是放在`_createElement`函数中，`createElement`函数做了一层兜底，兼容了不传`data`的情况(因为一个节点是`v-pre`的节点的子节点，那么`el.plain=true`，不会传`VNodeData`也就是`data`变量到`createElement`函数中)

下面看看核心的`_createElement`方法，简单来说，`_createElement`方法的作用是在不同的条件下渲染不同的VNode节点，例如：空节点，普通节点，组件节点等

```js
/*创建VNode节点*/
export function _createElement (
  context: Component,
  tag?: string | Class<Component> | Function | Object,
  data?: VNodeData,
  children?: any,
  normalizationType?: number
): VNode {
  /*
    如果data未定义（undefined或者null）或者是data的__ob__已经定义（代表已经被observed，上面绑定了Oberver对象），
    https://cn.vuejs.org/v2/guide/render-function.html#约束
    那么创建一个空节点
  */
  if (isDef(data) && isDef((data: any).__ob__)) {
    process.env.NODE_ENV !== 'production' && warn(
      `Avoid using observed data object as vnode data: ${JSON.stringify(data)}\n` +
      'Always create fresh vnode data objects in each render!',
      context
    )
    return createEmptyVNode()
  }
  /*如果tag不存在也是创建一个空节点*/
  if (!tag) {
    // in case of component :is set to falsy value
    return createEmptyVNode()
  }
  // support single function children as default scoped slot
  /*默认默认作用域插槽*/
  if (Array.isArray(children) &&
      typeof children[0] === 'function') {
    data = data || {}
    data.scopedSlots = { default: children[0] }
    children.length = 0
  }
  if (normalizationType === ALWAYS_NORMALIZE) {
    children = normalizeChildren(children)
  } else if (normalizationType === SIMPLE_NORMALIZE) {
    children = simpleNormalizeChildren(children)
  }
  let vnode, ns
  if (typeof tag === 'string') {
    let Ctor
    /*获取tag的名字空间*/
    ns = config.getTagNamespace(tag)
    /*判断是否是保留的标签*/
    if (config.isReservedTag(tag)) {
      // platform built-in elements
      /*如果是保留的标签则创建一个相应节点*/
      vnode = new VNode(
        config.parsePlatformTagName(tag), data, children,
        undefined, undefined, context
      )
    } else if (isDef(Ctor = resolveAsset(context.$options, 'components', tag))) {
      // component
      /*从vm实例的option的components中寻找该tag，存在则就是一个组件，创建相应节点，Ctor为组件的构造类*/
      vnode = createComponent(Ctor, data, context, children, tag)
    } else {
      // unknown or unlisted namespaced elements
      // check at runtime because it may get assigned a namespace when its
      // parent normalizes children
      /*未知的元素，在运行时检查，因为父组件可能在序列化子组件的时候分配一个名字空间*/
      vnode = new VNode(
        tag, data, children,
        undefined, undefined, context
      )
    }
  } else {
    // direct component options / constructor
    /*tag不是字符串的时候则是组件的构造类*/
    vnode = createComponent(tag, data, context, children)
  }
  if (isDef(vnode)) {
    /*如果有名字空间，则递归所有子节点应用该名字空间*/
    if (ns) applyNS(vnode, ns)
    return vnode
  } else {
    /*如果vnode没有成功创建则创建空节点*/
    return createEmptyVNode()
  }
}
```

让我们细细来分析一下，首先如果传入`_createElement`函数的data存在`__ob__`属性，说明该`data`是被`Observer`注册过的`$options.data`，这种情况下，Vue会创建一个空的`VNode`节点

如果tag参数不存在的话，同样创建一个空的`VNode`节点，毕竟不知道tag，Vue也不知道给你生成啥节点啊，臣妾做不到啊，虽然`template`编译不会出现这些问题，但是还是要杜绝手写render可能引发的问题

然后程序判断`tag`参数是否为`String`类型，如果不为`String`类型的话，Vue会认为tag是一个组件的构造器，那么会调用`create-component.js`内的createComponent函数生成一个组件节点；如果为`String`类型的话，如果tag为`reserveTag`也就是`div`之类的平台保留节点，Vue会生成一个普通的`VNode`节点，如果tag存在于`vm.$options.components`中的话，同样生成组件节点(这是Vue引入模块的最常见方法)，如果都不匹配的话，认为是未知的元素，在运行时检查，因为父组件可能在序列化子组件的时候分配一个名字空间

### create-component.js

在`create-element.js`中，我们可以看到当`tag`在components中注册过，或者`tag`是一个构造类的时候，调用了`createComponent`方法，下面我们就来看看定义了该方法的`create-component.js`文件

```js
/*创建一个组件节点，返回Vnode节点*/
export function createComponent (
  Ctor: Class<Component> | Function | Object | void,
  data?: VNodeData,
  context: Component,
  children: ?Array<VNode>,
  tag?: string
): VNode | void {
  /*没有传组件构造类直接返回*/
  if (isUndef(Ctor)) {
    return
  }

  /*_base存放了Vue,作为基类，可以在里面添加扩展*/
  const baseCtor = context.$options._base

  // plain options object: turn it into a constructor
  if (isObject(Ctor)) {
    Ctor = baseCtor.extend(Ctor)
  }

  // if at this stage it's not a constructor or an async component factory,
  // reject.
  /*如果在该阶段Ctor依然不是一个构造函数或者是一个异步组件工厂则直接返回*/
  if (typeof Ctor !== 'function') {
    if (process.env.NODE_ENV !== 'production') {
      warn(`Invalid Component definition: ${String(Ctor)}`, context)
    }
    return
  }

  // async component
  /*处理异步组件*/
  if (isUndef(Ctor.cid)) {
    Ctor = resolveAsyncComponent(Ctor, baseCtor, context)
    if (Ctor === undefined) {
      // return nothing if this is indeed an async component
      // wait for the callback to trigger parent update.
      /*如果这是一个异步组件则会不会返回任何东西（undifiened），直接return掉，等待回调函数去触发父组件更新。s*/
      return
    }
  }

  // resolve constructor options in case global mixins are applied after
  // component constructor creation
  resolveConstructorOptions(Ctor)

  data = data || {}

  // transform component v-model data into props & events
  if (isDef(data.model)) {
    transformModel(Ctor.options, data)
  }

  // extract props
  const propsData = extractPropsFromVNodeData(data, Ctor, tag)

  // functional component
  if (isTrue(Ctor.options.functional)) {
    return createFunctionalComponent(Ctor, propsData, data, context, children)
  }

  // extract listeners, since these needs to be treated as
  // child component listeners instead of DOM listeners
  const listeners = data.on
  // replace with listeners with .native modifier
  data.on = data.nativeOn

  if (isTrue(Ctor.options.abstract)) {
    // abstract components do not keep anything
    // other than props & listeners
    data = {}
  }

  // merge component management hooks onto the placeholder node
  mergeHooks(data)

  // return a placeholder vnode
  const name = Ctor.options.name || tag
  const vnode = new VNode(
    `vue-component-${Ctor.cid}${name ? `-${name}` : ''}`,
    data, undefined, undefined, undefined, context,
    { Ctor, propsData, listeners, tag, children }
  )
  return vnode
}
```

`createComponent`方法如上所示，首先，让我们回忆一下，`Vue`有一个全局的`Vue.extend`的方法，用于创造一个构造器，在`createComponent`中，`context.$options._base`取出的就是该`extend`方法.

如果传入的`Ctor`是一个函数，那么不会掉用`extend`方法，也就意味着不会得到`cid`，那么就会调用`resolveAsyncComponent `加载的就是一个异步组件，如果不了解Vue加载异步组件，可以先看看[官方文档](https://cn.vuejs.org/v2/guide/components-dynamic-async.html#%E5%BC%82%E6%AD%A5%E7%BB%84%E4%BB%B6)，下面代码中的注释写的比较详细，可以结合官方文档看看

```
function ensureCtor (comp, base) {
  return isObject(comp)
    ? base.extend(comp)
    : comp
}

export function resolveAsyncComponent (
  factory: Function,
  baseCtor: Class<Component>,
  context: Component
): Class<Component> | void {
  /*出错组件工厂返回出错组件*/
  if (isTrue(factory.error) && isDef(factory.errorComp)) {
    return factory.errorComp
  }

  /*resoved时候返回resolved组件*/
  if (isDef(factory.resolved)) {
    return factory.resolved
  }

  /*加载组件过程中，显示loading组件*/
  if (isTrue(factory.loading) && isDef(factory.loadingComp)) {
    return factory.loadingComp
  }

  if (isDef(factory.contexts)) {
    // already pending
    factory.contexts.push(context)
  } else {
    const contexts = factory.contexts = [context]
    let sync = true
	 
	 // 加载完成，异步更新context
    const forceRender = () => {
      for (let i = 0, l = contexts.length; i < l; i++) {
        contexts[i].$forceUpdate()
      }
    }

    const resolve = once((res: Object | Class<Component>) => {
      // cache resolved
      factory.resolved = ensureCtor(res, baseCtor)
      // invoke callbacks only if this is not a synchronous resolve
      // (async resolves are shimmed as synchronous during SSR)
      if (!sync) {
        forceRender()
      }
    })

    const reject = once(reason => {
      process.env.NODE_ENV !== 'production' && warn(
        `Failed to resolve async component: ${String(factory)}` +
        (reason ? `\nReason: ${reason}` : '')
      )
      if (isDef(factory.errorComp)) {
        factory.error = true
        forceRender()
      }
    })

    const res = factory(resolve, reject)

    if (isObject(res)) {
    	// () => Promise类型
      if (typeof res.then === 'function') {
        if (isUndef(factory.resolved)) {
          res.then(resolve, reject)
        }
      } else if (isDef(res.component) && typeof res.component.then === 'function') {
		  // const AsyncComponent = () => ({
			  // 需要加载的组件 (应该是一个 `Promise` 对象)
			  // component: import('./MyComponent.vue'),
			  // 异步组件加载时使用的组件
			  // loading: LoadingComponent,
			  // 加载失败时使用的组件
			  // error: ErrorComponent,
			  // 展示加载时组件的延时时间。默认值是 200 (毫秒)
			  // delay: 200,
			  // 如果提供了超时时间且组件加载也超时了，
			  // 则使用加载失败时使用的组件。默认值是：`Infinity`
			  // timeout: 3000
		  })
        res.component.then(resolve, reject)

        if (isDef(res.error)) {
          factory.errorComp = ensureCtor(res.error, baseCtor)
        }

        if (isDef(res.loading)) {
          factory.loadingComp = ensureCtor(res.loading, baseCtor)
          if (res.delay === 0) {
            factory.loading = true
          } else {
            setTimeout(() => {
              if (isUndef(factory.resolved) && isUndef(factory.error)) {
                factory.loading = true
                forceRender()
              }
            }, res.delay || 200)
          }
        }

        if (isDef(res.timeout)) {
          setTimeout(() => {
            reject(
              process.env.NODE_ENV !== 'production'
                ? `timeout (${res.timeout}ms)`
                : null
            )
          }, res.timeout)
        }
      }
    }

    sync = false
    // return in case resolved synchronously
    return factory.loading
      ? factory.loadingComp
      : factory.resolved
  }
}
```

当传入的`Ctor`是对象的时候，隐式的生成一个构造器，然后调用`resolveConstructorOptions`方法去获取最新的`options`，这个方法定义在`core/instance/init.js`中

随后，Vue调用了`transformModel`方法，用于给`v-model`绑定处理函数，其实`v-model`就是一个语法糖，通过监听`input`事件，来响应式修改`value`数值，方法如下

```js
function transformModel (options, data: any) {
  const prop = (options.model && options.model.prop) || 'value'
  const event = (options.model && options.model.event) || 'input'
  ;(data.props || (data.props = {}))[prop] = data.model.value
  const on = data.on || (data.on = {})
  if (isDef(on[event])) {
    on[event] = [data.model.callback].concat(on[event])
  } else {
    on[event] = data.model.callback
  }
}
```

紧接着，Vue调用了`extractPropsFromVNodeData`方法，这个方法是生成propsData对象的。我们知道，一个Vue组件，在props中可以定义组件接收的变量，而propsData则是父组件真实传入的变量，剩余的变量，可以通过`default`定义

当`Ctor.options.functional`为true的时候，Vue认为这是一个函数式组件，会调用`createFunctionalComponent`函数，至于什么是函数式组件，我们可以把函数式组件想像成组件里的一个函数，入参是渲染上下文(render context)，返回值是渲染好的HTML，函数式组件是没有this的，也就意味着组件本身是没有状态的，具体可以看看这篇[掘金blog](https://juejin.im/post/5b38f4bef265da59bc2cb921)

在完成上述准备操作之后，Vue会执行`new VNode(...params)`，得到组件对应的一个VNode节点(占位所用)，不过该VNode在当前context中，没有elm，只有componentInstance，这也是ref的时候，获取的是vm，可以直接调用methods中方法的原因
