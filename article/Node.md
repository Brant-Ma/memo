## Node
> 借助 Node 强化一下对前端的理解

### 1. 简介

#### 1.1 Node 与 JavaScript
Node 是 JavaScript 在后端的宿主环境，采用的运行时是 V8 引擎。Node 的出现让 JavaScript 穿越 HTTP 协议栈的次元壁，实现资源访问，而不再局限于 UI 构建。Node 选择 JavaScript 作为后端编程语言，主要基于三个原因：

- JavaScript 引擎 V8 的高性能
- JavaScript 符合事件驱动
- JavaScript 在后端没有历史包袱

另外，更重要的一点原因是 JavaScript 的普及具有开发优势，而且能够实现前后端的语言统一。

#### 1.2 Node 的特点与场景
Node 的特点实际上是前端领域的重要特点在后端领域的保持：

- 底层 API 的构建基于异步 I/O，异步方法的调用借助事件抽象和回调函数。需要考虑异步流程控制和事件间的协作
- 单线程。需要考虑在无法利用多核 CPU 的情况下解决大量计算，以及如何避免单个错误使整个应用崩溃的问题
- 跨平台。Node 通过在模块系统和操作系统之前构建平台层 libuv 解决了跨平台问题

基于 Node 的并行 I/O 特点，不难得出其更适合 I/O 密集型业务。但得益于 V8 引擎的高性能，Node 在 CPU 密集型业务中也发挥得不错。

### 2. 模块

#### 2.1 后端的模块引入机制
根据 CommonJS 规范，模块的引用由 `require()` 方法完成，其参数代表模块的标识。模块的定义由 `exports` 属性完成，该属性是 `module` 对象的属性，且自身也是个对象。 

根据模块标识的不同，模块分为核心模块（如 `http` 和 `fs`），文件模块（以 `.` 或 `..` 开始的相对路径，以 `/` 开始的绝对路径，或者非路径形式）。前者由 Node 提供，后者由用户编写。

抛开优先尝试缓存加载，模块引入的实际步骤分为三步：

- 路径分析：逐个尝试模块路径（`module.paths`）数组的路径，直到找到目标文件。具体顺序类似作用域链，从 `./node_modules` 直至 `/node_modules`
- 文件定位：在某个模块路径下，如果模块标识不含扩展名时，会依次尝试 `.js`、`.json`、`.node` 扩展名。如果失败，就尝试该路径中 `package.json` 的 `main` 属性指定的文件；如果再次失败，就默认使用 `index` 为文件名；如果依然失败，就放弃该路径，尝试下一个模块路径
- 编译执行：`.js` 文件和 `.json` 文件会通过 `fs` 模块同步读取内容，然后经过处理将模块的 `exports` 属性返回给调用方使用

其中，编译执行的处理过程，`.json` 文件极为简单：通过 `JSON.parse()` 把读取到的内容转为对象，然后赋值给 `module.exports` 属性。而 `.js` 文件则会被包装进函数，以实现作用域隔离：

	// hi.js 文件的包装
	(function(exports, require, module, __filename, __dirname) {
	  // hi.js 文件的内容
	});

Node 进程启动时，会直接把核心模块载入内存，文件定位和编译执行的步骤会省略，因此核心模块的加载速度很快。另外，除了作为纯粹的功能模块，核心模块也经常被文件模块调用。

在模块的基础上可以形成包。二者的侧重点不同：模块（module）是可以被 `require()` 的文件或目录，用于解决代码的组织性；包（package）是被 `package.json` 描述的文件或目录，用于解决模块的组织性。

#### 2.2 前端的模块引入机制
CommonJS 规范并不适合前端环境下的模块。后端从硬盘中加载代码，而前端是通过网络加载代码，带宽瓶颈决定了加载速度会很慢。Node 的模块引入是同步的，如果浏览器也采用同步的方式，脚本的初始化会消耗大量的时间。但是采用异步的话，当前模块所依赖的模块就需要提前加载。针对依赖的书写位置和加载时机，有着不同的实现方式：

- 基于 AMD 规范的实现（如RequireJS）采取前置书写，提前执行的策略，让同一模块的所有依赖行为统一
- 基于 CMD 规范的实现（如SeaJS）采取就近书写，延迟执行的策略，让同一模块的所有依赖行为独立

两种方式借助不同的策略，将依赖以对象的形式异步的载入模块。相比前端早期只有全局变量的模块风格，实现了质的飞跃。但是这仍然不是最优情况：AMD 或 CMD 均属于运行时的模块引入机制，时机太晚了，避开了很多优化的可能。

