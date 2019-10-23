### 前言

我们通常说：

> JavaScript是一门单线程、异步、非阻塞、解释型的脚本语言。

那么什么是单线程，什么是异步，什么又是非阻塞呢？这篇文章会进行逐一解释，并以此引出Event Loop的概念。



### 进程和线程

网络上很多解释类似于”进程是CPU资源分配的最小单位，线程是CPU调度的最小单位“未免过于抽象，我们可以简单的比喻为：

- 工厂的每一个车间都是一个进程
- 车间中的每一个工人都是一个线程
- 一个车间可以拥有多个工人共同完成任务（一个进程可以有多个线程）

浏览器中的进程和线程又是怎样的呢？

以`Chrome`为例，每一个Tab标签就是一个进程，当我们打开多个页面时，查看任务管理器会发现有多个Chrome的进程，所以Chrome是多进程的。

![](https://mexion.xyz/images/study-notes/JavaScript-Event-Loop-in-the-browser/multi-progress.jpg)

Chrome还有其他的进程，但对于前端来说，主要需要了解的就是渲染进程。渲染进程是多线程的，一个渲染进程包含了以下线程：

- `GUI渲染线程`

- - 负责渲染页面，布局和绘制
  - 页面需要重绘和回流时，该线程就会执行
  - 与js引擎线程互斥，防止渲染结果不可预期

- `JS引擎线程`

- - 负责处理解析和执行Javascript脚本程序
  - 只有一个JS引擎线程（单线程）
  - 与GUI渲染线程互斥，防止渲染结果不可预期

- `事件触发线程`

- - 用来控制事件循环（鼠标点击、setTimeout、ajax等）
  - 当事件满足触发条件时，将事件放入到JS引擎所在的执行队列中

- `定时触发器线程`

- - setInterval与setTimeout所在的线程
  - 定时任务并不是由JS引擎计时的，是由定时触发线程来计时的
  - 计时完毕后，通知事件触发线程

- `异步http请求线程`

- - 浏览器有一个单独的线程用于处理AJAX请求
  - 当请求完成时，若有回调函数，通知事件触发线程

看完上面的描述后，有两点需要注意的：

- **我们说JavaScript是单线程的指的是JavaScript引擎只有一个线程**
- **JS线程和GUI渲染线程是互斥的**

#### 为什么JavaScript要设计成单线程？

JavaScript之所以设计成单线程是有很重要的原因的，因为JavaScript设计是在浏览器中运行的，我们需要在浏览器中对DOM进行操作，假如JavaScript设计成多线程的，当多个线程同时对一个DOM进行操作时，会导致意想不到的错误发生，比如一个线程向某个DOM添加事件，而另外一个线程删除了这个DOM。为了避免这样的事情发生，所以JavaScript选择了只使用一个线程来执行代码。

#### 为什么渲染线程和JS引擎线程是互斥的

这是因为在修改DOM时如果同时渲染界面，可能会导致渲染线程前后获得的元素不一致。

因此，两个线程之间是互斥的，当JS引擎线程执行时，GUI渲染线程会被挂起，渲染线程会等待JS引擎线程空闲时执行。



### JavaScript是异步非阻塞的

什么是阻塞？什么是异步？

JavaScript是单线程的，那么对于任务的处理自然是逐件处理的。就像银行柜台排队，假如只有一个柜台，工作人员会根据队列逐一办理业务，假如前面有一个人因为某些原因半个小时都没结束，这时候整个队列就阻塞了半个小时。

那么如果JavaScript没有特殊的处理方式，假如一个任务需要耗时很久，后面的所有任务都会被阻塞，比如一个网络请求还未结束，这时候用户的其他操作会一直等待这个网络请求结束，这样显然是行不通的。

JavaScript的做法是在遇到对应的任务时，会将这个任务交由对应的线程进行处理（比如网络请求线程处理网络请求、定时器线程处理定时器），在达到触发条件之时这些线程会将对应的回调放入任务队列之中。

这样就使JavaScript引擎是单线程的同时，通过异步的方式实现了代码的非阻塞。



### 浏览器中的Event Loop

Event Loop翻译过来就是事件循环，什么是事件循环？

先理解一些概念：

- JS 分为同步任务和异步任务
- 同步任务都在JS引擎线程上执行，形成一个 `执行栈`
- 事件触发线程管理一个 `任务队列`，异步任务触发条件达成，将回调事件放到 `任务队列`中
- `执行栈`中所有同步任务执行完毕，此时JS引擎线程空闲，系统会读取 `任务队列`，将可运行的异步任务回调添加到 `执行栈`中，开始执行

- JS引擎线程只执行执行栈中的事件
- 执行栈中的代码执行完毕，就会读取事件队列中的事件
- 事件队列中的回调事件，是由各自线程插入到事件队列中的

执行栈的执行规则为：

```javascript
执行栈执行 -> 将异步任务交由对应线程处理 -> 对应线程在达成条件后将回调放入任务队列 -> 执行栈执行完毕 -> 读取任务队列 -> 回到第一步（执行栈执行） 如此循环
```

#### 宏任务、微任务

任务队列中的任务分为宏任务（macro-task）和微任务（micro-task）。

- 宏任务：主代码块、setTimeout、setInterval、I/O等
- 微任务：Promise.then、process.nextTick等

宏任务和微任务的执行顺序是：先执行宏任务，然后将当前微任务队列中的所有微任务逐个执行，当执行完毕之后会执行下一个宏任务，然后执行下一个宏任务，接着再清空微任务队列，如此循环。

由于`JS引擎线程`和 `GUI渲染线程`是互斥的关系，浏览器为了能够使 `宏任务`和 `DOM任务`有序的进行，会在一个 `宏任务`执行结果后，在下一个 `宏任务`执行前， `GUI渲染线程`开始工作，对页面进行渲染。
所以实际的执行顺序为：
```javascript
宏任务 -> 清空当前微任务队列 -> 渲染 -> 宏任务 -> 微任务 -> 渲染......
```

接下来看实际的代码：

```javascript
setTimeout(() => { console.log(1) }, 3000)

new Promise((resolve,reject) =>{
	console.log(2);
	resolve();
}).then(() => {console.log(3)}).then(() => {console.log(4) })

setTimeout(() =>{ console.log(5) },2000)

new Promise((resolve,reject) => {
	resolve()
}).then(() => {console.log(6)})

// 输出结果为: 2 -> 3 -> 6 -> 4 -> 5 -> 1
```

分析为何这样输出：

- 首先运行到`setTimeout(() => { console.log(1) }, 3000)`，这是一个宏任务，这时会将这个任务挂起，交由定时触发器线程处理，定时触发器线程会在3000ms后将这个回调放入宏任务队列中

- 然后运行`new Promise((resolve,reject) =>{ console.log(2); resolve(); })`，这是一个同步任务，所以会直接输出2，然后调用了then方法，将`then(() => {console.log(3)})`放入微任务队列，需要注意的是Promise本身不进入微任务队列，它的then方法才进入队列

- 接下来是`setTimeout(() =>{ console.log(5) },2000)`，这是一个宏任务，这时会将这个任务挂起，交由定时触发器线程处理，定时触发器线程会在2000ms后将这个回调放入宏任务队列中

- 然后是`new Promise((resolve,reject) => {resolve()})`，调用then方法后将`then(() => {console.log(6)})`放入到微任务队列中

- 此时所有主代码执行完毕，已经输出了2,宏任务队列`[console.log(5), console.log(1)]`，微任务队列`[then(() =>{ console.log(3) }), then(() => { console.log(6) })]`

- 主代码是宏任务，所以接下来会清空当前的微任务队列，运行`then(() => { console.log(3) })`，输出3，这时再次调用了then方法，将`then(() => {console.log(4) })`放入微任务队列，此时的微任务队列`[then(() => { console.log(6) })， then(() => { console.log(4) })]`

- 微任务只执行了一个，要清空，所以执行剩余的微任务，输出6,4

- 微任务已经清空，接下来执行宏任务，输出5

- 没有微任务了，执行身下的宏任务，输出1

  

```javascript
setTimeout(() => { console.log(1) }, 1000)

new Promise((resolve,reject) =>{
	console.log(2);
	resolve();
}).then(() => {console.log(3)}).then(() => {console.log(4) })

setTimeout(() =>{ 
Promise.resolve(7).then(data => { console.log(data) })
console.log(5) },2000)

new Promise((resolve,reject) => {
	resolve()
}).then(() => {console.log(6)})

// 输出结果： 2 -> 3 -> 6 -> 4 -> 1 -> 5 -> 7
```

这段前面的执行结果相同，后面由于`setTimeout(() => { console.log(1) }, 1000)`改成了1000ms，所以会被先放入宏任务队列。



```javascript
setTimeout(() => { console.log(1) }, 3000)

new Promise((resolve,reject) =>{
	console.log(2);
	resolve();
}).then(() => {console.log(3)}).then(() => {console.log(4) })

setTimeout(() =>{ 
Promise.resolve(7).then(data => { console.log(data) });
console.log(5); },2000)

new Promise((resolve,reject) => {
	resolve()
}).then(() => {console.log(6)})

// 输出结果： 2 -> 3 -> 6 -> 4 -> 5 -> 7 -> 1
```

这段由于`setTimeout(() => { console.log(1) }, 3000)`3000ms后才会被放入宏任务队列，所以后执行，在执行`setTimeout(() =>{ Promise.resolve(7).then(data => { console.log(data) });console.log(5); },2000)`时输出5，由于调用了then方法，所以会将`then(data => { console.log(data) })`放入微任务队列，在执行完这个宏任务之后会清空微任务队列，所以输出7，最后执行下一个宏任务，输出1。

