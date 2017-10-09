# js 异步编程

> 引言：现实生活中，事情是可以一件件按顺序干，也可以一边一边共同干，这就是编程执行环境中的单线程、多线程。

## 参考

- [JavaScript 异步编程学习笔记](http://www.jianshu.com/p/6d4d2e21d041)
- [JS异步编程的解决方案](https://lenshen.com/2016/12/22/js-async-coding/)
- [JavaScript 异步编程魔法](http://fsux.me/js/%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BA/2016/03/24/Asynchronous-programming-magic.html)

## 了解

Javascript语言的执行环境是”单线程”（single thread）。

**所谓”单线程”**：就是指一次只能完成一件任务。如果有多个任务，就必须排队，前面一个任务完成，再执行后面一个任务，以此类推。这种模式的好处是实现起来比较简单，执行环境相对单纯；坏处是只要有一个任务耗时很长，后面的任务都必须排队等着，会拖延整个程序的执行。常见的浏览器无响应（假死），往往就是因为某一段Javascript代码长时间运行（比如死循环），导致整个页面卡在这个地方，其他任务无法执行。为了解决这个问题，Javascript语言将任务的执行模式分成两种：**同步（Synchronous）和异步（Asynchronous）**。

**“同步模式”** ：就是上一段的模式，后一个任务等待前一个任务结束，然后再执行，程序的执行顺序与任务的排列顺序是一致的、同步的；”异步模式”则完全不同，每一个任务有一个或多个回调函数（callback），前一个任务结束后，不是执行后一个任务，而是执行回调函数，后一个任务则是不等前一个任务结束就执行，所以程序的执行顺序与任务的排列顺序是不一致的、异步的。

**“异步模式”**：非常重要。在浏览器端，耗时很长的操作都应该异步执行，避免浏览器失去响应，最好的例子就是Ajax操作。在服务器端，”异步模式”甚至是唯一的模式，因为执行环境是单线程的，如果允许同步执行所有http请求，服务器性能会急剧下降，很快就会失去响应。
 
## js 运行机制

![](https://raw.githubusercontent.com/NARUTOne/resources-github/master/imgs/asay/event-loop.png)

上图中，主线程运行的时候，产生堆（heap）和栈（stack），栈中的代码调用各种外部API，它们在"任务队列"中加入各种事件（click，load，done）。只要栈中的代码执行完毕，主线程就会去读取"任务队列"，依次执行那些事件所对应的回调函数。

## 异步编程方式

### 回调函数 callback

```
var fn = function(callback) {
    // do something here
    ...
    setTimeout(function() {
      callback.apply(this, this.arguments);
    }, 300)
}
 
var mycallback = function(parameter) {
  // do something
}
 
fn(mycallback);
```

> 优点是简单、容易理解和部署，缺点是不利于代码的阅读和维护，各个部分之间高度耦合（Coupling），流程会很混乱，而且每个任务只能指定一个回调函数。

回调函数多次回调后，代码可读性严重受阻。

**回调地域**

```
f1(function(err, result) {
    f2(function(err, result) {
        f3(function(err, result) {
            f4(function(err, result) {
                f5(function(err, result) {
                    // do something useful
                })
            })
        })
    })
})
```

### setTimeout 延时异步
> setTimeout 异步编程 也是会有些问题的，详情见下面：
[You Don't Know setTimeout](https://mechanicianw.github.io/2017/06/04/from-settimeout-to-event-loop/)

```
setTimeout(function(){
    console.log("this will be exectued after 1 second!");
},1000);

var start = new Date()
setTimeout(function(){
  var end = new Date()

  console.log(end - start)
},500)

while(new Date - start<1000){

}

```

因为 while 循环会持续一秒，线程繁忙，结果是大于等于 1000 的数字，可能会稍有不同，这是因为 setTimeout 很不精准。不过，这个数字至少是1000，因为setTimeout回调在while循环结束之前不可能被触发。

之所以会这样，全是因为 **任务队列** 的存在。

**任务队列**
- 同步任务：在主线程中排队依次执行的任务
- 异步任务：不进入主线程，进入任务队列的任务。只有任务队列通知主线程某异步任务可以执行，该任务才会进入主线程执行。

> 异步任务通常可以分为两大类：I/O 函数（AJAX、readFile等）和计时函数（setTimeout、setInterval）

>其实所谓的『回调函数』，就是那些被主线程挂起来的代码。异步任务必须指定回调函数，当主线程开始执行异步任务，就是执行对应的回调函数。

### 类事件监听(jQuery)

> 采用 jquery $.on() $.trigger()

```
function f1(){
　setTimeout(function () {
　// f1的任务代码
　　f1.trigger('done');   //触发事件
　}, 1000);
}

f1.on('done', mycallback);
```
这种方法的优点是比较容易理解，可以绑定多个事件，每个事件可以指定多个回调函数，JS 和 浏览器提供的原生方法基本都是基于事件触发机制的，耦合度很低，不过事件不能得到流程控制。

### 事件监听升级——设计模式  发布+订阅

[jQuery.subscribe](http://wiki.jikexueyuan.com/project/javascript-design-patterns/observer-model-pattern.html)

```
function f2() {
  console.log('hello world!')
}

jQuery.subscribe("done", f2);

function f1(){
　setTimeout(function () {
　// f1的任务代码
　　jQuery.publish("done");
　}, 1000);
}
 
jQuery.unsubscribe("done", f2);
```

这种方法的性质与”事件监听”类似，但是明显优于后者。因为我们可以通过查看”消息中心”，了解存在多少信号、每个信号有多少订阅者，从而监控程序的运行。

### 高阶函数(泛函数)

```
function fn(str1) {
  return function(str2) {
    return str1 + str2;
  }
}

fn('hello')('world');
```

这种方法解耦程度很低，但是当参数过多时程序可读性非常不友好。地域回调

### Promise对象

[promise You-Dont-Know-JS](https://github.com/wz-fork/You-Dont-Know-JS/blob/1ed-zh-CN/async%20%26%20performance/ch3.md)

```
function timeout(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, ms, 'done');
  });
}

timeout(100).then((value) => {
  console.log(value);
});


var p1 = Promise.resolve( 42 );
var p2 = Promise.resolve( "Hello World" );
var p3 = Promise.reject( "Oops" );

Promise.race( [p1,p2,p3] )
.then( function(msg){
  console.log( msg );   // 42
} );

Promise.all( [p1,p2,p3] )
.catch( function(err){
  console.error( err ); // "Oops"
} );

Promise.all( [p1,p2] )
.then( function(msgs){
  console.log( msgs );  // [42,"Hello World"]
} );
```

简单说，它的思想是，每一个异步任务返回一个Promise对象，该对象有一个then方法，允许指定回调函数。比如，f1的回调函数f2,可以写成： f1().then(f2);

### Generator

> 执行权限暂停，权限转移

[Generator](http://es6.ruanyifeng.com/#docs/generator)

```
function* gen(x){
  try {
    var y = yield x + 2;
  } catch (e){ 
    console.log(e);
  }
  return y;
}
var g = gen(1);
g.next(); // 3
g.throw（'出错了'）;// 出错了

```

ES6 引入的 Generator 可以理解为可在运行中转移控制权给其他代码，并在需要的时候返回继续执行的函数；利用 Generator 可以实现协程的功能。

所谓的协程，意思是多个线程互相协作，完成异步任务。

### async/await
> ES7 提出的async 函数，终于让 JavaScript 对于异步操作有了终极解决方案。

[async](http://es6.ruanyifeng.com/#docs/async)

```
var sleep = function (time) {
    return new Promise(function (resolve, reject) {
        setTimeout(function () {
            resolve();
        }, time);
    })
};
var start = async function () {
    // 在这里使用起来就像同步代码那样直观
    console.log('start');
    await sleep(3000);
    console.log('end');
};
start()
```

### 异步类库

**promise 规范**

- [jquery deferred](http://javascript.ruanyifeng.com/jquery/deferred.html)
- [when.js 快速上手](https://imququ.com/post/promises-when-js.html)

**xhr**
- [fetch](https://github.com/camsong/blog/issues/2)
- [fetch api](https://github.com/github/fetch)

### script 标签

```
<html>
    <head>
        <script src="a.js"></script>
        <script defer src="b.js"></script>
    </head>
    <body>
        <!-- content -->
        <script async defer src="c.js"></script>
        <script async defer src="d.js"></script>
    </body>
</html>

```

如果同时使用 defer 和 async ，async 会覆盖掉 defer 。