ES6 Module 最后完成了收割。它有如下优势：

- 它是官方指定的模块化规范，更有发展前景
- 它的模块引入是在编译时完成，输出为值的引用，而非值的拷贝
- 相比 `require` 和 `exports` 关键字，`import` 和 `export` 显然更优雅，也更清爽

另外，采用 webpack 或者 browserify 等打包工具，可以将模块的引入提前至构建时。

### 3. 异步

#### 3.1 什么是异步

阻塞和同步是容易混淆的概念，非阻塞和异步也是如此。所以先来解释两组概念：

- 阻塞/非阻塞：描述调用者在等待调用结果时的状态，如果可以在等待时执行其他事务就是非阻塞
- 同步/异步：描述被调用者在处理完成后的通信方式，如果可以在完成后主动通知调用者就是异步

前后端编程涉及很多 I/O 操作（如用户输入、网络请求、数据库操作、文件读写）。从前端的用户体验视角，由于 JS 解释引擎和 UI 渲染引擎共用一个线程，I/O 操作将会阻塞 UI。

从后端的资源分配视角，如果单线程串行执行任务会阻塞 I/O 导致资源利用不足，而如果多线程并行执行任务又会带来线程创建和切换成本、锁和状态同步问题。

Node 给出了单线程异步 I/O 的折中方案，兼顾了资源利用和开发成本。那么前端是如何解决异步的？其实异步的解决方案是两个层面的事情：一个是宿主提供的机制，一个是编程所需的方案。

#### 3.2 宿主机制

完整的异步机制包含了脚本引擎的 call stack，浏览器宿主的 web APIs，event loop 及其 callback queue，而 event loop 在其中起到了纽带的作用，衔接了关键的环节。

浏览器的每个线程（Browsing Context 或 Web Worker）都是通过 event loop 机制来实现异步的。当 call stack 为空时，event loop 开始正式工作，即不停的循环。每个循环被称为一个 tick，每个 tick 的大致工作流程如下：

1. 上个 tick 完毕，开始本次 tick
2. call stack 已空
3. event loop 从任意的 macroTask queue 中取出一个任务，丢给 call stack 执行
4. call stack 已空
5. event loop 从唯一的 microTask queue 中逐个取出任务，丢给 call stack 执行
6. call stack 已空
7. 更新 view
8. 本次 tick 完毕，进入下个 tick

可以看出，宏任务和微任务主要有三个区别：宏任务在 tick 内的优先级更高；微任务的整个队列将在 tick 内清空；每个 event loop 中，macroTask queue 可以有多个，而 microTask 只会有一个。

有个重要细节需要注意：web APIs 会为自己的回调安排（schedule）一个任务，并放进 callback queue 中，但任务来源有哪些？任务类型又有哪些？任务应该被放进哪种及哪个 queue 中？

- 任务来源：DOM 操作（DOM manipulate），用户交互（user interaction），网络（networking），历史遍历（history traversal）
- 任务类型：调度事件（dispatching event），解析文档（parsing HTML），执行回调（running callback），使用资源（using resource），响应 DOM 操作（reacting to DOM manipulate）

关于任务应该被放进哪个 queue 中，可以看下来自 [Jake Archibald](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/) 的示例：

	var outer = document.querySelector('.outer');
	var inner = document.querySelector('.inner');
	
	new MutationObserver(() => console.log('mutate'))
	  .observe(outer, { attributes: true });
	
	function onClick() {
	  console.log('click');
	
	  setTimeout(() => console.log('timeout'), 0);
	
	  Promise.resolve().then(() => console.log('promise'));
	
	  outer.setAttribute('data-random', Math.random());
	}
	
	inner.addEventListener('click', onClick);
	outer.addEventListener('click', onClick);
	
	inner.click();

最后输出的 log 依次为 `click` `click` `promise` `mutate` `promise` `timeout` `timeout`，由此可知：macroTask 包括 timer 回调，I/O 回调等，会被安排进下次 tick；microTask 包括 mutation observer 回调，promise 回调等，会被安排进本次 tick。

另外，event loop 中的“事件”指的是系统层面产生的，而非代码层面产生的。前者才是异步，后者只是同步。前者比如来自用户交互触发的真实点击；后者比如来自 DOM API 触发的虚拟点击。

#### 3.3 编程方案

异步编程带来的困难包括但不限于：异常处理，回调地狱，线程挂起，工作线程，流程理解...

