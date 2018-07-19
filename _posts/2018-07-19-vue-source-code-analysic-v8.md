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

🚧under construction🚧



