---
layout:     post
title:      "promise-implement"
subtitle:   "手写promise"
date:       2018-03-17 23:30:00
author:     "Mickey"
header-img: "img/post-bg-ios9-web.jpg"
tags:
    - 前端开发
    - javaScript
---

在ES6中，Promise算是非常重要的一部分，它的出现，很大程度上解决了js中回调地狱的问题，极大的简化了js代码的书写，这篇文章基于的是Promise/A+的规范来实现的，对应的[代码仓库](https://github.com/mickey0524/promise-implement)

## promise-version-1

第一个版本的promise还是相当简单的，then方法绑定回调函数至promise实例上，resolve函数执行的时候，依次执行绑定的回调函数，类似于观察者模式

```js
/**
 * 最简单的Promise雏形
 */
function Promise(fn) {
  var callbacks = [];
  
  this.then = (onFulfilled) => {
    callbacks.push(onFulfilled);
  }

  function resolve(value) {
    callbacks.forEach(callback => {
      callback(value);
    });
  }

  fn(resolve);
}
```

## promise-version-2

v1的promise中如果执行的是同步代码的话，还没等执行then函数绑定回调函数就resolve了，这样callbacks中的回调函数就不会得到执行，可以利用setTimeout(() => {}, 0)将callbacks中函数的执行放到这次事件循环的末尾，同时在then函数中return this返回当前实例，便于链式调用

```js
/**
 * 利用setTimeout将callbacks的处理放到事件循环的最后
 * then方法返回this用以完成链式调用
 */
function Promise(fn) {
  var callbacks = [];

  this.then = (onFulfilled) => {
    callbacks.push(onFulfilled);
    return this;
  }

  function resolve(value) {
    setTimeout(() => {
      callbacks.forEach(callback => {
        callback(value);
      });
    }, 0);
  }

  fn(resolve);
}
```

## promise-version-3

v2中的promise在resolve之后通过then绑定上的回调函数是不会被执行的，因为错过了foreach的时间点，增加value变量记录resolve的参数以及state变量记录当前promise实例的状态，用于决定通过then方法绑定的回调函数的执行形式，如果当前还在pending状态，则回调函数存入callbacks，如果已经是resolved的状态，则立即执行

```js
/**
 * 增加state，用于处理在Promise resolve 之后通过then注册的回调函数
 */
function Promise(fn) {
  var state = pending,
    value = null,
    callbacks = [];
  
  this.then = (onFulfilled) => {
    if (state === 'pending') {
      callbacks.push(onFulfilled);
      return this;
    }
    onFulfilled(value);
    return this;
  }

  function resolve(newValue) {
    state = 'resolved';
    value = newValue;
    setTimeout(() => {
      callbacks.forEach(callback => {
        callback(newValue);
      });
    }, 0);
  }

  fn(resolve);
}
```

## promise-version-4

v3和v2一样，也是通过返回this来进行链式调用，这是不对的，如下所示

```js
new Promise((resolve) => {
  resolve(1);
}).then(res => {
  console.log(res);
}).then(res => {
  console.log(res);
});

v3版本的promise的输出应该是 1 1，而真正的promise的输出应该是 1 undefined
```

因此，then函数不应该返回当前的promise实例，而应该返回一个新创建的桥接promise实例，当then函数绑定的回调函数返回的也是一个promise实例的时候，最关键的在于要将桥接promise的resolve挂在then函数的参数返回的promise上，这样才能完成链式传值

```js
/**
 * then 方法返回一个 new 出来的Promise用来作为桥接Promise
 * 这样实现了串形链式调用
 * 关键就是给then里面return 的 promise 注册一个 桥接 promise 的 resolve 函数
 */
function Promise(fn) {
  var state = 'pending',
    value = null,
    callbacks = [];

  this.then = function(onFulfilled) {
    return new Promise(resolve => {
      handle({
        onFulfilled: onFulfilled || null,
        resolve: resolve,
      });
    });
  }

  function handle(callback) {
    if (state === 'pending') {
      callback.push(callback);
      return;
    }
    if (!callback.onFulfilled) {
      callback.resolve(value);
      return;
    }
    var ret = callback.onFulfilled(value);
    callback.resolve(ret);
  }

  function resolve(newValue) {
    if (newValue && (typeof newValue === 'object' || typeof newValue === 'function')) {
      var then = newValue.then;
      if (typeof then === 'function') {
        then.call(newValue, resolve);
        return;
      }
    }
    state = 'resolved';
    value = newValue;
    setTimeout(() => {
      callbacks.forEach(callback => {
        handle(callback);
      });
    }, 0);
  }

  fn(resolve);
}
```

## promise-version-5

v4的promise算是一个比较完备的实现了，我们知道真正的promise的then函数是可以传入两个参数的，一个用于执行resolved的回调，另外一个用于执行rejected的回调，在此版本中，我们增加rejected状态，同时实现then函数传入两个参数的需求

```js

/**
 * 前面4个版本的Promise只考虑了resolve，这个版本考虑reject
 */
function Promise(fn) {
  var state = 'pending',
    value = null,
    callbacks = [];

  this.then = (onFulfilled, onRejected) => {
    return new Promise((resolve, reject) => {
      handle({
        onFulfilled: onFulfilled || null,
        onRejected: onRejected || null,
        resolve: resolve,
        reject: reject
      });
    });
  }

  function handle(callback) {
    if (state === 'pending') {
      callbacks.push(callback);
      return;
    }
    var cb = state === 'resolved' ? callback.onFulfilled : callback.onRejected,
      ret;
    if (cb == null) {
      cb = cb === 'resolved' ? callback.resolve : callback.reject;
      cb(value);
      return;
    }
    ret = cb(value);
    callback.resolve(ret);
  }
  
  function resolve(newValue) {
    if (newValue && (typeof newValue === 'object' || typeof newValue === 'function')) {
      var then = newValue.then;
      if (typeof then === 'function') {
        then.call(newValue, resolve, reject);
        return;
      }
    }
    state = 'resolved';
    value = newValue;
    execute();
  }

  function reject(error) {
    state = 'rejected';
    value = error;
    execute();
  }

  function execute() {
    setTimeout(() => {
      callbacks.forEach(callback => {
        handle(callback);
      })
    }, 0);
  }

  fn(resolve, reject);
}
```

## promise-version-6

v5版本只能通过then函数的第二个参数来捕捉错误，这是es6中不推荐的方法，es6推荐使用catch来捕捉错误，在此版本中，我们实现了catch方法

```js
/**
 * v5版本给then方法增加 onRejected 方法，v6尝试实现catch方法
 */
function Promise(fn) {
  var state = 'pending',
    value = null,
    callbacks = [];

  this.then = (onFulfilled, onRejected) => {
    return new Promise((resolve, reject) => {
      handle({
        onFulfilled: onFulfilled || null,
        onRejected: onRejected || null,
        resolve: resolve,
        reject: reject
      });
    });
  }

  this.catch = (onRejected) => {
    return new Promise((resolve, reject) => {
      handle({
        onRejected: onRejected || null,
        reject: reject
      });
    });
  }

  function handle(callback) {
    if (state === 'pending') {
      callbacks.push(callback);
      return;
    }
    var cb = state === 'resolved' ? callback.onFulfilled : callback.onRejected,
      ret;
    if (cb == null) {
      cb = cb === 'resolved' ? callback.resolve : callback.reject;
      cb(value);
      return;
    }
    ret = cb(value);
    cb = callback.resolve || callback.reject;
    cb(ret);
  }

  function resolve(newValue) {
    if (newValue && (typeof newValue === 'object' || typeof newValue === 'function')) {
      var then = newValue.then;
      if (typeof then === 'function') {
        then.call(newValue, resolve, reject);
        return;
      }
    }
    state = 'resolved';
    value = newValue;
    execute();
  }

  function reject(error) {
    state = 'rejected';
    value = error;
    execute();
  }

  function execute() {
    setTimeout(() => {
      callbacks.forEach(callback => {
        handle(callback);
      })
    }, 0);
  }

  fn(resolve, reject);
}
```

## 总结

通过阅读源码，实现copy版本以及撰写博客，让我对promise的理解变得更深了，希望对各位看官有所帮助～