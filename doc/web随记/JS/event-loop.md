# JS运行机制及Event Loop

> 了解js执行顺序及机制，更好的理解js特性，理顺代码中的一些异步操作 🚀

- JavaScript是单线程的语言
- Event Loop是javascript的执行机制
- synchronous：同步任务、asynchronous：异步任务、task queue/callback queue：任务队列、execution context stack：执行栈、heap：堆、stack：栈、macro-task：宏任务、micro-task：微任务

## 背景

JS 从诞生起就是一门单线程的语言。至于为什么是**单线程**，是因为作者认为 JS 是在浏览器执行的脚本语言，对它的要求不是很高，早期的网页对 JS 需求没那么高，都是轻量级的。而且写起来一定要简单，而多线程逻辑会造成交互、DOM 操作复杂。

**单线程问题**，任务执行阻塞，具现到页面上：ajax数据请求，http延迟，等待完成，其他操作无法执行，页面长时间无法接受反应。像页面操作（点击）的代码就无法立即执行，只是 WebAPIs 把 callback 放到任务队列里而已，用户就会认为页面卡住了，操作无反应了。

另外，UI 渲染和 JS 执行是互斥的。虽然两者属于不同的线程，但是由于 JS 执行结果可能会对页面产生影响，所以浏览器对此做了处理，大部分情况下 JS 线程执行，render 线程就会暂停，当 JS 的同步代码执行完再去渲染。

所以 JS 用**异步任务 (asynchronous callback)**去解决这个问题。

## 阻塞、异步

```js
fetch('url'); //假设是同步 fetch，阻塞代码执行。

render(); // 阻塞情况下无法执行
```

上面提到的这种执行模式算是阻塞执行，也就是在任务返回结果之前，JS 引擎线程 pending 了，导致后面的任务阻塞
对于浏览器这种实时性、交互性要求高的场景，肯定是不允许的。所以首先要**非阻塞调用**。非阻塞调用就是浏览器发送的这个请求任务，不需要等待结果就立即返回，但是具体数据回没回来，就需要 JS 线程**不定时的查看**了。而更好的做法是，这个请求任务完成后（得到数据），通知我们，让我们去获取数据。这就需要用到**异步 I/O 的模式**。主线程执行遇到异步任务（请求），不需要等待结果返回，只需要代码块执行完毕，即去运行其他的任务。当异步任务完成后，会以某种方式“通知”主线程，假如主线程处于闲置，就会接收异步任务的结果，亦或是没有结果。在 JS 中，异步任务执行完都是以 **callback** 的形式，返回给主线程结果的。所以，JS 是这种异步非阻塞的方式，去满足页面实时交互的需求，避免“页面”假死的。

**浏览器小补充**：

现代浏览器一个 tab 的线程包括不局限于：

- GUI 渲染线程
- JS引擎线程
- 事件触发线程
- 定时器触发线程
- 异步http请求线程

JS 中的异步操作是通过 WebAPIs 去支持的，常见的有 `XMLHttpRequest`，`setTimeout`，事件回调（`onclik`, `onscroll`等）。而这几个 API 浏览器都提供了单独的线程去运行，所以才会有地方去做定时器的计时，request 的回调。

即当代码中出现这几个定义的异步任务，是浏览器提供能力去满足需求，**是不跟 JS 引擎同属一个线程的，是浏览器实现了它们跟 JS 引擎的通信**。

## 任务队列

上面提到了异步任务完成后，会通知主线程，以 callback 的方式获取结果或者执行回调。但是如果当前的主线程是忙碌的，异步任务的信号无法接收到怎么办呢？
所以还需要一个地方保存这些 callback，也就是**任务队列(task queue)**

>🔥描述：就是主线程运行产生**堆、栈**，执行栈遇到异步任务（浏览器通常是调用 WebAPIs），不会等待，而是继续执行往下执行。而异步任务就会以各种方式，把 callback 加入任务队列中。待当前执行栈执行完毕，也就是出栈完毕，主线程就会从任务队列里读取第一个 callback，执行。同样的生成执行栈，结束后主线程如果发现任务队列中还有 callback，则会继续取出执行，如此重复操作。而这种循环的机制，就称之为**事件循环（Event Loop）**。

