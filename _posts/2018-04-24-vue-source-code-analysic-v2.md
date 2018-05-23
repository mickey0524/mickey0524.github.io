---
layout:     post
title:      "vue-source-code-analysic（2）"
subtitle:   "阅读Vue源码有感（2）"
date:       2018-04-24 23:00:00
author:     "Mickey"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
tags:
    - 前端开发
    - Vue
---

上一篇博客大概讲述了Vue中视图和数据绑定的方法，以及更新data中数据如何映射到视图中，这篇博客从源码的角度进行分析，加深理解～

首先看一下Vue官网的一张图片

![响应式原理](/img/in-post/vue-source-code-analysic-v2/1.png)

这张图比较清晰的展现了Vue中的响应式原理，和前一篇博客中展示的一样，首先通过一次渲染触发Data的get函数进行依赖收集，每一个data中的key对应一个dep观察者，dep中存在一个subs的数组，保存着该key所有的watcher实例，在data发生改变的时候，触发set函数，进而触发dep的notify函数，通知所有的watcher对象进行更新，之后再根据diff算法判断是否更新视图。

Vue在初始化组件数据时，在生命周期的`beforeCreate`与`created`钩子函数之间实现了对data、props、computed、methods、events以及watch的处理

## initData

initData首先对data进行一些预处理，然后调用observe()函数绑定Observer实例到data的\_\_ob\_\_属性上，进而进行依赖收集，可以参考源码instance下的state.js文件

```js
function initData (vm: Component) {
  /*
   * 得到data数据，将$options.data挂载到vm上
   */
  let data = vm.$options.data;
  data = vm._data = typeof data === 'function'
    ? getData(data, vm)
    : data || {};

  /**
   * 判断是否是对象
   */
  if (!isPlainObject(data)) {
    data = {}
    process.env.NODE_ENV !== 'production' && warn(
      'data functions should return an object:\n' +
      'https://vuejs.org/v2/guide/components.html#data-Must-Be-a-Function',
      vm
    )
  }

  /**
   * 遍历data对象，如果data中存在和props中相同的key，props优先，则报警
   */
  const keys = Object.keys(data);
  const props = vm.$options.props;
  let i = keys.length;

  while (i--) {
    if (props && hasOwn(props, keys[i])) {
      process.env.NODE_ENV !== 'production' && warn(
        `The data property "${keys[i]}" is already declared as a prop. ` +
        `Use prop default value instead.`,
        vm
      )
    }
    /**
     * 判断是否是保留字段，判断是否为$或_打头的key
     */ 
    else if (!isReserved(keys[i])) {
      /**
       * 这里是我们前面讲过的代理，将data上面的属性代理到了vm实例上
       */
      proxy(vm, `_data`, keys[i])
    }
  }
  /**
   * Github:https://github.com/answershuto
   * 从这里开始我们要observe了，开始对数据进行绑定，这里有尤大大的注释asRootData
   * 这步作为根数据，下面会进行递归observe进行对深层对象的绑定。
   */
  observe(data, true /* asRootData */)
}

function proxy (target: Object, sourceKey: string, key: string) {
  sharedPropertyDefinition.get = function proxyGetter () {
    return this[sourceKey][key]
  }
  sharedPropertyDefinition.set = function proxySetter (val) {
    this[sourceKey][key] = val
  }
  Object.defineProperty(target, key, sharedPropertyDefinition)
}
```

## observe

这个函数定义在core文件下oberver的index.js文件中

```js
/**
 * Attempt to create an observer instance for a value,
 * returns the new observer if successfully observed,
 * or the existing observer if the value already has one.
 */
/**
 * 尝试创建一个Observer实例（__ob__），如果成功创建Observer实例则返回新的Observer实例
 * 如果已有Observer实例则返回现有的Observer实例
 */
export function observe (value: any, asRootData: ?boolean): Observer | void {
  if (!isObject(value)) {
    return;
  }
  let ob: Observer | void;

  /**
   * 这里用__ob__这个属性来判断是否已经有Observer实例，如果没有Observer实例则会新建一个
   * Observer实例并赋值给__ob__这个属性，如果已有Observer实例则直接返回该Observer实例
   */
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
    ob = value.__ob__;
  } else if (
    /** 
     * 这里的判断是为了确保value是单纯的对象，而不是函数或者是Regexp等情况
     */
    observerState.shouldConvert &&
    !isServerRendering() &&
    (Array.isArray(value) || isPlainObject(value)) &&
    Object.isExtensible(value) &&
    !value._isVue
  ) {
    ob = new Observer(value);
  }
  if (asRootData && ob) {
    /**
     * 如果是根数据则计数，后面Observer中的observe的asRootData非true
     */
    ob.vmCount += 1;
  } 
  return ob;
}
```

