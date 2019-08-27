---
title: webWokrer
---

## 一、`webWokrer`介绍

javascript 是一种单线程语言，即所有任务只能在一个线程上完成，一次只能完成一个任务，如果前面的任务没有完成，后面的任务将无法执行。到目前为止，计算机的运算能力已经得到了极大的提升，单线程无法充分利用起计算机的运算能力。
 
web Worker 是HTML5 标准的一部分,这一规范定义了一套API，运行一段javascript 程序运行在主线程之外的另一个线程之中，子线程不会影响到主线程程序的运行，这样就能提高对计算机的运算能力的利用。

##### 1.1 web worker需要注意的几个小点

##### （1）同源策略
webWorker脚本必须和主线程的脚本同源

##### （2）全局对象限制
webWorker脚本所在的全局对象，与主线程不一致，无法使用：

`docement`、 `window`、 `parent`、 `DOM`对象，这样即代表着webWorker不能更新DOM节点；并且webWorker不能执行`alert()`方法和`confirm()`方法，

但是可以使用：

`navigator`、`location`（只读）、`XMLHttpRequest`（可以创建AJAX请求） 、`setTimeout/setInterval` 方法、`Application Cache`、通过 `importScripts()`方法加载其他脚本、创建新的 Web Worker

##### （3）通信限制
Worker线程和主线程不在同一个上下文环境，之间不能直接通信，只能通过消息事件进行通信，这里的传递只是进行值传递（拷贝传递），而不是进行引用地址传递，这种方法叫做结构化克隆传递数据（structured cloning）。

在进行二进制传递的时候，是使用可转让对象传递数据（transferable objects），即主线程把二进制数据直接转让给了子进程，此时主进程将无法使用此部分二进制数据，反过来亦如此。这是为了防止出现多个线程同时修改数据的问题。


##### （4）文件限制
Worker 线程无法读取本地文件，即不能打开本机的文件系统`（file://）`，它所加载的脚本，必须来自网络。

## 二、Worker的基础用法

### 2.1主线程
新建一个Worer线程
```
var worker = new Worker('work.js');
```
`Worker()`构造函数的参数是一个脚本文件，该文件就是 Worker 线程所要执行的任务。由于 Worker 不能读取本地文件，所以这个脚本必须来自网络。如果下载没有成功（比如404错误），Worker 就会默默地失败。

然后主线程和子线程进行通信，主线程通过调用`postMessage`方法向子线程传递数据，传递的数据可以为任何的数据类型。

ps: 数据传递方式分为转让所有权和不转让所有权，区别会在下面详细说明。
```
worker.postMessage('Hello World');
```

主线程通过`onmessage`指定监听函数，接受子线程发送回来的消息。
```
// 写法一
worker.onmessage= function(e){
    console.log('onmessage', e.data)
}
// 写法二
worker.addEventListener('message', function(e){
    console.log('addEventListener', e.data)
})
```

`message`事件必须挂在worker下；上面代码中的`data`是获取`Worker`发送的数据。当`Worker`完成任务后，通过`terminate`关闭
```
worker.terminate();
```
worker 线程会被立即杀死，不会有任何机会让它完成自己的操作或清理工作

### 2.2 Worker线程
说完了主线程，就该说一下Worker线程如何进行通信。
```
// work.js
// 接受数据
self.onmessage = function(e){ 
    console.log(e.data)
    self.postMessage('已拿到数据') // 给主线程发送数据
    postMessage()
}
onmessage = function(e){ 
    console.log(e.data)
}
self.addEventListener('message', function(e){
    console.log(e.data)
})

// 执行完计算后可以关闭自身线程
close()
slef.close()

```
上面的代码中，`self`代表子线程本事，为子线程的全局对象。但是，在子线程中`this`为`undefined`，所以`this.onmessage`是不允许的。

当执行完运算后，worker线程也可以通过调用`close`方法进行关闭。同样的，执行完`close`后，worker 线程会被立即杀死，`close`之后的运行将不会进行。
### 2.3 错误处理
主线程可以监听Worker是否发生错误。如果发生错误，Worker 会触发主线程的error事件。

