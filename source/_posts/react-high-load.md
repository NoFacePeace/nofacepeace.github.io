---
title: 记一次排查 React 应用跑满 CPU 问题
date: 2022-04-23 21:53:22
tags:
---

> 一般来说，跑满 CPU 大部分都是程序陷入了死循环，优先朝着这个方向进行排查

<!-- more -->

## 问题

近期有同事反馈说，线上 React 应用把 CPU 跑满了，我打开页面同时打开 Chrome 的任务管理器一看，确实 CPU 跑到 100% 以上。

![](task-manager.png)

## 排查

随即打开开发者工具，切换到源代码 Tab，点击暂停脚本执行，发现调用堆栈重复调用 onmessage 和 postMessage 两个函数。

![](stack.png)

进一步点击单步调试，发现程序在项目代码和 Chrome 扩展来回执行，猜想是不是由于 Chrome 扩展导致的死循环？
我重新用无痕模式打开页面，同时关注任务管理器，发现 CPU 是正常的，尝试在控制台执行 `window.postMessage("hh", "*")`，发现 CPU 跑上来了，看起来像是 Chrome 扩展执行了 window.postMessage 函数触发应用某段死循环代码。
React 应用本身没有监听 message 这个事件，猜想是不是引用的依赖库监听 message 事件导致？
把代码拉到本地运行起来，通过无痕模式打开页面，依旧在控制台执行 `window.postMessage("hh", "*")`，发现 CPU 跑上去了，点击暂停脚本执行，程序停止在 index.worker.js 这个文件。

![](source.png)

通过阅读代码，我们发现，这是一段会造成死循环的代码，onmesage 里面又调用 postMessage，相当于自己监控 message 事件，自己同时也发送 message 消息。
这段代码出自 @antv/algorithm 这个库，本身是 avtv 内置的常用的图算法 JS 实现，提供给 G6 及 Graphin 用于图分析场景使用。
React 应用本身没有直接依赖这个库，通过 package.json 文件发现，应用直接依赖 @antv/g6 库，而这个库依赖链如下

1. @antv/g6-pc
2. @antv/g6-core
3. @antv/algorithm

@antv/algorithm 的版本是 `0.1.8-beta.5`，从 node_modules 里面查看 index.worker.js 的实现，如下

```
import * as algorithm from './algorithm';
import { MESSAGE } from './constant';
var ctx = self;

ctx.onmessage = function (event) {
  var _a = event.data,
      type = _a.type,
      data = _a.data;

  if (typeof algorithm[type] === 'function') {
    var result = algorithm[type].apply(algorithm, data);
    ctx.postMessage({
      type: MESSAGE.SUCCESS,
      data: result
    });
    return;
  }

  ctx.postMessage({
    type: MESSAGE.FAILURE
  });
}; // https://stackoverflow.com/questions/50210416/webpack-worker-loader-fails-to-compile-typescript-worker


export default null;
```

从源码可知，如果其他地方执行过 window.postMessage 的话，这段代码会陷入死循环。
从 github 访问 @antv/algorithm 最新版本的实现，发现已经 fix 这个问题，新版本改用 TypeScript，index.worker.ts 代码如下

```
import * as algorithm from './algorithm';
import { MESSAGE } from './constant';

const ctx: Worker = (typeof self !== 'undefined') ? self : {} as any;

interface Event {
  type: string;
  data: any;
}

ctx.onmessage = (event: Event) => {
  const { _algorithmType, data } = event.data;
  // 如果发送内容没有私有类型。说明不是自己发的。不管
  // fix: https://github.com/antvis/algorithm/issues/25
  if(!_algorithmType){
    return;
  }
  if (typeof algorithm[_algorithmType] === 'function') {
    const result = algorithm[_algorithmType](...data);
    ctx.postMessage({ _algorithmType: MESSAGE.SUCCESS, data: result });
    return;
  }
  ctx.postMessage({ _algorithmType: MESSAGE.FAILURE });
};

// https://stackoverflow.com/questions/50210416/webpack-worker-loader-fails-to-compile-typescript-worker
export default null as any;
```

代码注释里面有个 issue 的链接，点击跳转过去，该 issue 确实也是反馈这段代码会陷入死循环，链接如下：
https://github.com/antvis/algorithm/blob/master/packages/graph/src/workers/index.worker.ts

## 思考

1. 奇怪的是，陷入死循环，为什么页面不会卡死，还能响应点击等其他事件
   这里的问题出在 window.onmessage 里面调用 window.postMessage，而事件监听是异步，其他事件是可以获得执行时间的，不像一般的逻辑死循环，例如 for、while、递归这种，整个执行线程都被占用
1. Chrome 扩展为什么会调用 postMessage？
   通过阅读 Chrome 扩展文档，了解到页面和扩展是可以使用 window.postMessage 通信的
   https://developer.chrome.com/docs/extensions/mv3/content_scripts/#host-page-communication
1. 在实现事件监听回调的时候，如果有必要再触发事件本身，提前对事件消息进行校验，避免响应非自身触发的事件
1. 在保持兼容的情况，尽量使用最新版本的库，新版本一般会伴随着修复旧版本的 Bug
1. 开发者在使用第三方库的时候，不能因为库流行或者是大厂开源就完全信赖，还是要对依赖库有更深入的了解，防止 Bug 产生