异常处理：异步方法在提交请求和处理请求之间，包括了 event loop 的调度。传统的 `try/catch` 只能捕获异步方法自身的异常，无法捕获异步方法的回调异常。

回调地狱：相互依赖的异步方法会存在嵌套，最终导致整个代码书写深不见底。

线程挂起：前端脚本天然具备线程阻塞的特点（比如滥用事件，或者运算量极大的同步代码），但线程挂起却很难实现，除非使用 `alert()` `confirm()` `prompt()` 模态弹框。

工作线程：尽管 UI 渲染和 JS 执行 共用同一个线程具有一定的必要性（提高脚本引发的那部分渲染的效率），但如果把脚本执行独立出来，将提高 CPU 的利用率、减少对 UI 渲染的阻塞。

流程理解：异步代码和同步代码的混合书写会带来一定的理解成本。

尽管我们更擅长同步，但异步才更接近现实。可以优化这些困难的模式或方案有 publish/subscribe 模式、promise/deferred 模式，以及一些流程控制库（尾触发、async、Step、EventProxy、wind 等等）。

### 4. 其他

Node 剩下的内容与前端的关系相对松散，简单总结如下。

#### 4.1 内存

Chrome 的每个标签页使用一个 V8 实例，JavaScript 中对象的内存被分配在 V8 的堆内存里。V8 将内存空间分为新生代和老生代，前者存储存活时间较短的对象，后者存储存活时间较长或者常驻内存的对象。整个堆内存的容量是有限制的，并且堆内存被 V8 的垃圾回收机制所管理。

内存的分配与释放与作用域有关，标识符查找的过程是沿着作用域链进行的。除了全局作用域，函数调用也会创建作用域，但会在执行完毕后销毁。作用域释放后，内部的局部变量所引用的对象会在下次垃圾回收时被释放。

全局变量是常驻内存的，需要被主动释放。方法有两种：通过 `delete` 删除变量与对象的引用关系，或者通过给变量重新赋值来脱离与旧对象的引用关系。

通过作用域链，内部作用域可以访问到外部作用域的变量。但借助高阶函数的特性（函数可以作为参数或返回值）可以实现外部作用域访问内部作用域的变量，即闭包。作为返回值的中间函数具有引用其词法作用域中的局部变量的能力，如果这个中间函数被一个变量所引用，就会产生一系列的内存占用：变量持有中间函数，中间函数持有原始作用域，原始作用域持有内存。除非解除该变量对中间函数的引用，否则内存无法释放。

综上所述，全局变量的引用和闭包，是两种导致内存无法被立即收回的可能。在实际情况中，内存回收可能会出现特殊的问题，使一些对象成为了常驻内存中的对象，即发生了内存泄漏。内存泄漏的原因有作用域未释放、将内存当为缓存、队列消费不及时。

#### 4.2 Buffer 与 Stream

V8 主要负责的堆内存管理通常是字符串层面的，即以 char 为单位的字符流。而以 bit 为单位的字节流内存可以不受 V8 的内存限制。

Buffer 是一种类似数组的对象，可以存储字节流。特点是先将数据读取到内存，再在内存中处理数据。Stream 是一种对 Buffer 的封装，根据实现分为网络流、文件流等。特点是一边读取数据，一边处理数据。

可以看出，Stream 既节省了空间（内存只占有当前要处理的数据）又节省了时间（不必等待所有数据都读取完毕），这极大的方便了管道流的链式操作。

	var reader = fs.createReadStream('in.txt');
	var writer = fs.createWriteStream('out.txt');
	reader.pipe(writer);

例子是文件流的用法：读取 `in.txt` 文件中的数据，并将数据写入 `out.txt` 文件中。

#### 4.3 HTTP 服务 与 Web 应用

HTTP 服务负责处理 HTTP 请求和发送 HTTP 响应。此外，还可以通过 WebSocket 服务建立长连接，或者通过 HTTPS 服务建立安全连接。

Web 应用除了涉及对请求报文头部的处理，还包括了许多复杂的功能，比如数据上传、路由解析和页面渲染。

### 参考
1. [深入浅出 Node.JS - 朴灵 | 豆瓣](https://book.douban.com/subject/25768396/)
2. [require，import区别？- 寸志 | 知乎](https://www.zhihu.com/question/56820346/answer/150724784)
3. [What the heck is the event loop anyway? - Philip Roberts | YouTube](https://www.youtube.com/watch?v=8aGhZQkoFbQ)
4. [解读 JavaScript 异步函数 - abbshr | GitHub](https://github.com/abbshr/abbshr.github.io/issues/50)
