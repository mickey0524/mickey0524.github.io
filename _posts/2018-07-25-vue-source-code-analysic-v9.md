---
layout:     post
title:      "vue-source-code-analysic（9）"
subtitle:   "阅读Vue源码有感（9）"
date:       2018-07-25 00:30:00
author:     "Mickey"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
tags:
    - 前端开发
    - Vue
---

在上篇博客中，我们介绍了Vue中相当重要的两个文件--`create-element.js`以及`create-component.js`，这篇博客我们来介绍一下`patch.js`文件，也就是Vue中最核心的AST更新的知识点

首先需要明确的是，Vue使用的diff算法来自于[snabbdom](https://github.com/snabbdom/snabbdom)，算法复杂度为O(n)

## 什么是virtual dom

如果不了解virtual dom，要理解Vue的diff过程是非常困难的，虚拟dom其实就是对真实dom节点的映射，使用`document.createElement`和`document.createTextNode`所创建的就是真实节点。

下面举个真实dom映射虚拟dom的栗子

```html
<div class="dom-node">
  Vue  
</div>
```

可以用如下的数据结构来代表上面的HTML

```js
{
    tag: 'div',
    staticClass: 'dom-node',
    children: [
        {
            text: 'Vue',
        }
    ],
}
```

可能很多同学会有疑问，jq天下第一，手动修改dom节点可还行，当dom结构相对简单的时候，手动修改dom节点的效率是高于MVVM框架的，但是，当页面庞大，结构很复杂的时候，手工修改节点会显得繁琐而不可维护，而virtual dom能够让fe专注于js层的代码，由框架来进行页面的render，代码可读性高，可维护性强，对fe也更友好

## diff图解

其实MVVM框架的diff过程都大同小异，都是将dom抽象层树型结构，然后子节点同层之间进行比较，如下图所示:

![diff图](/img/in-post/vue-source-code-analysic-v9/diff.png)

另外分享一篇有关diff的相当经典的文章[React’s diff algorithm](https://calendar.perfplanet.com/2013/diff/)

## 源码分析

大概介绍了一下virtual dom之后，让我们对Vue的源码进行逐个功能点的分析

### function sameVnode

```js
function sameVnode (a, b) {
  return (
    a.key === b.key &&
    a.tag === b.tag &&
    a.isComment === b.isComment &&
    isDef(a.data) === isDef(b.data) &&
    sameInputType(a, b)
  )
}
```

这个函数用于判断两个VNode节点是否是同一个节点，需要满足以下条件:

* key相同
* tag（当前节点的标签名）相同
* isComment（是否为注释节点）相同
* 是否data（当前节点对应的对象，包含了具体的一些数据信息，是一个VNodeData类型，可以参考VNodeData类型中的数据信息）都有定义
* 当标签是`<input>`的时候，type必须相同

### function patchVnode

执行`patch`操作的时候，Vue调用`sameVNode`发现是同一个节点，就会调用`patchVnode`方法直接修改当前的节点

```js
function patchVnode (oldVnode, vnode, insertedVnodeQueue, removeOnly) {
  // 两个VNode节点相同则直接返回
  if (oldVnode === vnode) {
    return
  }
  // reuse element for static trees.
  // note we only do this if the vnode is cloned -
  // if the new node is not cloned it means the render functions have been
  // reset by the hot-reload-api and we need to do a proper re-render.
  // 如果新旧VNode都是静态的，同时它们的key相同（代表同一节点），
  // 并且新的VNode是clone或者是标记了once（标记v-once属性，只渲染一次），
  // 那么只需要替换elm以及componentInstance即可。
  if (isTrue(vnode.isStatic) &&
      isTrue(oldVnode.isStatic) &&
      vnode.key === oldVnode.key &&
      (isTrue(vnode.isCloned) || isTrue(vnode.isOnce))) {
    vnode.elm = oldVnode.elm
    vnode.componentInstance = oldVnode.componentInstance
    return
  }
  let i
  const data = vnode.data
  if (isDef(data) && isDef(i = data.hook) && isDef(i = i.prepatch)) {
    // i = data.hook.prepatch，如果存在的话，见"./create-component componentVNodeHooks"。
    i(oldVnode, vnode)
  }
  const elm = vnode.elm = oldVnode.elm
  const oldCh = oldVnode.children
  const ch = vnode.children
  if (isDef(data) && isPatchable(vnode)) {
    // 调用update回调以及update钩子
    for (i = 0; i < cbs.update.length; ++i) cbs.update[i](oldVnode, vnode)
    if (isDef(i = data.hook) && isDef(i = i.update)) i(oldVnode, vnode)
  }
  // 如果这个VNode节点没有text文本时
  if (isUndef(vnode.text)) {
    if (isDef(oldCh) && isDef(ch)) {
      // 新老节点均有children子节点，则对子节点进行diff操作，调用updateChildren
      if (oldCh !== ch) updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly)
    } else if (isDef(ch)) {
      // 如果老节点没有子节点而新节点存在子节点，先清空elm的文本内容，然后为当前节点加入子节点
      if (isDef(oldVnode.text)) nodeOps.setTextContent(elm, '')
      addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue)
    } else if (isDef(oldCh)) {
      // 当新节点没有子节点而老节点有子节点的时候，则移除所有ele的子节点
      removeVnodes(elm, oldCh, 0, oldCh.length - 1)
    } else if (isDef(oldVnode.text)) {
      // 当新老节点都无子节点的时候，只是文本的替换，因为这个逻辑中新节点text不存在，所以直接去除ele的文本
      nodeOps.setTextContent(elm, '')
    }
  } else if (oldVnode.text !== vnode.text) {
    // 当新老节点text不一样时，直接替换这段文本
    nodeOps.setTextContent(elm, vnode.text)
  }
  // 调用postpatch钩子
  if (isDef(data)) {
    if (isDef(i = data.hook) && isDef(i = i.postpatch)) i(oldVnode, vnode)
  }
}
```

🚧under construction🚧
