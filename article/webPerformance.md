## web Performance
> 性能优化有关的最佳实践具有很多现实意义

基本上算是三本书的读书笔记了：

- [Yahoo Rules](#yahoo-rules)
- [About Web](#about-web)
- [JS Best Practice](#js-best-practice)

### Yahoo Rules

雅虎的 14 条性能优化规则有个基本点：针对客户端用户的体验感知，前端优化比后端优化更重要。如果具体一些的话，我觉着前端主要是跟资源打交道，涉及资源的加载、解析和执行，其中最影响性能的是资源的加载，毕竟网络 I/O 通常是最慢的。

性能黄金法则也进一步提出，最终用户响应时间中只有 10% ~ 20% 来自页面的加载，剩下的 80% ~ 90% 都是页面中各种资源或者说组件的加载。

对雅虎的 14 条性能优化规则的理解大致如下：

1. **减少 HTTP 请求（make fewer HTTP requests）**：首次访问通常是无缓存的，因此减少 HTTP 请求十分重要。最基本的是合并脚本和样式，这也是前端工程构建最早触碰的点，早在 Gulp 时代就已经出现了。除了脚本和样式，剩下的大多是图片，但图片地图和 CSS sprites 其实很少用了，更多的可能是字体图标。此外，使用基本 base64 的 data: url 内联图片也偶尔会用到。
2. **使用内容发布网络（use a content delivery network）**：毕竟 CDN 也是服务器，相当于强化了 HTTP 的响应能力。如果用户量激增时，把静态资源放在 CDN 上效果可能会很不错。
3. **添加 Expires 头（add an Expires header）**：磁盘 I/O 比网络 I/O 快一个数量级，能缓存的资源尽量去缓存。但如果突然想提供最新的资源，最好能采用修改版本号或添加时间戳的方式，不能依赖让用户主动清空缓存。
4. **压缩组件（gzip components）**：gzip 是对数据进行编码，比单纯的 minify 要厉害很多。经过 minify 的资源可能会减少百分之几十的体积，但经过 gzip 后通常是降低一个数量级。所以可以对样式和脚本进行 gzip，毕竟现在浏览器都支持 gzip 解码。
5. **将样式表放在顶部（put stylesheets at the top）**：渲染引擎是对构建出的 render tree 进行渲染，如果样式迟迟未被解析，dom tree 只能黯然神伤，如果还有一堆其他资源抢在样式表之前加载，那 render tree 的构建就更是遥遥无期。把样式表放在顶部会让构建和渲染同步进行，用户也能早点看到内容，即使不够完整。
6. **将脚本放在底部（put scripts at the bottom）**：脚本的加载和执行默认会阻塞页面中其他的资源加载和解析，靠前的脚本将影响整个页面的渲染，所以把脚本后置是最好。如果使用 async 或 defer 脚本则会异步加载，不会阻塞页面解析，可以根据依赖关系来选择。
7. **避免 CSS 表达式（avoid CSS expression）**：个人感觉 CSS 表达式有些越界，很多使用场景也没有必要。
8. **使用外部 JavaScrip 和 CSS（make JavaScript and CSS external）**： 访问页面不是一锤子买卖，采用外部的资源文件才能触发缓存。其实跟第一点结合的话会有新的意义：请求数目与资源数目是正相关的，而稳定的资源数目与缓存数目是正相关的。
9. **减少 DNS 查询（reduce DNS lookups）**：所有资源加载前都要有 DNS 查询，主机名过多会增加 DNS 查询的负担，主机名过少会浪费并行下载的机会。好在 DNS 通常有本地缓存，而且本身的时间消耗并不多，所以一般不太重要。
10. **精简 JavaScript（minify JavaScript）**：脚本的 minify 主要是删除空白和注释，能够在 gzip 前先对文本进行一轮瘦身，这在工程构建中属于基本流程了。另外，如果平时有不写分号的习惯也会对性能有一点微小的贡献。
11. **避免重定向（avoid redirects）**：重定向会额外增加一次 HTTP 请求。如果不是出于重要目的，就尽可能的避免。如果是为了美化提供给用户的 URL，完全可以采用二级域名。好心的浏览器只负责填充主机名后面缺少的斜杠，如果是为了偷懒少写斜杠，就得承担一次重定向成本。
12. **删除重复脚本（remove duplicate scripts）**：尽管浏览器通常会避免此类情况下的重复请求和重复执行，但包含有相同的脚本还是太蠢了。合理的团队沟通和工程构建都能避免这种问题的出现。
13. **配置 ETag（configure ETags）**：目前还没接触过，感觉一般也用不到。
14. **使 Ajax 可缓存（make Ajax cacheable）**：或许这个主要指异步请求数据也要遵循基本法，比如使用时减少 DNS 查询和避免重定向。

### About Web

web 性能优化其实只是项目开发中的一小部分，而且质量、成本和时间通常只能满足两个。各个因素之间的权衡（trade off）要求我们在考虑优化时，最好先评估各个模块的开销，然后再决定优化哪个部分能够取得最大的收益。有时候我们习惯性认为脚本质量是性能瓶颈，但实际的罪魁祸首可能是 DOM 操作和布局渲染。

如果不考虑 Web Worker 的话，JS 在浏览器中是运行在单线程上，各种基于事件和回调的 API 通过 Event Loop 机制实现异步。为了保证浏览器能够及时响应用户交互，主线程上应该尽可能少的出现计算量过大的操作。通常操作延迟大于 0.1 秒就会被用户感知到，如果超过 1 秒就会被认为缓慢，而如果超过了 10s，用户的耐心就会消磨殆尽。

渲染初始页面通常不需要所有的脚本。需要在 `onload` 事件触发前执行的脚本很有限，如果能够把页面初始化过程并不执行的脚本的加载过程延后，将会提高页面的初始化速度。常规的 `script` 在加载和执行时会阻塞页面的解析和其他资源的加载，而通常来说脚本的同步执行时必要的，因为可能会存在依赖关系和竞争状态，但同步加载是没必要的。

由于 JavaScript 运行在单线程中，因此执行时间过长的脚本将会阻塞用户与浏览器的交互。这需要我们避免几种常见的脚本执行时间过长的情况：过多的 DOM 操作，过多的循环逻辑，以及过多的递归。通常 0.1 秒的延迟就会被用户感知到，因此尽可能避免计算量过大的逻辑，如果存在的话最好也进行拆分，或者通过 timer 将其作为回调纳入事件循环中。

所有的前端资源中，除了文档、样式和脚本，体积最庞大的可能就是图片了。如果无法避免图片的使用，就需要做到无损压缩，即优化后的图片能够保证与原图具有相同的像素级视觉质量，通常 PNG8 是较好的选择。不要在 `img` 标签里对图片进行缩放。可以为图片划分单独的域，即不同的主机名，来换取更多的服务器连接数目，从而提高图片的并发下载能力。

HTML 和 CSS 方面也有优化的可能性。比如少用 `iframe` 元素，因为它的创建开销很大，而且会阻塞父窗口的 `onload` 事件，还会占用相同主机名的服务器的连接数目。比如简化 CSS 选择器的使用，这里的重点是要知道选择器是从右向左匹配的，这决定了选择器最右侧的参数才是关键选择器，因为它负责了元素的初次匹配。目前，CSS 对性能的影响通常已经不是选择器了，而是重复的样式和昂贵的效果。

### JS Best Practice

有关 JavaScript 的最佳实践。

#### Loading and Execution

脚本的加载和执行不仅会阻塞页面的解析和渲染，还会阻塞用户界面的交互。因此针对脚本资源需要以下几个优化：

- 脚本位置：最好放在 `body` 元素底部，避免阻塞页面渲染
- 脚本数量：最好能适当的进行合并，减少一些 http 请求次数
- 加载方式：最好使用异步并行加载的 defer 脚本，能够延迟到 `onload` 事件触发前执行

#### Data Access

数据有四种存储方式：字面量、变量、数组元素、对象成员。整体上，前两者的存取速度要快于后两者。

每个函数都有一个执行上下文（execution context），它对应了函数的生命周期。每个执行上下文都具有作用域链（scope chain），它用于解析标识符。函数被调用时，执行上下文被创建，作用域链也被初始化，即通过复制得到一些变量对象（variable object），变量对象则包含了函数执行所需要的 `this`、`arguments` 和各种变量。

局部变量位于作用域链最顶部的活动对象（activation object）中，而全局变量位于作用域链底部。函数执行过程中的每次标识符查询，都会沿着作用域链从顶部到底部进行搜索。因此在标识符查询方面，局部变量的解析速度要快于全局变量。

如果函数内部存在内层函数（比如一个回调函数，或者一个事件绑定），则外层函数的活动对象会被内层函数的作用域链所持有，因此即使外层函数的执行上下文已经被销毁，内层函数依然拥有该词法作用域的变量的访问权，即形成了函数闭包。内层函数的作用域链将具有三个变量对象：自身的活动对象，外层函数的活动对象，以及全局对象所对应的变量对象。

访问对象成员的速度通常会很慢。对象成员分为两种：实例成员和原型成员。前者直接存储在对象实例中，后者则从对象原型中继承而来。在解析对象的一个属性或方法时，会沿着原型链从实例到原型链尾端进行搜索。为了提高访问性能，通常可以把需要多次访问的多层嵌套的属性成员保存在局部变量中，但对象的方法成员并不适用，因为会改变其 this 指向的执行上下文。

总之，可以将深层作用域链的变量和深层原型链的属性都缓存在局部变量中。

#### DOM Scripting

在浏览器环境中，DOM API 连接了文档渲染引擎和脚本解释引擎，这种桥梁作用决定了 DOM 操作不可能会很快。

为了提升速度，就尽量把运算放在 JavaScript 这一端，尽可能少的访问 DOM。必须进行某种 DOM 操作时，选择最高效的 DOM API，比如 `querySelector()` 和 `querySelectorAll()` 要比其他选择器 API 更高效：无需级联即可完成，返回的 NodeList 也不与文档保持实时同步。如果需要多次访问某个 DOM 节点，就用局部变量保存其引用。

渲染引擎解析完文档的各种资源后，会构建出两个数据结构：表示页面结构的 DOM tree，表示页面呈现的 render tree。后者实际上对应了每一个需要被显示的节点的盒模型。然后渲染引擎就会按照 render tree 在屏幕上进行绘制。这里涉及两个概念：

- 重排（reflow）：元素的几何属性发生变化时，render tree 重新构建的过程
- 重绘（repaint）：因为某些变化（未必是元素的几何属性），屏幕上显示的内容需要重新绘制的过程

可以看出，重排必然会引起重绘，但重绘未必因为重排。比如 `display: none` 会同时触发重排和重绘，而 `visibility: hidden` 只触发重绘。但对渲染引擎来说，二者都是十分昂贵的操作。浏览器为了避免频繁的排版和绘制，通常会批量化处理 render tree 的变化，但如果代码主动要获取布局相关的信息，则会直接执行并刷新渲染队列。

如果想避免低效的代码，首先需要知道具体什么情况会触发重排：

- 用户操作变化：窗口尺寸改变，页面发生滚动
- 页面结构变化：页面进行初始化渲染；DOM 节点的增删改
- 节点自身变化：节点位置改变，节点尺寸改变，节点内容改变

通用的策略减少可能会触发重排和重绘的操作：

- 如果节点具有动画（通常节点的位置、尺寸，甚至内容都会频繁的变化），通过绝对定位使其脱离文档流，动画结束时再还原布局
- 如果对节点的操作涉及布局信息，就将其赋值给局部变量缓存起来
- 如果需要修改大量的节点样式，避免大量使用 `style` 接口，建议修改 `className` 并配合 CSS 使用

如果有一些列复杂的 DOM 操作，考虑先将元素脱离文档流，操作完毕后再放回文档。方式有 3 种：

第一种是对目标元素先隐藏再还原：

```javascript
ele.style.display = 'none'
// 一些操作
ele.style.display = 'block'
```

第二种是操作片段元素再放回文档：

```javascript
let fragment = document.createDocumentFragment()
// 一些操作
parent.appendChild(fragment)
```

第三种是操作副本元素再替换本体：

```javascript
let clone = target.cloneNode(true)
// 一些操作
target.parentNode.replaceChild(clone, target)
```

最后，还需要优化一下 DOM 事件。常规的方式会给每一个需要响应事件的元素都绑定一个 handler，但大量的 handler 不仅会增加代码的执行量，还会因为浏览器的追踪而占用大量内存。最好的方式是将子元素的事件处理委托给父元素，即在父元素中通过检查 `event.target` 来判断冒泡上来的事件的真实目标，然后针对性的书写逻辑。

#### Algorithm and Flow Control

代码结构相关的优化主要涉及三类：循环优化，条件优化，递归优化。

对循环来说，对象的 `for-in` 循环因为包括了所有的实例属性和继承属性而稍慢；数组的 `forEach` 迭代因为基于调用外部方法也会稍慢。传统的循环优化策略通常是两点：减少迭代工作量，减少迭代总次数。

减少迭代工作量：

```javascript
// 普通 for 循环
for (let i = 0; i < items.length; i++) { /* ... */ }

// 减少属性查找
let len = items.length
for (let i = 0; i < len; i++) { /* ... */ }

// 反转迭代顺序
for (let i = items.length; i--; ) { /* ... */ }
```

减少迭代总次数：

```javascript
let i = items.length % 8
while(i) {
  process(items[i--])
}

i = Math.floor(items.length / 8)
while(i) {
  process(items[i--])
  process(items[i--])
  process(items[i--])
  process(items[i--])
  process(items[i--])
  process(items[i--])
  process(items[i--])
  process(items[i--])
}
```

对条件语句来说，为了平衡可读性和性能，通常在条件少时选 `if-else`，条件多时选 `switch`。整体上，前者适合判断两个离散值（或者若干个值域），后者适合判断多个离散值。多值域判断场景下，需要让目标语句尽早的到达：把概率大的分支前置，或者使用二分法嵌套分支。对某个 `value` 值进行逻辑映射时，可以结合数组查找表 `results[value]` 进行代码优化。

对递归来说，调用栈的大小限制可能会导致 stack overflow 错误。对递归的优化通常有：将递归部分展开为迭代方式；缓存递归结果来避免重复计算。

#### Strings and Regular Expressions

字符串连接方面，其实 `+` 或 `+=` 已经足够，而且通常比数组方法或字符串方法要快：

```javascript
// 较慢
str3 = ['a', 'b', 'c'].join('')
str4 = str4.concat('b', 'c')
```

（todo）




### 参考
1. [高性能网站建设指南 - Steve Souders](https://book.douban.com/subject/26411563/)
2. [高性能网站建设进阶指南 - Steve Souders](https://book.douban.com/subject/26411563/)
3. [高性能 JavaScript - Nicholas C. Zakas](https://book.douban.com/subject/26599677/)
