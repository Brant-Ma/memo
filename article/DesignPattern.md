## Design Pattern
> 学会它，忘掉它

### 1. 什么是设计模式

设计模式是 GoF 提出的 23 种用于解决面向对象软件设计过程中特定问题的方案，因此最契合设计模式的语言应该是 Java 等纯粹的面向对象语言。

设计模式往往是对语言缺陷的补充。JavaScript 的语言特性决定了它并不需要严格意义上的设计模式，但它也确实提供了一套便于沟通交流的设计词汇。

### 2. 常用的设计模式

设计模式可以分为三类：描述对象的创建过程（创建型），描述对象之间的关系（结构型），描述对象的行为（行为型）。

#### 2.1 原型模式（Prototype）

原型模式是 JavaScript 面向对象的基础。主要有四个要点：所有的数据（除了 `undefined`）都是对象；所有的对象（的构造器）都有原型；对象的创建是基于克隆原型而非实例化类；对象会把无法响应的请求委托给它的原型。

尽管 ES6 提供了 class 语法，但其本质依然是基于原型而非基于类。JavaScript 中基于原型创建对象的核心实现如下：

	Object.create = function(obj) {
	  var F = function() {}
	  F.prototype = obj
	  return new F()
	}

#### 2.2 单例模式（Singleton）

单例模式是指一个类仅有一个实例，且具有全局的访问点。单例模式的最大意义在于将创建对象和管理单例的逻辑进行分离。

惰性单例是指只有在真正需要时才创建对象。核心实现如下：

	var getSingle = function(createObj) {
	  var obj
	  return function() {
	    return obj || (obj = createObj.apply(this, arguments))
	  }
	}

#### 2.3 策略模式（Strategy）

策略模式是指将一系列算法进行封装，以实现相互替换使用。策略模式的最大意义在于将策略的实现和使用进行分离。核心实现如下：

	var useStrategy = function(name, args) {
	  return strategy[name](args)
	}

#### 2.4 代理模式（Proxy）

代理模式是指为一个对象提供一个代理，从而控制对它的访问。代理模式的最大意义在于借助替身与本体的接口一致性，实现本体在处理请求前的预处理。

根据替身对请求的预处理类型，代理模式可分为虚拟代理和缓存代理。虚拟代理是将请求延迟执行。真正开始处理请求的时机可能是达到了足够多的请求数目，也可能满足了某种特殊的条件。缓存代理是用缓存替代执行。核心实现如下：

	var cacheProxy = function() {
	  var cache = {}
	  return function() {
	    if (/* 命中缓存 */) return cache[args]
	    return cache[args] = realProcess()
	  }
	}

#### 2.5 迭代器模式（Iterator）

迭代器模式是指按一定的顺序循环访问对象的元素，但不暴露对象的内部结构。迭代器模式的最大意义在于分离迭代过程和业务逻辑。

内部迭代器是封装好迭代规则的迭代器，优点是调用简单，比如借助 for 循环实现 `each()` 方法。外部迭代器则需要显示的请求迭代下一个元素，但更加灵活。核心实现如下：

	var Iterator = function(obj) {
	  var current = 0;
	  var next = function() { current += 1 }
	  var isDone = function() { return current >= obj.length }
	  var getCurrItem = function() { return obj[current] }
	  return {
	    next: next,
	    isDone: isDone,
	    getCurrItem: getCurrItem
	  }
	}

#### 2.6 观察者模式（Observer）

观察者模式又叫做发布-订阅模式（Publish/Subscribe），是指一种对象间的一对多关系：当一个对象的状态发生改变时，所有依赖它的对象都将收到通知。