```
// 主线程
worker.onerror = function (event) {
  event.preventDefault()
  console.log([
    'ERROR: Line ', event.lineno, ' in ', event.filename, ': ', event.message
  ].join(''));
};

worker.addEventListener('error', function (event) {
  // ...
});
```

该事件不会冒泡并且可以被取消；为了防止触发默认动作，worker 可以调用错误事件的 `preventDefault()`方法。
在子线程中无法触发`onerror`事件。 

`lineno`：发生错误时所在脚本文件的行号。

`filename`：发生错误的脚本文件名。

`message`：读性良好的错误消息。

### 2.4 subWorker
Worker 线程能够访问一个全局函数importScripts()来引入脚本，该函数接受0个或者多个URI作为参数来引入资源；以下例子都是合法的：
```
importScripts();                        /* 什么都不引入 */
importScripts('foo.js');                /* 只引入 "foo.js" */
importScripts('foo.js', 'bar.js');      /* 引入两个脚本 */
```
浏览器加载并运行每一个列出的脚本。每个脚本中的全局对象都能够被 worker 使用。如果脚本无法加载，将抛出 NETWORK_ERROR 异常，接下来的代码也无法执行。而之前执行的代码(包括使用 window.setTimeout() 异步执行的代码)依然能够运行。importScripts() 之后的函数声明依然会被保留，因为它们始终会在其他代码之前运行。
### 2.5 SharedWorker

## 三、Worker的数据通信
上面在2.1的时候说到，worker的数据传递分为两种形式，转让所有权和不转让所有权。

**不转让所有权**指的是主线程与worker通信的数据是通过深拷贝进行传值。 通常，主线程与worker进程通信的内容为文本或者对象，并且通常这种数据的大小不会大，所以这个时候通过拷贝的方式进行传值并不会太大影响。但是当数据量特别大的时候，就会影响到主线程的运行。

**转让所有权**是一种性能更好的方法，**可转让对象**（Transferable Objects）从一个上下文转移到另一个上下文而不会经过任何拷贝操作。例如，当你将一个 ArrayBuffer 对象从主应用转让到 Worker 中，原始的 ArrayBuffer被清除并且无法使用。它包含的内容会(完整无差的)传递给 Worker 上下文。
```
// 二进制
var abBuffer = new ArrayBuffer(32);
aDedicatedWorker.postMessage(abBuffer, [abBuffer]);

// image
var imageData = context.createImageData(img.width, img.height);
// 转化为类型数组进行传递
var int8s = new Int8Array(imageData.data);
var data = {
    data: int8s,
    width: imageData.width,
    height: imageData.height
};
// 在postMessage方法的第二个参数中指定transferList
work.postMessage(data, [data.data.buffer]);
```

ps:可转让对象支持的常用数据类型有`ArrayBuffer`和`ImageBitmap`

## 四、Worker的线程问题
`Javascript`引擎是单线程运行的，`Worker`能够让`Javscript`进行多线程进行运行，是否就代表着`Javascript`就变成的一个支持多线程的语言了呢？
### 4.1 进程和线程
- 进程是cpu资源分配的最小单位（是能拥有资源和独立运行的最小单位）
- 线程是cpu调度的最小单位（线程是建立在进程的基础上的一次程序运行单位，一个进程中可以有多个线程）
- 浏览器是多进程的，每个进程包含了多个线程

**为了方便理解，后续会忽略进程，只考虑线程，所以在某些细节上会存在一些不合理的地方**


### 4.2 worker的线程

`js`是运行在浏览器中的，浏览器是同时拥有多个线程的，比如：
```
javascript引擎线程
界面渲染线程（GUI渲染线程）
浏览器事件触发线程
Http请求线程
```
而`Web Worker`是浏览器提供的一个能力，通过相关API，`js`可以在浏览器中新开一个线程去运行`js`代码，最后的线程关系如下:
```
javascript引擎线程（主线程）
界面渲染线程
浏览器事件触发线程
Http请求线程
Worker线程
```
所以，`Web Worker`并不代表着`js`这个语言就支持了多线程。

