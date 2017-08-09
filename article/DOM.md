## DOM
> 框架可以让人专注数据逻辑，但文档结构永远是基础

文档对象模型（DOM）是文档的编程接口。它提供了一种对文档的结构化表述，以及一种对其结构进行访问的方式。简言之，DOM 连接了 web 页面和脚本程序。

笔记结构如下：

- [DOM Standard](#dom-standard)
  - [infrastructure](#infrastructure)
  - [events](#events)
  - [nodes](#nodes)
  - [others](#others)
- [DOM Manipulate](#dom-manipulate)
  - [window](#window)
  - [document](#document)
  - [element](#element)
  - [event](#event)

### DOM Standard

DOM 标准貌似有两个版本的（无力吐槽啊），whatwg 和 w3c 的存在一些差异。这里学习的是 whatwg 的。

#### infrastructure

##### 树

树是一种有限层级结构，树序是按深度优先遍历来预设的。树中的对象，除了根（root）都有父（parent），除了叶（leaf）都有子（child）。父/子关系是一种特殊的祖先/后代（ancestor/descendant）关系。

共享父的对象是兄弟（sibling）关系。在树序中，一个对象的第一个前（precede）兄弟和第一个后（follow）兄弟分别被称为它的上一个（previous）兄弟和下一个（next）兄弟。对象的索引（index）指的是它的前兄弟的个数。

##### 其他

有序集合（ordered set）的 parser 负责将字符串按空白切分为 token，而 serializer 负责通过添加空白将 token 拼接为字符串。

#### events

##### 机制

事件是调度（dispatch）给对象某个事情，比如网络活动，或者用户交互。对象可以通过添加监听器（listener）来观察（observe）某类事件，也可以通过移除监听器来取消观察。

```javascript
var cb = function() { /* 处理逻辑 */ }
obj.addEventListener('click', cb, false)
obj.removeEventListener('click', cb, false)
```

内置事件由浏览器视情况自动触发，自定义事件则需要人工手动触发。广义上，事件的触发（fire）包括创建过程、初始化过程和调度过程。自定义事件的创建可以通过 `Event` 构造函数或 `CustomEvent` 构造函数。

```javascript
obj.addEventListener('cat', e => { process(e.detail) })
var event = new CustomEvent('cat', {'detail': data})
obj.dispatchEvent(event)
```

事件流用来描述一个事件在被调度时在对象树中的传递轨迹。事件的活动范围包括了目标对象及其所有的祖先对象，整个过程分为三个阶段（phase）：从 window 对象沿树序到达目标对象的捕获（capture）阶段，停留在目标对象的（target）阶段，以及从目标对象沿反树序到达 window 对象的冒泡（bubble）阶段。

##### 接口

监听器的回调函数中，默认第一个参数是事件对象本身。事件对象具有很多常用属性，以供回调函数可以精细的控制逻辑。

- `event.type`：事件的类型，如 click
- `event.target`：事件调度的目标对象，树结构的最底层
- `event.isTrusted`：事件的调度是否来自浏览器
- `event.currentTarget`：目标对象的某个祖先对象，它的事件监听器的回调函数正在被触发
- `event.stopPropagation()`：停止事件的传递，停留在当前对象
- `event.preventDefault()`：阻止事件原有的默认行为

普通的事件监听无法覆盖到动态添加的元素，并且会在树结构中引入过量的监听器。利用冒泡机制，可以只在高层节点监听。

#### nodes

DOM API 用于访问和操作文档，这里的文档指的是任何基于标记（markup）的资源。文档的 parser 可以将标记转化为节点树，节点树分为文档树（document tree）和影子树（shadow tree）。

##### Node

Node 接口被所有类型的节点（node）使用，包括 Document、Element 等。常用的属性有：

- `node.baseURI`：返回节点所在文档的基址
- `node.nodeType`：返回节点类型的数值
- `node.nodeName`：返回节点名称的大写字符串
- `node.isConnected`：返回节点是否被连接
- `node.ownerDocument`：返回节点所在的文档
- `node.nodeValue`：返回或设置当前节点的值
- `node.textContent`：返回或设置节点及其后代的文本内容
- `node.parentNode`：返回节点的父
- `node.parentElement`：返回节点的父元素
- `node.childNodes`：返回节点的所有孩子
- `node.firstChild`：返回节点的第一个子
- `node.lastChild`：返回节点的最后一个子
- `node.previousSibling`：返回节点的上一个兄弟
- `node.nextSibling`：返回节点的下一个兄弟

常用的方法有：

- `node.getRootNode()`：返回节点的根
- `node.getRootNode({ composed:true })`：返回节点的含影子的根
- `node.hasChildNodes()`：返回节点是否有子
- `node.contains(node)`：返回另一个节点是否是其后代
- `node.cloneNode([deep = false])`：返回节点的一个拷贝，deep 为 `true` 则包含节点的后代
- `node.isEqualNode(otherNode)`：返回两个节点是否具有完全相同的属性
- `node.normalize()`：删除空的文本节点，合并相邻的文本节点
- `node.insertBefore(node, childNode)`：将指定的节点插入到当前节点的某个子节点之前
- `node.appendChild(node)`：将指定的节点添加为当前节点的最后一个子节点，若参数节点已存在于 DOM 树就移动它
- `node.replaceChild(childNode, node)`：将当前节点的某个子节点替换为指定的节点
- `node.removeChild(child)`：将某个子节点从当前节点移除

Node 接口具有一些子接口，比如 ParentNode 和 ChildNode，它们的属性和方法相对来说并不常用。

##### NodeList

集合（collection）是表示一系列节点的对象。集合一般是实时的而非静态的，即它的属性和方法是对实际数据而非快照数据在操作。集合是某个根的子树视图，且仅包含某个过滤器的一些匹配节点。视图是线性的，集合节点按照树序排序。

HTMLCollection 对象是 element 的集合，但已不推荐使用。NodeList 对象是 node 的集合。属性和方法有：

- `collection.length`：返回集合中节点的数目
- `collection.item(index)`：返回集合中匹配参数 index 的节点
- `collection[index]`：同上

NodeList 是一种类数组对象，但由于其原型链为：`NodeList.prototype --> Object.prototype`，因此并不具有数组的任何方法。可以通过 `Array.from()` 来将 NodeList 转化为数组。

##### Element

Element 继承自 Node，同时它也是所有元素（HTMLElement）和具体元素（如 Image）的基础。常用的属性有：

- `element.id`：返回 id 特性
- `element.tagName`：返回大写的标签名
- `element.className`：返回 class 特性的字符串
- `element.classList`：返回 class 特性的 token
- `element.attributes`：返回元素的特性组成的对象

常用的方法有：

- `element.hasAttribute(name)`：返回元素是否有指定特性
- `element.hasAttributes()`：返回元素是否有特性
- `element.getAttribute(name)`：返回指定特性名对应的特性值
- `element.getAttributeNames()`：返回由元素的特性组成的数组
- `element.setAttribute(name, value)`：设置指定的特性
- `element.removeAttribute(name)`：移除指定的特性
- `element.getElementsByTagName(name)`：返回后代中能匹配指定标签名的元素集合
- `element.getElementsByClassName(classNames)`：返回后代中能匹配类名的元素集合
- `element.querySelector(selectors)`：返回后代中第一个能匹配选择器字符串的节点
- `element.querySelectorAll(selectors)`：返回后代中所有能匹配选择器字符串的节点列表
- `element.insertAdjacentElement(where, element)`：将指定元素插入到指定的 4 种位置之一

HTMLElement 是代表所有 HTML 元素的接口，常用的接口之间的继承关系如下：

```
HTMLElement --> Element --> Node --> EventTarget
```

##### Document

同 Element 一样，Document 也是继承自 Node 的接口。Document 代表了浏览器中载入的 web 页面，是页面内容（即 DOM tree）的入口。

通常的实现是指 HTMLDocument 接口。常用的属性有：

- `document.URL`：返回文档的 location 对象的字符串表示
- `document.origin`：返回文档的源，即协议和域名
- `document.domain`：返回文档的域，即域名
- `document.ducumentElememnt`：返回文档的直接子元素，通常是 `<html>` 元素
- `document.characterSet`：返回文档所使用的字符编码，通常是 UTF-8
- `document.contentType`：返回文档由 MIME 头部指定的内容类型，通常是 text/html
- `document.doctype`：返回文档的文档类型声明（DTD），通常是 `<!DOCTYPE html>`
- `document.lastModified`：返回文档的最后修改的时间戳

常用的方法有：

- `document.getElementById(id)`：返回匹配 id 特性的一个后代元素
- `document.getElementsByTagName(name)`：返回匹配标签名的后代元素集合，参数为 `*` 则代表全部
- `document.getElementsByClassName(classNames)`：返回完全匹配参数各项（以空格间隔的类名）的集合
- `document.querySelector(selectors)`：返回后代中第一个能匹配选择器字符串的节点
- `document.querySelectorAll(selectors)`：返回后代中所有能匹配选择器字符串的节点列表
- `document.createElement(tagName [, options])`：创建一个元素
- `document.createDocumentFragment()`：创建一个空的 DocumentFragment 节点

##### MutationObserver

MutationObserver 对象用来观察节点树的突变。突变算法描述了在进行一系列节点操作（insert，append，replace，remove）时的步骤。创建观察者时需要传入回调，观察者开始观察后，目标的所有突变记录会被观察者记录在 MutationRecord 中，回调函数会被放入微任务（microtask）队列中等待处理。

```javascript
var cb = function(record, observer) { /* 逻辑 */ }
var observer = new MutationObserver(cb)
observer.observe(target, options)
observer.disconnect()
observer.takeRecords()
```

`observe()` 的 options 参数对象用来描述要观察的突变类型，`takeRecords()` 返回的突变记录会作为回调函数的参数。

#### others

第 4 章讲述了范围（ranges）。Range 对象表示两个边界点之间的节点树的内容。它通常用于对选择和拷贝的内容进行编辑，比如用户选择的文本。

第 5 章讲述了遍历（traversal）。NodeIterator 对象和 TreeWalker 对象可以用来过滤和遍历节点树。

第 6 章讲述了集合（sets）。DOMTokenList 对象是历史遗留问题。

第 7 章讲述了历史（historical）。该部分记录了 DOM Events、DOM Core、DOM Ranges，以及 DOM Traversal 的变更历史。

### DOM Manipulate

DOM 提供了许多接口，但有些其实很糟粕或者并不实用（比如 CSS 相关的接口）。这里总结一些常用的。

#### window

window 对象有一些有用的方法，比如 `postMessage()` `setTimeout()` `setInterval()`。

另外，还包含一些子对象，比如 location 对象，history 对象，document 对象，screen 对象。

location 对象的属性有：`href` `protocol` `host` `hostname` `port` `pathname` `search` `hash`。属性值为字符串，并且可读也可写。其中，`href` 是完整的 url 信息，`host` 是主机名和端口号，`hostname` 只含主机名。方法有 `assign(url)` `replace(url)` `reload()`。三者都用于载入一个文档，且无返回值。

history 对象的基本的属性和方法有：`length` `go(index)` `back()` `forward()`。常用的属性和方法有：`state` `pushState(state, title, url)` `replaceState(state, title, url)`。

#### document

document 对象也是 window 的子对象。

可以获取一些文档元数据。常用的属性有 `readyState` 和 `title`。二者的属性值均为字符串。

可以获取特定的元素。常用的方法有 `getElementById(id)` `querySelector(str)` `getElementsByTagName(tag)` `getElementsByClassName(class)`  `querySelectorAll(str)`。其中前两个返回 HTMLElement 或者 null，后三个返回可能为空的 NodeList。返回 NodeList 后可以继续链式调用。

#### element

HTMLElement 对象代表 DOM 结构中的元素。

可以获取元素的元数据。属性 `id` `tagName` `className` 可以获取元素对应属性的字符串，属性 `disabled` `checked` `hidden` 可以获取元素对应属性的布尔值。

可以操作元素的类列表。`classList` 可以获取 `className` 的 DOMTokenList 对象，该对象具有 `length` 属性，以及一些操作类成员的方法：`add(class)` `remove(class)` `contains(class)` `toggle(class)`。

可以操作元素的属性值。一些操作属性值的方法：`getAttribute(key)` `setAttribute(key, val)` `hasAttribute(key)` `removeAttribute(key)`。

还有一组最复杂的 DOM 操作，通过这些操作可以修改 DOM 层级结构：`appendChild(ele)` `removeChild(ele)` `insertBefore(newEle, childEle)` `innerHTML` `outerHTML`。

#### event

事件的机制和接口参考 [前面的表述](#events)，本部分主要总结常用的事件类型。

资源事件有：

- cached：列在 manifest 中的资源已下载完毕，即应用已被缓存时触发
- error：某个资源加载失败时触发
- abort：某个资源加载中止时触发
- load：某个资源及其依赖资源加载完毕时触发
- beforeunload：window，document 及其资源即将被卸载时触发
- unload：document 及其资源正在被卸载时触发

网络事件有：

- online：浏览器获得网络访问时触发，此时 `navigator.onLine` 值为 `true`
- offline：浏览器失去网络访问时触发，此时 `navigator.onLine` 值为 `false`

焦点事件：

- focus：元素获得焦点时触发（不冒泡）
- blur：元素失去焦点时触发（不冒泡）
- focusin：元素获得焦点时触发（冒泡）
- focusout：元素失去焦点时触发（冒泡）

会话历史事件：

- pagehide：刚好已遍历某个记录条目时触发
- pageshow：刚好要遍历某个记录条目时触发
- popstate：正要导航到某个记录条目时触发

表单事件：

- reset：点击重置按钮时触发
- submit：点击提交按钮时触发
- change：`<input>` `<select>` 或 `<textarea>` 的 value 值的更改被提交时触发
- input：`<input>` `<select>` 或 `<textarea>` 的 value 值更改时同步触发

文本组合事件：

- compositionstart：（使用 IME 时）进入组字模式时触发
- compositionupdate：（使用 IME 时）组字更新时触发
- compositionend：（使用 IME 时）离开组字模式时触发

视图事件：

- resize：文档视图被调整大小时触发
- scroll：文档视图或元素被滚动时触发

剪贴板事件：

- cut：选择的内容被剪切并复制进了剪贴板时触发
- copy：选择的内容被复制进剪贴板时触发
- paste：剪贴板的内容被粘贴时触发

键盘事件：

- keydown：任意一个键被按下时触发
- keypress：某个字符键被按压时触发
- keyup：任意一个键被释放时触发

鼠标事件：

- mousedown：（在元素上）按下鼠标时触发
- mouseup：（在元素上）释放鼠标时触发
- click：（在元素上）按下并释放鼠标时触发
- dblclick：（在元素上）双击鼠标时触发
- mousemove：（在元素内）移动指针时触发
- mouseenter：指针进入目标元素时触发（不冒泡）
- mouseover：指针进入目标元素或其子元素时触发（冒泡）
- mouseleave：指针离开目标元素时触发（不冒泡）
- mouseout：指针离开目标元素或其子元素时触发（冒泡）

拖放事件：

- dragstart：用户开始拖动元素或文本选区，在 draggable target 上触发
- drag：用户正在拖动元素或文本选区，在 draggable target 上触发
- dragend：用户结束拖动元素或文本选区，在 draggable target 上触发
- dragenter：（被拖动的）元素或文本选区进入有效释放目标区，在 drop targets 上触发
- dragover：（被拖动的）元素或文本选区正在有效释放目标区，在 drop targets 上触发
- dragleave：（被拖动的）元素或文本选区离开有效释放目标区，在 drop targets 上触发
- drop：在有效释放目标区内释放元素或文本选区，在 drop targets 上触发

还有一些其他的常用事件：

- storage：`localStorage` 或 `sessionStorage` 被修改时触发
- hashchange：url 的 hash 部分改变时触发
- visibilitychange：标签页内的内容可见或隐藏时触发
- DOMContentLoaded：初始文档加载并解析完毕后触发，不等待样式、图片和子框架

### 参考
1. [DOM Standard | whatwg](https://dom.spec.whatwg.org/)
2. [DOM - web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)
3. [You-Dont-Need-jQuery](https://github.com/oneuijs/You-Dont-Need-jQuery)
4. [You Might Not Need jQuery](http://youmightnotneedjquery.com/)
5. [事件类型一览表 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/Events)