从回调函数的视角来看，订阅是函数的存储过程，发布是函数的调用过程。通过一个全局的 `Event` 对象，可以实现模块间的通信。核心实现如下：

	var Event = (function() {
	  var sub = {}
	  var listen = function(key, fn) {
	    if (!fn) return
	    if (!sub[key]) sub[key] = []
	    sub[key].push(fn)
	  }
	  var remove = function(key, fn) {
	    if (!sub[key]) return
	    if (!fn) sub[key] = []
	    var fns = sub[key]
	    for (var i = fns.length - 1; i >= 0; i--) {
	      if (fns[i] === fn) fns.splice(i, 1)
	    }
	  }
	  var trigger = function() {
	    var key = Array.prototype.shift.call(arguments)
	    if (!sub[key]) return
	    var fns = sub[key]
	    for (var i = 0; i < fns.length; i++) {
	      fns[i].apply(this, arguments)
	    }
	  }
	  return {
	    on: listen,
	    off: remove,
	    emit: trigger
	  }
	})()

#### 2.7 命令模式（Command）

命令模式是一个对象向另一个对象发送执行某种命令的请求。命令模式的最大意义在于分离请求的发送者和接受者，以及请求的具体操作。核心实现如下：

	var setJob = function(receiver, action) {
	  return function() {
	    receiver[action]()
	  }
	}
	var setCommand = function(sender, event, command) {
	  sender.addEventListener(event, command, false)
	}
	// 生成命令
	var command = setJob(employee, job)
	setCommand(employer, situation, command)

#### 2.8 组合模式（Composite）

组合模式是指通过构建树形的层次结构，实现对整颗树的统一操作。组合模式的最大意义在于忽略了组合对象和基本对象的差异，保证了操作的一致性。核心实现如下：

	var composite = function() {
	  return {
	    list: [],
	    add: function(leaf) { this.list.push(leaf) },
	    exec: function() { this.list.forEach(item => { item.exec() }) }
	  }
	}
	var leaf = {
	  add: function() { throw new Error('can not do this') },
	  exec: function() { /* 具体逻辑 */ }
	}

#### 2.9 模板方法模式（Template Method）

模板方法模式是通过继承使得子类拥有父类的算法结构，并可以选择重写其具体实现。模板方法模式的最大意义在于将子类的方法种类和执行顺序抽象到了父类的模板方法中。核心实现如下：

	var Parent = function() {}
	Parent.prototype.step = function() { /* 空方法 */ }
	Parent.prototype.init = function() {
	  this.step()
	}
	var Child = function() {}
	Child.prototype = new Parent()
	Child.prototype.step = function() { /* 具体实现 */}
	// 使用模板
	var obj = new Child()
	obj.init()

#### 2.10 享元模式（Flyweight）

享元模式是指仅保留对象中可共享的内部状态，并在合适的时刻将已剥离的易变化的外部属性组装进共享对象。模式的最大意义在于通过共享对象实体，解决大量相似对象带来的内存开销。核心实现如下：

	var Fly = function(inner) { this.inner = inner }
	Fly.prototype.manager = function(outer) {
	  this.outer = outer
	}
	var shareObj = new Fly()
	shareObj.manager( /* 首次使用传入外部属性 */ )
	shareObj.manager( /* 再次使用修改外部属性 */)


#### 2.11 职责链模式（Chain of Responsibility）

职责链模式将一些对象连接为一条链，请求在链上依次被传递，直到被某个对象处理。职责链模式的最大意义在于解耦了请求的发送者与接收者。核心实现如下：

	var somePro = function() {
	  if (/* 该节点无法处理 */) return 'nextSuccessor'
	  // 该节点可以处理，提供业务逻辑
	}
	Function.prototype.after = function(fn) {
	  var self = this
	  return function() {
	    var ret = self.apply(this, arguments)
	    if (ret === 'nextSuccessor') {
	      return fn.apply(this, arguments)
	    }
	    return ret
	  }
	}
	var fullProcessor = firstPro.after(secondPro).after(lastPro)

#### 2.12 中介者模式（Mediator）

中介者模式通过引入中介对象，将对象之间的多对多关系解耦为一对多关系。中介者模式的最大意义在于将对象之间的交互移交给了中介对象。核心实现如下：

	var mediator = (function() {
	  return function(obj, type) {
	    // 处理某个节点的某种类型的业务
	  }
	})()
	obj.prototype.someType = function() {
	  mediator(this, someType)
	}