Vue的响应式数据都会有一个\_\_ob\_\_的属性作为标记，里面存放了该属性的观察器，也就是Observer的实例，应用单例模式防止重复绑定，下面我们看一下Observer类的实现

## Observer

```js
/*
 * not type checking this file because flow doesn't play well with
 * dynamically accessing methods on Array prototype
 */

import { def } from '../util/index'

/**
 * 取得原生数组的原型
 * 创建一个新的数组对象，修改该对象上的数组的七个方法，防止污染原生数组方法
 */
const arrayProto = Array.prototype
export const arrayMethods = Object.create(arrayProto)

/**
 * 这里重写了数组的这些方法，在保证不污染原生数组原型的情况下重写数组的这些方法
 * 截获数组的成员发生的变化，执行原生数组操作的同时dep通知关联的所有观察者进行响应式处理
 */
;[
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
]
.forEach(function (method) {
  /**
   * 将数组的原生方法缓存起来，后面要调用
   */
  const original = arrayProto[method];
  def(arrayMethods, method, function mutator () {
    // avoid leaking arguments:
    // http://jsperf.com/closure-with-arguments
    let i = arguments.length;
    const args = new Array(i);
    while (i--) {
      args[i] = arguments[i];
    }
    /**
     * 调用原生的数组方法
     */
    const result = original.apply(this, args);

    /**
     * 数组新插入的元素需要重新进行observe才能响应式
     */
    const ob = this.__ob__;
    let inserted;
    switch (method) {
      case 'push':
        inserted = args;
        break;
      case 'unshift':
        inserted = args;
        break;
      case 'splice':
        inserted = args.slice(2);
        break;
    }
    if (inserted) ob.observeArray(inserted)
      
    /** 
     * dep通知所有注册的观察者进行响应式处理
     */
    ob.dep.notify();
    return result;
  });
});

/**
 * Observer class that are attached to each observed
 * object. Once attached, the observer converts target
 * object's property keys into getter/setters that
 * collect dependencies and dispatches updates.
 */
export class  {
  value: any;
  dep: Dep;
  vmCount: number; // number of vms that has this object as root $data

  constructor (value: any) {
    this.value = value;
    this.dep = new Dep();
    this.vmCount = 0;

    /** 
     * 将Observer实例绑定到data的__ob__属性上面去
     * 之前说过observe的时候会先检测是否已经有__ob__对象存放Observer实例了
     * def方法定义可以参考
     * https://github.com/vuejs/vue/blob/dev/src/core/util/lang.js#L16 
     */
    def(value, '__ob__', this);
    if (Array.isArray(value)) {

      /**
       * 如果是数组，将修改后可以截获响应的数组方法替换掉该数组的原型中的原生方法
       * 达到监听数组数据变化响应的效果
       * 这里如果当前浏览器支持__proto__属性，则直接覆盖当前数组对象原型上的原生数组方法
       * 如果不支持该属性，则直接覆盖数组对象的原型
       */
      const augment = hasProto
        ? protoAugment  // 直接覆盖原型的方法来修改目标对象
        : copyAugment   // 定义（覆盖）目标对象或数组的某一个方法
      augment(value, arrayMethods, arrayKeys);
      /**
       * Github:https://github.com/answershuto
       * 如果是数组则需要遍历数组的每一个成员进行observe
       */
      this.observeArray(value);
    } else {
      /**
       * 如果是对象则直接walk进行绑定
       */
      this.walk(value);
    }
  }

  /**
   * Walk through each property and convert them into
   * getter/setters. This method should only be called when
   * value type is Object.
   */
  walk (obj: Object) {
    const keys = Object.keys(obj);

    /**
     * walk方法会遍历对象的每一个属性进行defineReactive绑定
     */
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i], obj[keys[i]]);
    }
  }

  /**
   * Observe a list of Array items.
   */
  observeArray (items: Array<any>) {

    /**
     * 数组需要便利每一个成员进行observe
     */
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i]);
    }
  }
}

/**
 * Augment an target Object or Array by intercepting
 * the prototype chain using __proto__
 */
/**
 * 直接覆盖原型的方法来修改目标对象或数组
 */
function protoAugment (target, src: Object) {
  /* eslint-disable no-proto */
  target.__proto__ = src;
  /* eslint-enable no-proto */
}

/**
 * Augment an target Object or Array by defining
 * hidden properties.
 */
/**
 * 定义（覆盖）目标对象或数组的某一个方法
 */
function copyAugment (target: Object, src: Object, keys: Array<string>) {
  for (let i = 0, l = keys.length; i < l; i++) {
    const key = keys[i];
    def(target, key, src[key]);
  }
}
```