![Event-Loop](https://raw.githubusercontent.com/NARUTOne/resources-github/master/imgs/js/Event-Loop.png)

## Event Loop

> 一种确保了这些异步任务的有序执行的机制

- heap（堆）是用户主动请求而划分出来的内存区域，比如你new Object()，就是将一个对象存入堆中，可以理解为heap存对象。
- stack（栈）是由于函数运行而临时占用的内存区域，函数都存放在栈里。

```js
setTimeout(() => {
  console.log(1);
}, 1000);

setTimeout(() => {
  console.log(2);
}, 0); // 先进 task queue，等待前一个任务执行完成（单线程），立即执行，不需要读秒后再放入event queue

console.log(3);

for (let i = 0; i < 10000; i++) {
  console.log(1);
}

setTimeout(() => {
  console.log(4);
}, 100); // 最后放入task queue
// 3, 10000 * 1, 2, 1, 4
```

有时delay时间并不准确

```js
setTimeout(() => {
  console.log(1);
  setTimeout(() => {
    console.log(2);
  }, 99);
}, 100);

setTimeout(() => {
  console.log(3);
}, 200);

console.log(4);
```

> 结果就不确定了，取决于同步代码的计算时间，所以定时器这个东西还是比较坑的，delay 并不准确，取决于你的 JS 线程是否闲置以及执行效率。

## 宏任务/微任务

> ⌛js异步有一个机制，就是遇到宏任务，先执行宏任务，将宏任务放入eventqueue，然后在执行微任务，将微任务放入eventqueue最骚的是，这两个queue不是一个queue。当你往外拿的时候先从微任务里拿这个回掉函数，然后再从宏任务的queue上拿宏任务的回掉函数🔥

- 宏任务一般是：包括整体代码script，setTimeout，setInterval、setImmediate。
- 微任务：原生Promise(有些实现的promise将then方法放到了宏任务中)、process.nextTick、Object.observe(已废弃)、 MutationObserver记住就行了。

```js
// 1. 加入 tasks 队列
setTimeout(()=>{
  // 7. 首次 eventloop 结束，从 tasks 中取出 setTimeout callback，执行。
  console.log('timer');
  // 8. 加入 microtask 中
  Promise.resolve().then(function() {
    console.log('promise1')
  })
  // 9. task 执行完，清空 micro 队列，输出 'promise1'
}, 0);

// 2. 加入 microtask 队列
Promise.resolve().then(function() {// 5. 第一个 microtask 任务
  console.log('promise2');
  // 6. 把 promise3 加入 micro 队列，发现队列不为空，执行输出 'promise3'
  Promise.resolve().then(function() {
    console.log('promise3');
  })
})

// 3. 执行，输出 'script'
console.log('script');
// 4. 第一个 eventloop task 阶段完毕, 开始执行 microtask queue

// result: script, promise2, promise3, timer, promise1
```

```js
Promise.resolve().then(()=>{
  console.log('Promise1')  // 3、先执行微任务，清空micro-task
  setTimeout(()=>{ // 4、宏任务 callback 放入 event queue
    console.log('setTimeout2') // 9执行
  },0)
})

setTimeout(()=>{ // 2 宏任务 callback 放入 event queue
  console.log('setTimeout1') // 5、执行callback
  new Promise((resolve) => {
    console.log('promise start'); // 6、执行同步任务
    resolve();
  }).then(()=>{// 7、微任务
    console.log('Promise2') ///8、执行微任务
  })
},0)

console.log('start'); // 1、同步任务

```

## Node

> Node环境下异步执行顺序与浏览器下js执行顺序不完全一致

node新加了一个微任务(process.nextTick)和一个宏任务(setImmediate)

简单的来说，就是**node在处理一个执行队列的时候不管怎样都会先执行完当前队列，然后再清空微任务队列，再去执行下一个队列**。

```js
console.log('start');
setTimeout(function() {
  console.log(2);
  new Promise((resolve) => {
    console.log('promise');
    resolve();
  }).then(() => {
    console.log('promise then');
  })
});

setTimeout(() => {
  console.log(3);
})

console.log('end');

// node: start, end, 2, promise, 3, promise then
// 浏览器js: start, end, 2, promise, promise then, 3
```

> 🔥 微任务中process.nextTick比promise.then快

## 面试

```js
console.log('1');

setTimeout(function() {
    console.log('2');
    process.nextTick(function() {
        console.log('3');
    })
    new Promise(function(resolve) {
        console.log('4');
        resolve();
    }).then(function() {
        console.log('5')
    })
})
process.nextTick(function() {
    console.log('6');
})
new Promise(function(resolve) {
    console.log('7');
    resolve();
}).then(function() {
    console.log('8')
})

setTimeout(function() {
    console.log('9');
    process.nextTick(function() {
        console.log('10');
    })
    new Promise(function(resolve) {
        console.log('11');
        resolve();
    }).then(function() {
        console.log('12')
    })
})

// 1，7，6，8，2，4，3，5，9，11，10，12
// node环境下的事件监听依赖libuv驱动I/O库与前端环境不完全相同，输出顺序可能会有误差
```

## 参考

- [https://juejin.im/post/5b498d245188251b193d4059](https://juejin.im/post/5b498d245188251b193d4059)
- [https://juejin.im/post/5ac715b8f265da23986780fe](https://juejin.im/post/5ac715b8f265da23986780fe)
- [https://github.com/sunyongjian/blog/issues/38](https://github.com/sunyongjian/blog/issues/38)