#### 2.13 装饰者模式（Decorator）

装饰者模式是在运行时动态的为对象添加职责，而又不改变对象本身。装饰者模式的最大意义在于可以动态的添加个性化功能。

基于面向切面编程（AOP）的思路，可以做到用户行为的数据统计、表单提交前效验，甚至带 token 的 ajax 请求等装饰性功能。核心实现如下：

	Function.prototype.before(beforeFn) {
	  var selfFn = this
	  return function() {
	    // 插入前置功能
	    beforeFn.apply(this, arguments)
	    return selfFn.apply(this, arguments)
	  }
	}
	Function.prototype.after(afterFn) {
	  var selfFn = this
	  return function() {
	    var ret = selfFn.apply(this, arguments)
	    // 插入后置功能
	    afterFn.apply(this, arguments)
	    return ret
	  }
	}

#### 2.14 状态模式（State）

状态模式是指对象借助状态的转换来改变行为。状态模式的最大意义在于用户只需要关心状态的转换，而无需关心行为的变化。

如果对象在不同状态下具有不同的行为，即状态可以接管行为，那么就很适合通过状态机（State Machine）来描述。核心实现如下：

	var Obj = function(state) {
	  this.currState = state
	}
	Obj.prototype.action = function() {
	  // 将对象行为委托给状态机
	  this.currState.action.apply(this, arguments)
	}
	var FSM = {
	  state1: {
	    action: function() { /* 处理业务，改变状态 */ }
	  },
	  state2: {
	    action: function() { /* 处理业务，改变状态 */ } 
	  }
	}

#### 2.15 适配器模式（Adapter）

适配器模式是通过接口转换的方式解决两个接口不匹配的问题。适配器模式的最大意义在于避免了对复杂接口的直接修改。核心实现如下：

	var adapter = function(oldInterface) {
	  return function() {
	    // 接口转换
	    return newInterface
	  }
	}

#### 2.16 外观模式（Facade）

外观模式是定义了一个高层接口来封装一组子系统，。外观模式的最大意义在于为一组复杂的子系统提供了简单的入口，避免了用户与子系统的直接联系。核心实现如下：

	var system1 = function() { /* 复杂业务 */ }
	var system2 = function() { /* 复杂业务 */ }
	var facade = function() {
	  system1()
	  system2()
	}

### 3. 基本的设计原则

设计原则（Design Principle）是更为一般化的准则。在程序的设计和重构中，通常都会体现出设计原则。比如著名的 SOLID 五大原则：

- 单一职责（Single Responsibility）原则：每个对象只有单一的功能
- 开放封闭（Open/Closed）原则：对象允许被扩展，但不能被修改
- 里氏替换（Liskov Substitution）原则：使用父对象的地方都可以用子对象来替换
- 接口隔离（Interface Segregation）原则：庞大臃肿的接口应该拆分为细小具体的接口
- 依赖反转（Dependency Inversion）原则：具体实现应该依赖于抽象接口

此外，还有一些流传很广的设计原则：

- DRY（Don’t repeat yourself）原则：避免重复代码，解决方案是 WET（write everything twice），事不过三，多次出现的代码就考虑抽象出来
- YAGNI（You aren’t going to need it）原则：只有真正需要时才进行抽象，允许被愚弄一次，避免一开始就过度设计
- KISS（Keep it simple, stupid）：永远追求简单，避免复杂
- 内聚耦合原则：模块内要高内聚，模块间要低耦合
- 最小知识原则：对象之间要尽可能少的交互，单一职责与最小知识可以整体理解为内聚耦合原则

### 参考

1. [JavaScript 设计模式与开发实践 - 曾探 | 豆瓣](https://book.douban.com/subject/26382780/)
2. [JavaScript Patterns - Stoyan Stefanov | O’REILLY](https://github.com/TooBug/javascript.patterns)
3. [Learning JavaScript Design Patterns - Addy Osmani | O’REILLY](https://addyosmani.com/resources/essentialjsdesignpatterns/book/)