Observer为每个数组进行视图和数据的绑定，当遇到data[key]为对象的时候，需要deep绑定，当data[key]为array的时候，需要对数组中的每个子元素进行绑定，我们知道Array.prototype中存在push，pop等方法用于操作数组，在vue中，同样可以使用这些方法，当修改数组中的元素的时候，我们同样需要监听改变，重新渲染视图，这个时候，我们就要考虑在原生方法上增加一些自定义操作了，其实说白了就是操作数组元素的时候调用`dep.notify()`通知更新以及给新增加的元素绑定Observer实例，于是我们需要修改Array的原型，如果浏览器支持\_\_proto\_\_的话，我们直接覆盖\_\_proto\_\_即可，否则我们需要重新定义，效率较低，因此优先考虑覆盖\_\_proto\_\_

## Dep

Dep是依赖收集类，用于收集观察者对象，在data更改的时候，执行notify方法调用每一个watcher实例的update方法更新视图

```js
/**
 * A dep is an observable that can have multiple
 * directives subscribing to it.
 */
export default class Dep {
  static target: ?Watcher;
  id: number;
  subs: Array<Watcher>;

  constructor () {
    this.id = uid++
    this.subs = []
  }

  /** 
   * 添加一个观察者对象
   */
  addSub (sub: Watcher) {
    this.subs.push(sub)
  }

  /**
   * 移除一个观察者对象
   */
  removeSub (sub: Watcher) {
    remove(this.subs, sub)
  }

  /**
   * 依赖收集，当存在Dep.target的时候添加观察者对象
   * 其实Watcher和Dep是完全双向数据互通的
   * 只要Dep的subs里面存在watcher实例
   * Watcher的Deps一定存在相应的dep实例
   */
  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }

  /** 
   * 通知所有订阅者
   */
  notify () {
    // stabilize the subscriber list first
    const subs = this.subs.slice()
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}

// the current target watcher being evaluated.
// this is globally unique because there could be only one
// watcher being evaluated at any time.
Dep.target = null // 依赖收集完需要将Dep.target设为null，防止后面重复添加依赖
```

## Watcher

Watcher是观察者对象，首次render后存放于Dep中，当data更改后，由Dep通知Watcher执行update函数重新渲染视图