**WebWorker和SharedWorker的线程不一样，区别在于WebWorker线程只属于单个页面的线程，不会与其他页面共享；而SharedWorker的线程则是多个页面共享。这里不做更详细的讨论**

### 4.3 worker异步和js异步有什么区别
`JavaScript`中耗时的I/O操作都被处理为异步操作，它们包括键盘、鼠标I/O输入输出事件、窗口大小的`resize`事件、定时器(`setTimeout`、`setInterval`)事件、Ajax请求网络I/O回调等。当这些异步任务发生的时候，它们将会被放入浏览器的事件任务队列中去，等到`JavaScript`运行时执行线程空闲时候才会按照队列先进先出的原则被一一执行，但终究还是单线程，最后的处理还是放在了**javascript引擎线程**，假如异步处理中有复杂运算的时候，就很容易发生卡顿的情况。

但是，`web Worker`是通过创建一个新的线程进行运行`js``，两个进程之间只有在数据交换的时候才会进行通信。所以即使相关运算很复杂，也不会影响到主线程中的代码运行。

### 4.4 连续创建多个Worker线程
如果在主线程中连续创建多个worker线程，这些worker线程在创建后就开始运行，不会互相影响，复杂运算也不用等待先创建的线程，关系如下
```
javascript引擎线程（主线程）
界面渲染线程
浏览器事件触发线程
Http请求线程
Worker1线程
Worker2线程
Worker3线程
...
```
在多个线程创建后，每个线程的运算都会同时进行
### 4.4 创建Worker线程有上限么
在MDN中，并没有发现针对创建Worker上限的说法，本人也在google浏览器上实验过，也并尝试出浏览器针对同时创建多个worker线程的限制。所以猜测Worker的线程是依赖于运行的cpu能力。

## 五、嵌入式worker
目前没有一种「官方」的方法能够像 <script> 元素一样将 worker 的代码嵌入的网页中。但是如果一个 <script> 元素没有 src 特性，并且它的 type 特性没有指定成一个可运行的 mime-type，那么它就会被认为是一个数据块元素，并且能够被 JavaScript 使用。「数据块」是 HTML5 中一个十分常见的特性，它可以携带几乎任何文本类型的数据。所以，你能够以如下方式嵌入一个 worker：
```
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8" />
<title>MDN Example - Embedded worker</title>
<script type="text/js-worker">
  // 该脚本不会被 JS 引擎解析，因为它的 mime-type 是 text/js-worker。
  var myVar = "Hello World!";
  // 剩下的 worker 代码写到这里。
</script>
<script type="text/javascript">
  // 该脚本会被 JS 引擎解析，因为它的 mime-type 是 text/javascript。
  function pageLog (sMsg) {
    // 使用 fragment：这样浏览器只会进行一次渲染/重排。
    var oFragm = document.createDocumentFragment();
    oFragm.appendChild(document.createTextNode(sMsg));
    oFragm.appendChild(document.createElement("br"));
    document.querySelector("#logDisplay").appendChild(oFragm);
  }
</script>
<script type="text/js-worker">
  // 该脚本不会被 JS 引擎解析，因为它的 mime-type 是 text/js-worker。
  onmessage = function (oEvent) {
    postMessage(myVar);
  };
  // 剩下的 worker 代码写到这里。
</script>
<script type="text/javascript">
  // 该脚本会被 JS 引擎解析，因为它的 mime-type 是 text/javascript。

  // 在过去...：
  // 我们使用 blob builder
  // ...但是现在我们使用 Blob...:
  var blob = new Blob(Array.prototype.map.call(document.querySelectorAll("script[type=\"text\/js-worker\"]"), function (oScript) { return oScript.textContent; }),{type: "text/javascript"});

  // 创建一个新的 document.worker 属性，包含所有 "text/js-worker" 脚本。
  document.worker = new Worker(window.URL.createObjectURL(blob));

  document.worker.onmessage = function (oEvent) {
    pageLog("Received: " + oEvent.data);
  };

  // 启动 worker.
  window.onload = function() { document.worker.postMessage(""); };
</script>
</head>
<body><div id="logDisplay"></div></body>
</html>
```

## 六、webpack中使用worker