```js
export default class Watcher {
  vm: Component;
  expression: string;
  cb: Function;
  id: number;
  deep: boolean;
  user: boolean;
  lazy: boolean;
  sync: boolean;
  dirty: boolean;
  active: boolean;
  /**
   * 用deps存储当前的依赖，而新一轮的依赖收集过程中收集到的依赖则会放到newDeps中
   * 之所以要用一个新的数组存放新的依赖是因为当依赖变动后
   * 比如由依赖a和b变成依赖a和c
   * 那么需要把原先的依赖订阅清除掉，也就是从b的subs数组中移除当前的watcher，因为我已经不想监听b的变动
   * 所以我要比对deps和newDeps，找出那些不再依赖的dep，然后dep.removeSub(当前watcher)
   */
  deps: Array<Dep>;
  newDeps: Array<Dep>;
  depIds: ISet;
  newDepIds: ISet;
  getter: Function;
  value: any;

  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: Object
  ) {
    this.vm = vm;
    /*_watchers存放订阅者实例*/
    vm._watchers.push(this);
    // options
    if (options) {
      this.deep = !!options.deep;
      this.user = !!options.user;
      this.lazy = !!options.lazy;
      this.sync = !!options.sync;
    } else {
      this.deep = this.user = this.lazy = this.sync = false;
    }
    this.cb = cb;
    this.id = ++uid; // uid for batching
    this.active = true;
    this.dirty = this.lazy; // for lazy watchers
    this.deps = [];
    this.newDeps = [];
    this.depIds = new Set();
    this.newDepIds = new Set();
    this.expression = process.env.NODE_ENV !== 'production'
      ? expOrFn.toString()
      : '';
    // parse expression for getter
    /*把表达式expOrFn解析成getter*/
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn;
    } else {
      this.getter = parsePath(expOrFn);
      if (!this.getter) {
        this.getter = function () {}
        process.env.NODE_ENV !== 'production' && warn(
          `Failed watching path: "${expOrFn}" ` +
          'Watcher only accepts simple dot-delimited paths. ' +
          'For full control, use a function instead.',
          vm
        );
      }
    }
    this.value = this.lazy
      ? undefined
      : this.get();
  }

  /**
   * Evaluate the getter, and re-collect dependencies.
   */
   /*获得getter的值并且重新进行依赖收集*/
  get () {
    /*将自身watcher观察者实例设置给Dep.target，用以依赖收集。*/
    pushTarget(this);
    let value;
    const vm = this.vm;

    /*
      执行了getter操作，看似执行了渲染操作，其实是执行了依赖收集。
      在将Dep.target设置为自生观察者实例以后，执行getter操作。
      譬如说现在的的data中可能有a、b、c三个数据，getter渲染需要依赖a跟c，
      那么在执行getter的时候就会触发a跟c两个数据的getter函数，
      在getter函数中即可判断Dep.target是否存在然后完成依赖收集，
      将该观察者对象放入闭包中的Dep的subs中去。
    */
    if (this.user) {
      try {
        value = this.getter.call(vm, vm);
      } catch (e) {
        handleError(e, vm, `getter for watcher "${this.expression}"`);
      }
    } else {
      value = this.getter.call(vm, vm);
    }
    // "touch" every property so they are all tracked as
    // dependencies for deep watching
    /*如果存在deep，则触发每个深层对象的依赖，追踪其变化*/
    if (this.deep) {
      /*递归每一个对象或者数组，触发它们的getter，使得对象或数组的每一个成员都被依赖收集，形成一个“深（deep）”依赖关系*/
      traverse(value);
    }

    /*将观察者实例从target栈中取出并设置给Dep.target*/
    popTarget();
    this.cleanupDeps();
    return value;
  }

  /**
   * Add a dependency to this directive.
   */
   /*添加一个依赖关系到Deps集合中*/
  addDep (dep: Dep) {
    const id = dep.id;
    if (!this.newDepIds.has(id)) {
      this.newDepIds.add(id);
      this.newDeps.push(dep);
      if (!this.depIds.has(id)) {
        dep.addSub(this);
      }
    }
  }

  /**
   * Clean up for dependency collection.
   */
   /*清理依赖收集*/
  cleanupDeps () {
    /*移除所有观察者对象*/
    let i = this.deps.length;
    while (i--) {
      const dep = this.deps[i];
      if (!this.newDepIds.has(dep.id)) {
        dep.removeSub(this);
      }
    }
    let tmp = this.depIds;
    this.depIds = this.newDepIds;
    this.newDepIds = tmp;
    this.newDepIds.clear();
    tmp = this.deps;
    this.deps = this.newDeps;
    this.newDeps = tmp;
    this.newDeps.length = 0;
  }

  /**
   * Subscriber interface.
   * Will be called when a dependency changes.
   */
   /*
      调度者接口，当依赖发生改变的时候进行回调。
   */
  update () {
    /* istanbul ignore else */
    if (this.lazy) {
      this.dirty = true;
    } else if (this.sync) {
      /*同步则执行run直接渲染视图*/
      this.run();
    } else {
      /*异步推送到观察者队列中，由调度者调用。*/
      queueWatcher(this);
    }
  }

  /**
   * Scheduler job interface.
   * Will be called by the scheduler.
   */
   /*
      调度者工作接口，将被调度者回调。
    */
  run () {
    if (this.active) {
      const value = this.get();
      if (
        value !== this.value ||
        // Deep watchers and watchers on Object/Arrays should fire even
        // when the value is the same, because the value may
        // have mutated.
        /*
            即便值相同，拥有Deep属性的观察者以及在对象／数组上的观察者应该被触发更新，因为它们的值可能发生改变。
        */
        isObject(value) ||
        this.deep;
      ) {
        // set new value
        const oldValue = this.value;
        /*设置新的值*/
        this.value = value;

        /*触发回调渲染视图*/
        if (this.user) {
          try {
            this.cb.call(this.vm, value, oldValue);
          } catch (e) {
            handleError(e, this.vm, `callback for watcher "${this.expression}"`);
          }
        } else {
          this.cb.call(this.vm, value, oldValue);
        }
      }
    }
  }

  /**
   * Evaluate the value of the watcher.
   * This only gets called for lazy watchers.
   */
  /*
   * 获取观察者的值
   * 比如computed的this.lazy=true
   * 只有到需要的时候才会调用evaluate方法
   */
  evaluate () {
    this.value = this.get();
    this.dirty = false;
  }

  /**
   * Depend on all deps collected by this watcher.
   */
   /*
    * 收集该watcher的所有deps依赖
    */
  depend () {
    let i = this.deps.length;
    while (i--) {
      this.deps[i].depend();
    }
  }

  /**
   * Remove self from all dependencies' subscriber list.
   */
   /*将自身从所有依赖收集订阅列表删除*/
  teardown () {
    if (this.active) {
      // remove self from vm's watcher list
      // this is a somewhat expensive operation so we skip it
      // if the vm is being destroyed.
      /*从vm实例的观察者列表中将自身移除，由于该操作比较耗费资源，所以如果vm实例正在被销毁则跳过该步骤。*/
      if (!this.vm._isBeingDestroyed) {
        remove(this.vm._watchers, this);
      }
      let i = this.deps.length;
      while (i--) {
        this.deps[i].removeSub(this);
      }
      this.active = false;
    }
  }
}
```

## defineReactive

defineReactive应该是Vue中的核心方法了，通过这个方法，调用`Object.defineProperty`方法为数据定义了set和get方法，实现了初次渲染的依赖收集，以及数据更改时的视图更新

```js
/**
 * Define a reactive property on an Object.
 */
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: Function
) {
  /**
   * 定义一个dep对象
   */
  const dep = new Dep();

  const property = Object.getOwnPropertyDescriptor(obj, key);
  if (property && property.configurable === false) {
    return;
  }

  /**
   * 如果之前该对象已经预设了getter以及setter函数则将其取出来，新定义的getter/setter中会将其执行
   * 保证不会覆盖之前已经定义的getter/setter
   */
  // cater for pre-defined getter/setters
  const getter = property && property.get;
  const setter = property && property.set;

  /**
   * 对象的子对象递归进行observe并返回子节点的Observer对象
   * 如果val为对象或者数组，那么该对象和数组会有一个__ob__属性
   * 用于存放Observer实例，当使用Vue.setthis.$set更改Array和Object的时候
   * target.__ob__.dep.notity()能够触发更新，不会丢失
   */
  let childOb = observe(val);
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {

      /*如果原本对象拥有getter方法则执行*/
      const value = getter ? getter.call(obj) : val;
      if (Dep.target) {
        dep.depend();
        if (childOb) {

          /**
           * 对象进行依赖收集，其实就是将同一个watcher观察者实例放进了两个depend中
           * 一个是正在本身闭包中的depend，另一个是子元素的depend
           */
          childOb.dep.depend();
        }
        if (Array.isArray(value)) {

          /**
           * 是数组则需要对每一个成员都进行依赖收集，如果数组的成员还是数组，则递归
           */
          dependArray(value);
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {

      /**
       * 通过getter方法获取当前值，与新值进行比较，一致则不需要执行下面的操作
       */
      const value = getter ? getter.call(obj) : val;
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return;
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter();
      }
      if (setter) {

        /**
         * 如果原本对象拥有setter方法则执行setter
         */
        setter.call(obj, newVal);
      } else {
        val = newVal;
      }

      /**
       * 新的值需要重新进行observe，保证数据响应式
       */
      childOb = observe(newVal);

      /**
       * dep对象通知所有的观察者
       */
      dep.notify();
    }
  })
}
```