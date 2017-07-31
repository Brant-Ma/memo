## ECMAScript 6
> ES6 跟 ES5 比起来就像是另一门语言

尼古拉斯的 Understanding ES6 都快第二版了，但国内第一版还没出来。那就不等啦。笔记目录如下：

1. [Block Binding](#block-binding)
2. [String & Regular Expression](#string-and-regular-expression)
3. [Function](#function)
4. [Object](#object)
5. [Destructuring](#destructuring)
6. [Symbol & Symbol Property](#symbol-and-symbol-property)
7. [Set & Map](#set-and-map)
8. [Iterator & Generator](#iterator-and-generator)
9. [Class](#class)
10. [Array](#array)
11. [Promise & Asynchronous Programming](#promise-and-asynchronous-programming)
12. [Proxy & Reflection](#proxy-and-reflection)
13. [Module](#module)

其他附录如下：

- [Minor Improvements](#minor-improvements)
- [ES7](#es7)

### Block Binding

使用 `var` 声明的变量，其声明部分会被提升至所在作用域的顶部。在没有块级作用域的情况下，变量提升（hoisting）会带来很多麻烦。因此 ES6 引入了块级作用域来改进。

在块内声明的变量无法在块外访问。块级作用域在两种情况下被创建：在一个函数内，或者在一个代码块内。使用 `let` 声明的变量只存在于当前代码块中，并且不存在变量提升。另外，也不能用 `let` 重复声明一个当前作用域已存在的标识符：

```javascript
var str = 'hello'

let str = 'world'  // 语法错误
```

与 `let` 声明一样，`const` 声明的变量也是块级声明，因此同样不能再块外被访问，也不存在变量提升，标识符也不能被重复声明。唯一的区别是，使用 `const` 声明的变量被认为是一个常量，它不能被再次赋值，因此必须在声明时就完成初始化。但如果声明的是对象，则稍有不同：

```javascript
const js = { edition: 'es5' }

js.edition = 'es6'       // 工作正常

js = { edition: 'es6' }  // 抛出错误
```

可以看出，`const` 只是阻止对变量绑定的修改，并不阻止对成员值的修改。

由于不存在变量提升，使用 `let` 和 `const` 声明的变量会被放在暂时性死区（temporal dead zone）内，在声明位置之前无法被访问。但在块级作用域外是可以被访问的：

```javascript
console.log(typeof hi)    // 返回 'undefined'

{
  console.log(typeof hi)  // 引用错误
  let hi = 'world'
}
```

如果 `var` 声明在 `for` 循环内会很糟糕：计数器变量在循环外部仍能被访问。更糟糕的是，如果循环内创建了引用它的函数，所有的函数只能共享循环结束后的计数值：

```javascript
var funcs = []

for (var i = 0; i < 10; i++) {
  funcs.push(() => { console.log(i) })
}

funcs.forEach((func) => { func() })  // 输出 10 次数值 '10'
```

在没有块级绑定的时候，我们会使用 IFFE 来解决这个问题，即将变量的副本（而非引用）传入函数：

```javascript
for (var i = 0; i < 10; i++) {
  funcs.push(((val) => {
    return () => {
      console.log(val)
    }  
  }(i)))
}
```

有了块级绑定后，只需将 `var` 改为 `let` 即可实现预期。循环内的 `lei` 声明具有特殊的行为：每次迭代都会创建一个新的 `i` 变量并对它进行初始化，保证每次迭代的 `i` 都是一个独立的 `i` 副本。在 `for-in` 和 `for-of` 循环中，`let` 声明也具有相同的行为。

与循环内的 `let` 声明不同的是，`const` 声明只能工作在只能在 `for-in` 和 `for-of` 循环中，而不能工作在 `for` 循环中。原因很简单，前者在每次迭代中创建了新的变量绑定，而后者试图修改已绑定的变量的值：

```javascript
for (const key in obj) { /* ... */ }          // 工作正常

for (const i = 0; i < 10; i++) { /* ... */ }  // 抛出错误
```

块级绑定在全局作用域也与 `var` 声明不同。在全局作用域使用 `var` 声明不仅会创建一个全局变量，也会成为全局对象的一个属性，这可能会无意间覆盖一个全局属性。而块级绑定只会创建全局变量，不会被添加到全局对象上。这意味着全局变量只会屏蔽全局属性，而不会覆盖它：

```javascript
let hello = 'hello'
console.log('hello' in window)  // false

var world = 'world'
console.log('world' in window)  // true
```

块级绑定的最佳实践是不使用 `var`，默认情况下使用 `const`，只有在知道变量值需要被更改时才使用 `let`。

### String and Regular Expression

JS 的字符串一直以来都是以 16 位字符编码方式为基础的：每 16 bit 作为一个码元（code unit），用以表示一个字符。Unicode 为世界上所有的字符提供全局的唯一标识符，即代码点（code point）。由于 Unicode 在多语言基本平面外增加了扩展平面，单个 16 位的码元已经无法表示所有的代码点。

解决方式是允许两个 16 位码元来表示单个代码点。比如 Unicode 字符 `𠮷` 实际上是两个 16 位的字符，所以原来针对 16 位的操作，将出现很多正确但不合理的情况：

```javascript
var text = '𠮷'
console.log(text.length);         // 2
console.log(/^.$/.test(text));    // false
console.log(text.charAt(0));      // ''
console.log(text.charAt(1));      // ''
console.log(text.charCodeAt(0));  // 55362
console.log(text.charCodeAt(1));  // 57271
```

ES6 针对这些问题，提供了很多新的针对 32 位的字符串方法来应对。`codePointAt()` 方法接收字符串中 16 位码元的位置并返回代码点。如果该位置本身就是 16 位字符，或者是 32 位字符的后半部分，则返回值与 `charCodeAt()` 一致；如果该位置是 32 位字符的前半部分，则返回完整的代码点：

```javascript
const is32Bit = (c) => c.codePointAt(0) > 0xffff

console.log(is32Bit('𠮷'))        // true
console.log('𠮷'.codePointAt(0))  // 134071
console.log('𠮷'.codePointAt(1))  // 57271
```

另外，`String.fromCodePoint()` 可以视为 `String.fromCharCode()` 的完善版本，或者说是 `codePointAt()` 的相反操作。

内容相同的字符可以有不同的代码点，比如 `æ` 和 `ae`，但是直接比较并不会相等。必须先用`normalized()` 方法可以将字符标准化为相同的形式。

正则表达式也是基于 16 位码元来表示单个字符的。使用 `u` 标志可以将工作模式切换到针对代码点而非码元：

```javascript
const codePointLength = (text) => {
  var result = text.match(/[\s\S]/gu)
  return result ? result.length : 0
}

console.log(codePointLength('𠮷'))  // 1
```

使用 `u` 标志前提前检查可用性也很有必要（稍微修改后也可以用于检测新的 `y` 标志）：

```javascript
const hasRegExpU = () => {
  try {
    var pattern = new RegExp('.', 'u')
    return true
  } catch (ex) {
    return false
  }
}
```

甚至还可以分别获取正则表达式的模式和标志：

```javascript
const reg = /abcde/imguy

console.log(reg.source)  // 'abcde'
console.log(reg.flags)   // 'gimuy'
```

使用 `str.indexOf(substr) !== -1` 来判断 `str` 包含 `substr` 很不语义。新的方式是使用 `includes()` 方法，`startsWith()` 方法和 `endsWith()` 方法。它们的第一个参数是子串，第二个参数是首次匹配的索引位置，返回布尔值。

只有需要确切位置，或者需要使用正则表达式时才使用 `indexOf()` 方法和 `lastIndexOf()` 方法。另外，新的 `repeat()` 方法可以将字符串重复固定的次数，返回一个新字符串。

ES6 在字符串方面引入的最大改进是模板字面量（template literal）。它提供了领域专用语言（domain-specific language）的语法，用于解决一些特殊的问题。模板字面量使用一对反引号来包裹内容，且无需对 `'` 或 `"` 转义（但反引号肯定是需要转义的）。

此前如果想要创建多行字符串，需要一些奇怪的方式。

```javascript
const msg = ['multiline', 'string'].join('\n')

const msg = 'multiline\n' + 'string'
```

现在可以直接在模板字面量中换行了。不过要注意缩进的问题：

```javascript
const msg = `multiline
string`

const msg = `multiline\nstring`

const html = `
<div>
  <p>paragragh</p>
</div>`.trim()
```

模板字面量的特点在于可以在 `${` 和 `}` 之间放入 JS 表达式，这被称为替换位。无论是变量、计算、函数调用，都可以放进替换位中。甚至可以嵌入另一个模板字面量（因为模板字面量本身也是表达式）：

```javascript
const name = 'world'
const msg = `hello, ${name}`
```

模板字面量最强大的功能是模板标签（template tag）。标签是在模板的第一个反引号之前被指定的函数。标签函数在调用时会接收模板字面量数据，并划分为独立片段，然后组合这些信息片段来输出最终的字符串。对于一个模板标签，其标签函数的形态如下：

```javascript
const msg = tag`hello, world!`

function tag(literals, ...substitutions) { /* ... */ }
```

其中，`literals` 是个数组，包含了被替换位隔开的字面量字符串，`substitutions` 是剩余参数数组，包含了替换位的解释值。可以得知，`literals.length === substitutions.length + 1`。如果用标签函数来模拟模板字面量默认的转换操作，模拟实现将大致如下：

```javascript
function defaultTag(literals, ...substitutions) {
  let result = ''

  for (let i = 0; i < substitutions.length; i++) {
    result += literals[i]
    result += substitutions[i]
  }

  result += literals[literals.length - 1]
  return result
}
```

可以看出模板字面量的默认标签行为是：交替拼接两个数组中的元素，最终创建出完整的结果字符串。需要注意的是，如果替换位是位于模板字面量的开始或结束，那么 `literals[0]` 或 `literals[literals.length - 1]` 的值是空字符串。此外，`literals` 数组元素的值一定是字符串，但 `substitutions` 数组元素的值则由替换位的解释结果决定。

```javascript
const product = 'AirPods'
let price = 159
let taxRate = 0.17
let exchangeRate = 6.925
const rmb = price * (1 + taxRate) * exchangeRate

let msg = `The ${product} costs ￥${parseInt(rmb)} in China.`
console.log(msg)  // 'The AirPods costs ￥1288 in China'
```

针对上面的例子，`literals` 数组的元素值分别为：`'The '`、`' costs ￥'` 和 `' in China.'`，而 `substitutions` 数组的元素值分别为 `'AirPods'` 和 `1288`。

其实模板标签也可以访问字符串的原始信息，即转义之前的形式。具体而言，原始信息是存放在 `literal.raw` 属性里，即 `literal[i]` 的原始信息存放在 `literal.raw[i]` 中。内置的 `String.raw()` 标签默认具有获取原始字符串值的功能，模拟实现如下：

```javascript
function rawTag(literals, ...substitutions) {
  let result = ''

  for (let i = 0; i < substitutions.length; i++) {
    result += literals.raw[i]
    result += substitutions[i]
  }

  result += literal.raw[literal.length - 1]
  return result
}
```

具体的使用上，`rawTag()` 与 `String.raw()` 是等价的：

```javascript
const msgOne = rawTag`multiline\nstring`
const msgTwo = String.raw`multiline\nstring`

console.log(msgOne)  // 'multiline\\nstring'
console.log(msgTwo)  // 'multiline\\nstring'
```

可以看出，转义字符是以原始形式返回的：作为换行字符的 `\n` 的代码形式是一个 `\` 字符和一个 `n` 字符，即 `\\n` 的原始形式。

### Function

JS 函数在调用时的参数数量可以不同于声明时的参数数量。如果调用时提供的参数过多，通常没有太大的影响；如果调用时提供的参数过少，则需要考虑使用默认值。ES6 提供了既安全又简洁的初始化方式来提供默认参数：

```javascript
// 不安全
function request(url, cb) {
  cb = cb || function(data) { console.log(data) }
}
// 太繁琐
function request(url, cb) {
  cb = (typeof cb !== 'undefined') ? cd : function(data) { console.log(data) }
}
// ES6 方式
function request(url, cb = function(data) { console.log(data) }) {
}
```

默认参数触发的条件是：没有传入该参数，或者显式的传入 `undefined` 赋值给参数。通常在参数声明中，默认参数排列在非默认参数的后面，但其实可以给任意参数指定默认参数，因为显式传入 `undefined` 可以保证占位，避免后面的非默认参数无法传入。

如果使用默认参数，`arguments` 对象将立即与具名参数分离，更不会反映具名参数后续的变化。即 `arguments` 对象将始终反映初始的调用状态。

```javascript
function func(a, b = 1) {
  console.log(arguments[0] === a)
  console.log(arguments[1] === b)
  a = 2
  b = 2
  console.log(arguments[0] === a)
  console.log(arguments[1] === b)
}

func()  // true false false false
```

参数的默认值未必是基本类型的值，也可以是函数引用，或者函数调用。

```javascript
let value = 5

const getValue = () => value++

const add = (one, two = getValue()) => one + two

add(1, 1)  // 2
add(1)     // 6
add(1)     // 7
```

默认参数甚至也可以引用前面的参数：

```javascript
const getValue = (value) => value + 5

const add = (one, two = getValue(one)) => one + two

add(1, 1)  // 2
add(1)     // 7
add(2)     // 9
```

但是唯独不能引用后面的参数，因为存在暂时性死区：和块级绑定类似，参数声明会创建一个新的标识符绑定，在参数初始化之前不允许被访问。参数初始化过程发生在函数调用时，无论是给参数传递一个值，还是使用参数的默认值，都属于参数初始化：

```javascript
const add = (one = two, two) => one + two

add(1, 1)          // 2
add(undefined, 1)  // 抛出错误
```

函数调用时实际发生的初始化过程如下：

```javascript
// 调用 add(1, 1)
let one = 1
let two = 1

// 调用 add(undefined, 1)
let one = two
let two = 1
```

很明显，`one` 在进行初始化时，`two` 尚未被初始化，因此它处在暂时性死区内。另外，函数参数的作用域与函数体的作用域相分离，即参数无法访问函数体内声明的变量。

前面提到，实参（arguments）的数目可以不同于形参（parameters）的数目。如果说默认参数（default parameters）让实参数目偏少时让代码书写更清晰，那么剩余参数（rest parameters）则可以让实参偏多时让代码书写更清晰。实际上，这是为了减少对 `arguments` 对象的依赖：

```javascript
function pick(obj, ...args) {
  const = result = Object.create(null)

  for (let i = 0, len = args.length; i < len; i++) {
    result[args[i]] = obj[args[i]]
  }

  return result
}
```

可以看出，剩余参数实际上表示的是其余参数组成的一个数组。剩余参数的限制条件有两个：函数只能有一个剩余参数，且必须被放在最后；不能在对象字面量的 setter 中使用，因为 setter 只能使用单个参数。

在 `Function` 构造器中也可以使用默认参数和剩余参数了（尽管几乎没人用）。

有一个与剩余参数十分类似的扩展运算符：与剩余参数相反（将多个参数合并为一个数组），扩展运算符是将一个数组分离为多个参数。实际上，它可以减少对 `apply()` 方法的依赖：

```javascript
const arr = [1, 2, 3, 4]

// apply 方法
Math.max.apply(Math, arr)

// spread 运算：单独使用
Math.max(...arr)
// spread 运算：混合参数
Math.max(5, ...arr)
```

由于 JS 中函数的形式千变万化，为了识别一个函数以方便在调用栈中调试（并不是用来获取对函数的引用），所有的函数都将有一个 `name` 属性。基本情况如下：

```javascript
function a() { /* ... */ }

var b = function() { /* ... */ }

var c = function d() { /* ... */ }

a.name               // 'a'
b.name               // 'b'
c.name               // 'd'
d.name               // 引用错误
b.bind().name        // 'bound b'
c.bind().name        // 'bound d'
new Function().name  // 'anonymous'
```

如果是对象中的 `getter` 或 `setter` 方法，则不能直接获取 `name` 属性：

```javascript
var person = {
  get age() { return this.age },
  sayName: function() { console.log(this.name) }
}

person.sayName.name  // 'sayName'
Object.getOwnPropertyDescriptor(person, 'age').get.name  // 'get age'
```

大部分函数具有两个内部方法：当函数不使用 `new` 进行调用时，`[[Call]]` 方法被执行并运行函数体内的代码；当函数使用 `new` 进行调用时，`[[Construct]]` 方法被执行并创建一个新的实例，函数体内的 `this` 指向这个新对象，并作为函数的返回值。

在 JS 中通常使用首字母大写的函数名来说明它是个构造器，但这种约定并不靠谱。ES6 通过引入 `new.target` 元属性来消除这种函数调用方面的不确定性。元属性是指非对象上的属性，用来提供关联目标的附加信息。`new.target` 只能在函数内使用：

```javascript
function Person(name) {
  if (new.target === Person) {
    this.name = name
  } else {
    throw new Error(`please use 'new' with Person`)
  }
}

var a = new Person('Tom')         // 正常
var b = Person('Tom')             // 出错
var c = Person.call(this, 'Tom')  // 出错
```

在 ES6 之前，通常不推荐在代码块中声明函数（更好的选择是函数表达式），但现在也可以使用函数声明了。如果是严格模式，函数声明将被提升至代码块顶部（包括函数名和函数体），并在代码块执行完毕后被回收。非严格模式下，函数声明将被提升至所在函数（或全局环境）的顶部。

箭头函数（arrow function）是一种新的函数风格。为了减少错误和不确定性，以及更好的引擎优化，箭头函数做了很多精简：没有 `this`、`super`、`arguments` 和 `new.target`，它们将有最靠近的非箭头函数来决定；不能更改 `this` 值；不能使用 `new` 调用；没有原型；不能有重复的具名参数。

箭头函数语法很简单：参数，箭头，函数体。如果但不同的场景下，会有不同的变体：

```javascript
// 函数体只有返回语句
const reflect = value => value

// 参数为空
const getName = () => 'es6'

// 多个参数，多条语句
const sum = (num1, num2) => {
  return num1 + num2
}

// 空函数
const empty = () => {}

// 返回对象
const getObject = (id, name) => ({ id: id, name: name })
```

立即调用函数（IIFE）有两种括号包裹的方法。如果用箭头函数来写，将只有一种写法：

```javascript
// 可以包裹后调用，或者调用后包裹
(function() { /* ... */ })()
(function() { /* ... */ }())

// 只能包裹后调用
(() => { /* ... */ })()
```

没有 `this` 绑定其实可以避免很多错误。比如绑定事件监听时，回调函数中的 `this` 会被绑定在事件的目标对象上。如果用箭头函数，就能避免这种错误：

```javascript
const handler = {
  id: '123'

  init: function() {
    document.addEventListener('click', () => {
      this.dosomething(event.type)
    }, false)
  }

  doSomething: function(type) {
    console.log(`handling ${type} for ${this.id}`)
  }
}
```

由于箭头函数本身没有 `this`，因此会沿着作用域链查找，使用最近的非箭头函数的 `this` 值（`arguments` 对象同理）。因此 `this` 被绑定在 `handler` 对象上，而非 `document` 对象上。另外，尽管 `doSomething` 方法没有返回值，但由于函数体内只有一条语句，因此花括号其实可以省略。

虽然箭头函数与普通函数有很多区别，但本质上仍是个函数。因此，也具有 `name` 属性；也能被 `typeof` 和 `instanceof` 识别；也能使用 `call()`、`apply()` 和 `bind()`（但不影响 `this` 绑定）。箭头函数最大的用途在于替代匿名函数，尤其是回调函数，比如各种数组方法。

最后，JS 引擎对函数进行了尾调用优化。像普通的函数调用一样，尾调用也会创建新的栈帧（stack frame）然后压入调用栈（call stack）中。由于所有的栈帧都被保留在内存中，如果调用栈过大会导致栈溢出，尤其是尾递归的情况下。

严格模式下，引擎可以进行尾调用优化：发生尾调用时，不再创建新的栈帧，而是清空当前栈帧后再次使用它。但尾调用优化的条件也十分苛刻：

```javascript
// 未优化：尾调用的结果需要作为当前函数的返回值
function one() { another() }

// 未优化：尾调用返回后不能有额外操作
function one() { return 1 + another() }

// 未优化：调用不在尾部
function one() {
  var result = another()
  return result
}

// 未优化：尾调用函数不能是闭包（不能引用当前栈帧中的变量）
function one() {
  var val = 1
  var another = () => val
  return another()
}
```

只有真正的尾调用才会被优化：

```javascript
// 被优化
function one() { return another() }
```

### Object

ES6 明确了对象的类别：

- 普通对象：拥有 JS 对象所有默认的内部行为
- 奇异对象：其内部行为在某些方面不同于默认行为
- 标准对象：在 ES6 中被定义的对象，可以是普通的或奇异的
- 内置对象：由 JS 运行环境提供的对象，标准对象都是内置的

在对象的字面量语法中，对象的属性和方法可以简写。其中，属性的简写本质上是将作用域内的变量赋值给对象的同名属性：

```javascript
// 完整写法：
function createPerson(name) {
  return {
    name: name,
    sayName: function() {
      console.log(this.name)
    }
  }
}
// 简写形式：
function createPerson(name) {
  return {
    name,
    sayName() {
      console.log(this.name)
    }
  }
}
```

对于对象字面量中的属性名，如果是合法标识符（仅含字母、数字和下划线）则可以直接书写；如果有特殊字符则需要以字符串字面量形式书写。如果属性名事先并不明确，需要通过计算获得，或者被包含在变量中，则可以使用需计算属性名：

```javascript
const suffix = ' name'

const person = {
  age: 18,
  'first name': 'Tim',
  ['last' + suffix]: 'Cook'
}
```

为了防止类型转换带来的困扰，在做值比较时通常会使用严格相等运算符（`===`）。但它依然有两个残留缺陷，而 `Object.is()` 方法完善了它：

```javascript
Object.is(+0, -0)    // false
Object.is(NaN, NaN)  // true
```

如果一个对象要获取另一个对象的属性，除了使用继承，还可以使用混入（Mixin） 模式，将供应者的自有属性浅复制到接收者：

```javascript
function mixin(receiver, supplier) {
  Object.keys(supplier).forEach((key) => {
    receiver[key] = supplier[key]
  })
  return receiver
}
```

ES6 使用 `Object.assign()` 方法接收了这个流行的模式。区别是该方法可以有很多个供应者，这也意味着靠后的供应者的同名属性会覆盖靠前的供应者。此外，由于此方法的原理是使用赋值运算符，因此供应者的访问器属性会转变为数据属性。

自有属性的枚举顺序此前并未纳入规范，但很多方法确实涉及顺序（比如 `Object.assign()`）。现在规范定义的顺序是：首先 number 类型的键升序排列；然后 string 类型的键按被添加到对象的顺序排列；最后是 symbol 类型的键按被添加到对象的顺序排列。

对象的原型是在创建该对象时被指定的，即通过构造器或者 `Object.create()`）完成原型的初始化。对象的原型被存储在内部属性 `[[Prototype]]` 上，可以通过 `Object.getPrototypeOf()` 来获取这个属性值。现在也可以通过 `Object.setPrototypeOf()` 来修改这个属性值了。

针对原型的另一项易用性改进是引入了 `super` 引用，不过它只能在上面提及的对象方法的简写形式中使用。`super` 是指向当前对象原型的指针，或者说是 `Object.getPrototypeOf(this)` 的简写形式。`super` 的最大用途是调用原型上的方法，同时保证该方法内部的 `this` 值被正确绑定在原型上。它在继承中十分有用：

```javascript
const person = {
  say() { return 'hello' }
}

const friend = {
  say() { return super.say() + ', world!' }
}

Object.setPrototypeOf(friend, person)
const relative = Object.create(friend)

person.say()    // 'hello'
friend.say()    // 'hello, world!'
relative.say()  // 'hello, world!'
```

“方法”一词也被 ES6 明确了下来。此前方法是指对象的函数属性（而非数据属性），如今的定义是拥有 `[[HomeObject]]` 内部属性的函数，该属性指向该方法所属的对象。在使用 `super` 引用时会使用到该属性，比如上例中的 `friend.say()` 调用时涉及到的 `super.say()` 调用，会有如下步骤：

- 获取 `friend.say` 的 `[[HomeObject]]` 值，即 friend
- 使用 `Object.getPrototypeOf(friend)` 获取方法所属对象的原型，即 person
- 在原型中查找同名方法，创建 `this` 绑定后调用它，即执行 `person.say.call(this)`

另外，对象字面量中的重复属性名不会再抛错，而是直接后者覆盖前者。

### Destructuring

对象和数组的字面量是 JS 中最常用的表示法，而解构可以使它们分解为更小的部分，从而简化数据的提取过程。

对象的解构赋值本质上是将对象的同名属性赋值给本地变量。可以用在任何期望有个值的位置：

```javascript
const person = {
  name: 'Cook',
  age: 40
}

// 用于变量声明
let { name, age } = person

// 用于变量赋值
({ name, age }) = person

// 用于传递参数
someFunc({ name, age } = person)
```

如果没有找到同名属性，左侧的本地变量将被赋值为 `undefined`。此时可以选择使用一个默认值，该默认值会在对应属性缺失（或对应属性值是 `undefined`）时生效：

```javascript
const person = { name: 'Cook' }

const { name, age = 40 } = person
```

解构赋值的属性和变量是同名的，但其实也可以使用不同的变量名。冒号前的标识符表示赋值位置，冒号后的标识符表示赋值目标：

```javascript
const person = { name: 'Cook' }

// 使用不同的变量名
const { name: localName, age: localAge} = person

// 使用不同的变量名，同时设定默认值
const { name: localName, age: localAge = 40} = person

console.log(localName)  // 'Cook'
console.log(localAge)   // 40
```

最复杂也最强大的结构赋值是嵌套语法，它可以深入到对象的内部结构中去提取数据。需要注意的是，必须区分赋值的位置和目标。解构语法中，冒号前的标识符表示位置，冒号后的标识符表示目标，而花括号表示目标在下一层：

```javascript
const profile = {
  name: 'Cook',
  location: {
    country: 'US',
    city: 'California'
  }
}

// 将 profile.location.city 赋值给 place
const { location: { city: place } } = profile
```

需要注意的是，解构赋值表达式的右侧不允许是 `null` 或 `undefined`，否则会报错。左侧的赋值目标也最好不要是空的花括号，尽管不会报错（但什么都不会发生）。

数组解构的语法与对象解构十分类似。不过因为数组没有具名属性，所以会直接作用在位置上，而变量名则没有要求。可以借助占位的方式来提取相应位置的值：

```javascript
const fruit = ['apple', 'banana', 'orange']

// 将 'banana' 赋值给 lunch
let [, lunch] = fruit

// 将 'apple' 赋值给 lunch
[lunch] = fruit
```

排序算法中经常用到互换变量值。数组解构可以在避免使用临时变量的情况下实现，原理是等号右侧创建了临时数组：

```javascript
let a = 1
let b = 2

[a, b] = [b, a]
```

默认值和嵌套也同样可以用在数组的解构赋值中。其中方括号和逗号用于定位：

```javascript
const arr = ['one', ['two', 'three'], 'four']

// 将 'two' 赋值给 num1，将 'five' 赋值给 num2
const [, [num1], , num2 = 'five'] = arr
```

数组解构中可以使用一个叫做剩余项（类似于剩余参数）的方式，可以将某位置后的所有剩余项目赋值给一个变量。主要功能是取出特定位置的项后，保留剩余的项：

```javascript
let arr = [1, 2, 3, 4]

const [, item, ...restItem] = arr

console.log(item)      // 2
console.log(restItem)  // [3, 4]
```

需要注意的是，剩余项必须是解构语法中的最后一项（即之后不允许有逗号出现）。剩余项的常见用途是拷贝数组：

```javascript
const arr = [1, 2, 3, 4]

// 多种克隆数组的方式
const clone1 = arr.slice()
const clone2 = arr.concat()
const [...clone3] = arr
```

对象和数组混合而成的解构中，可以使用混合解构。这对于提取 JSON 数据来说会十分方便：

```javascript
const profile = {
  name: 'Cook',
  location: {
    country: 'US',
    city: 'California'
  },
  friends: ['Jobs', 'Ive']
}

let {
  location: { city: place },
  friends: [, designer]
} = profile

console.log(place)     // 'California'
console.log(designer)  // 'Ive'
```

当函数需要接收大量可选参数时，可以使用参数解构。这在此前会十分麻烦，而且无法反映具体的参数项。使用参数解构后，就可以分离必需参数和可选参数，并且明确了可选参数的项。

```javascript
setCookie('name', 'Cook', {
  secure: false,
  expires: 10000
})

// 普通方式
function setCookie(key, value, options) {
  // 接收 options
  opts = options || {}

  let secure = opts.secure
  let domain = opts.domain
  let path = opts.path
  let expires = opts.expires

  // ...
}

// 参数解构
function setCookie(key, value, { secure, domain, path, expires }) {
  // ...
}
```

如果调用时未提供可选参数的某项，将被设定为 `undefined`（在解构语法的等号右侧没有查找到对应项）；如果调用时未提供可选参数，将会报错（解构语法的等号右侧不能是 `null` 或 `undefined`）。因此最佳方案是提供完整的默认值：

```javascript
// 包含完整默认值的参数解构
function setCookie(key, value,
  {
    secure = false,
    domain = '/',
    path = 'es6.com',
    expires = 999999999
  } = {}
) {
  // ...
}
```

### Symbol and Symbol Property

`Symbol` 是继 `undefined`、`null`、`Boolean`、`Number` 和 `String` 之后，一种新的基本类型。它的早期设计意图是为了创建非字符串类型的属性名，因为字符串属性名可以被轻易访问，缺乏私有性。使用符号可以创建不可枚举的属性，如果不引用符号将无法访问对应的属性。

尽管符号值是基本类型，但它没有字面量形式，只能使用全局的 `Sysbol()` 函数来创建符号值。创建的符号值可以交由一个变量来引用自身，并且可以附带描述信息以便调试：

```javascript
const firstName = Symbol('first name')

console.log(firstName)         // 'Symbol(first name)'
console.log(typeof firstName)  // 'symbol'
```

符号值可以在任何能使用需计算属性名的场合使用：

```javascript
const firstName = Symbol('first name')

const person = {
  [firstName]: 'es5'
}

Object.defineProperties(person, {
  [firstName]: {
    value: 'es6',
    writable: false
  }
})

console.log(person[firstName])  // 'es6'
```

如果需要在不同的对象中使用相同的符号属性，就要通过共享符号值来创建唯一的标识符。原理上，是在全局符号注册表中按照给定的键创建一个符号值：

```javascript
let uid1 = Symbol.for('uid')
let uid2 = Symbol.for('uid')

console.log(uid1 === uid2)        // true
console.log(Symbol.keyFor(uid1))  // 'uid'
console.log(Symbol.keyFor(uid2))  // 'uid'
```

JS 中的类型之间可以相互转换，但是符号类型与其他类型缺少合理的等价，因此一般不进行类型转化。例子中的输出是个符号值的字符串形式，实际上是调用了 `String()` 方法来展示其描述信息。另外，在逻辑运算中会被当做 `true`。

符号属性无法通过普通的方法检索，但可以通过另一个方法来检索它：

```javascript
const uid = Symbol.for('uid')
const obj = {
  age: 20,
  name: 'es6',
  [uid]: '123'
}
Object.defineProperty(obj, 'name', { enumerable: false })

console.log(Object.keys(obj))                   // ['age']
console.log(Object.getOwnPropertyNames(obj))    // ['age', 'name']
console.log(Object.getOwnPropertySymbols(obj))  // [Symbol(uid)]
```

ES6 将一些语言层面的内部操作暴露为一些知名符号，每个知名符号都对应了全局 `Symbol` 对象的一个属性。使用知名符号可以完成一些内部操作，比如：

```javascript
Array[Symbol.hasInstance](obj)  // 等价于 obj instanceof Array
```

对象在进行隐式转换时会被转换为基本类型值。如果是数值模式，则会先尝试 `valueOf()` 方法，如果不能返回一个基本类型值就再尝试 `toString()` 方法，依然不行就抛错。如果是字符串模式，两个方法的尝试顺序是相反的。除了 `Date` 类型，所有的常规对象默认采用数值模式。ES6 提供的 `Symbol.toPrimitive()` 方法则可以重写转换行为。

知名符号扩展了对常规对象的操作，但一般很少用到。

### Set and Map

Set 是不包含重复值的集合，通常我们会用它来检查某个值是否存在；Map 是键值对的集合，通常我们会用它来缓存数据，以便后续的快速访问。但过去 JS 的集合类型只有数组，它甚至还承担了队列和栈的功能。

ES5 中需要模拟 Set 和 Map：

```javascript
let set = Object.create(null)
set.foo = true
if (set.foo) {
 // 检查属性存在性后的操作
}

let map = Object.create(null)
map.foo = 'bar'
let value = map.foo
// 提取属性值后的操作
```

但存在的问题有很多。对于 `set` 来说， `if (set.foo)` 其实也无法检查属性的存在性，只能检查属性是否是非零值，如果使用 `in` 操作符，会检索它的原型（当然例子中使用 `null` 避开了这个问题）。对于 `map` 来说，由于对象的键只能是字符串，`map[5]` 会被转化为 `map['5']`，如果是把对象作为键，会被转化为 `map['[object Object]']`。

好在 ES6 提供了 Set 和 Map。

Set 是没有重复值的有序列表。其构造器可以接收任意可迭代对象作为参数：

```javascript
let set = new Set([1, 1, 2])  // set 初始化为：1, 2

set.add(3)
set.has(4)                    // false
set.delete(3)
set.size                      // 2
set.clear()
```

Set 使用 `forEach()` 方法可以进行迭代。像数组一样，第一个参数为回调函数，第二个参数是可选的上下文。但为了与 Array 和 Map 保持一致，回调函数的三个参数分别是：`value`、`key`、`owner`。

Set 擅长检查值的存在性（`has()` 方法使用 `Object.is()` 方法来比较键），但无法直接访问值。如果有此需求，可以用扩展运算符最好将其转化为数组，因为扩展运算符可以作用于任何可迭代的对象：

```javascript
let set = new Set([1, 1, 2, 3])

let arr = [...set]
```

注意到 Set 是不含重复值的，因此也可以借助一个临时的 Set 来进行数组去重：

```javascript
let arr = [1, 1, 2, 3]

let noDuplicates = [...new Set(arr)]
```

Set 的一个问题在于，如果存储了一个对象，它将持有一份引用。即使原有对象的引用已经被消除，它依然会持有该对象的引用。这份唯一的引用阻止了垃圾回收，这会造成一些问题。因此 ES6 提供了 WeakSet。它具有 `add()`、`delete()` 和 `has()` 方法，但只接收引用类型的值，并且对它们是弱引用：

```javascript
let set = new WeakSet()
let key = {}

set.add(key)

key = null
```

此时 `set` 将不再持有 `key`（如果是通过 `Set()` 构造将依然持有），尽管无法验证：因为已经没有对象的引用了，`has()` 方法无法使用，而且 WeakSet 也不具有 `size` 属性。

Map 是键值对的有序列表。其构造器与 Set 一样，也可以接收一个数组，但是数组的每一项都是一个数组。这是因为它的键和值都可以是任意类型的值，因此需要将每一项存在数组中（如果是存在对象中，键会被强制转换）：

```javascript
let map = new Map([['name', 'es6'], ['age', 20]])

map.forEach((value, key, owner) => {
  // 按属性被添加的顺序
})

map.set('city', 'Shanghai')
map.has('city')              // true
map.get('city')              // 'Shanghai'
map.delete('name')
map.size                     // 2
map.clear()
```

可以看出，Map 的 `forEach()` 方法和 Array 与 Set 保持一致。另外，也可以发现 Map 和 Set 实际上共享了许多方法和属性：`has()`、`delete()`、`clear()` 和 `size`。

与 WeakSet 相对应，WeakMap 也是具备弱引用能力的类型。但是只有它的键是弱引用，它的值如果是对象，依然会阻止垃圾回收，即使该对象的其他引用都被移除。

WeakMap 是键值对的无序列表，其键是非空对象，其值是任意类型。它具有 `set()`、`get()`、`delete()` 和 `has()` 方法。类似于 WeakSet，也无法确认 WeakMap 是否为空：其他引用被移除后，对键的引用不再有残留，`get()` 和 `has()` 方法无法使用，而且也不具有 `size` 属性。同 WeakSet 一样，由于没必要枚举，WeakMap 也没有 `clear()` 方法。

WeakSet 和 WeakMap 的最大用途是在内存管理方面，但它们在数据的类型和访问性上有一定的限制。


### Iterator and Generator

`for` 循环是容易出错的语法，因为需要追踪索引。使用迭代器（iterator）可以降低错误：

```javascript
function createIterator(items) {
  let i = 0
  return {
    next: function() {
      let done = (i >= items.length)
      let value = done ? undefined : items[i++]
      return {
        done: done,
        value: value
      }
    }
  }
}

const iterator = createIterator([1, 2, 3])
console.log(iterator.next());           // { value: 1, done: false }
console.log(iterator.next());           // { value: 2, done: false }
console.log(iterator.next());           // { value: 3, done: false }
console.log(iterator.next());           // { value: undefined, done: true }
```

然而迭代器写起来很麻烦。因此又有了能返回迭代器的生成器（generator）。生成器返回的迭代器在调用 `next()` 时，遇到 `yield` 会停止运行，再次执行 `next()` 依然会执行到 `yield` 后停止：

```javascript
function* createIterator(items) {
  for (let i = 0; i < items.length; i++) {
    yield items[i]
  }
}

const iterator = createIterator([1, 2, 3])
console.log(iterator.next());           // { value: 1, done: false }
console.log(iterator.next());           // { value: 2, done: false }
console.log(iterator.next());           // { value: 3, done: false }
console.log(iterator.next());           // { value: undefined, done: true }
```

`*` 需要放在 `function` 和圆括号之间，无法用于箭头函数，但可以用于对象的方法。而 `yield` 关键字只能严格用在生成器函数内部，即使是生成器内嵌套的函数也无法使用。`yield` 这种无法穿越函数边界的特性与 `return` 很像：函数执行中遇到 `return` 并不会返回到包含函数，而是完全退出。

ES6 中的 Array、Set、Map 和 String 都是可迭代对象（当然生成器创建的迭代器本身也是一个可迭代对象），可迭代对象包含了 `Symbol.iterator` 属性，该属性存放了能够返回迭代器的方法。

`for-of` 循环默认调用可迭代对象的 `Symbol.iterator` 方法，并返回一个迭代器。每次循环过程会默认调用迭代器的 `next()` 方法，并将 `next().value` 存储在给定变量中。循环会默认持续到 `next().done` 变为 `true` 为止：

```javascript
const items = [1, 2, 3]

for (let item of items) {
  console.log(item)
}
```

如果是没有迭代器的不可迭代对象，或者想更改迭代器的可迭代对象，则可以直接针对 `[Symbol.iterator]` 属性。下例的 `obj` 是个对象，不具有迭代器，但可以借用 `obj.items` 的迭代器：

```javascript
const obj = {
  items: [1, 2, 3],
  *[Symbol.iterator]() {
    for (let item of items) {
      yield item
    }
  }
}
```

ES6 的集合对象（Array、Set、Map）具备三个可以生成迭代器的方法，分别用于迭代不同的内容：

- `entries()`：返回一个处理键值对的迭代器
- `values()`：返回一个处理值的迭代器
- `keys()`：返回一个处理键的迭代器

而默认迭代器会反映了对象的初始化过程。对于 Array 来说：

```javascript
const arr = ['a', 'b', 'c']

for (let item of arr) {
  console.log(item) // 依次打印：'a' 'b' 'c'
}

for (let item of arr.entries()) {
  console.log(item) // 依次打印：[0, 'a'] [0, 'b'] [0, 'c']
}

for (let item of arr.values()) {
  console.log(item) // 依次打印：'a' 'b' 'c'
}

for (let item of arr.keys()) {
  console.log(item) // 依次打印：1 2 3
}
```

Weak Set 与 Weak Map 没有内置任何一种迭代器（默认迭代器也没有）。原因是它们使用了弱引用，无法获知集合内部有多少值，因此也无法迭代它们。

数组结构在 `for-of` 循环中也可以使用：

```javascript
const map = new Map()
map.set('name', 'es6')
map.set('age', 20)

for (let [key, value] of map) { /* ... */ }

for (let [key, value] of map.entries()) { /* ... */ }
```

String 在很多行为上与 Array 相似。而且 `for-of` 循环支持 Unicode 双字节字符：

```javascript
const msg = 'a𠮷b'

for (let char of msg) {
  console.log(char)  // 依次打印：'a' '𠮷' 'b'
}
```

NodeList 与 Array 也很相似，但仅局限于：使用 `length` 属性表示项的数量，使用方括号表示法访问各个项。之前是无法直接被迭代的 NodeList 现在也可以了：

```javascript
const divs = document.getElementsByTagName('div')

for (let div of divs) { /* ... */ }
```

由于扩展运算符的本质是调用可迭代对象的迭代器，因此所有的可迭代对象（Array、Set、Map、String、NodeList），都可以转换为数组：

```javascript
const arr = [1, 2]
const set = new Set([3, 4])
const map = new Map([['5', 5], [6, '6']])
const str = '78'

console.log([0, ...arr, ...set, ...map, ...str])
//  [0, 1, 2, 3, 4, ['5', 5], [6, '6'], '7', '8']
```

除了 `for-of` 循环和扩展运算符，迭代器和生成器还有很多复杂的能力。比如 `iterator.next()` 实际上可以传参；`iterator.throw()` 可以抛错；甚至还可以尝试 `return` 语句和委托功能。

传统的异步操作是基于回调的：

```javascript
const fs = require('fs')

fs.readFile('config.json', (err, data) => {
  if (err) throw err
  doSomething(data)
  console.log('done')
})
```

但如果存在嵌套的回调（需顺序执行的异步任务），情况将会变得复杂。使用生成器构建异步 task runner 可以简化这种情况。

### Class

 由于很多开发者喜欢模拟类，ES6 最终提供了类的语法糖。

### Array

此前创建一个数组有两种方式：字面量，构造器。ES6 增加了创建数组的新方法：`Array.of()` 和 `Array.from()`。

`Array.of()` 是为了规避构造器的意外情况：

```javascript
// 使用 constructor
let items = new Array(1)  // 生成数组：[]
items = new Array('1')    // 生成数组：['1']
items = new Array(1, 2)    // 生成数组：[1, 2]

// 使用 Array.of()
let items = Array.of(1)  // 生成数组：[1]
items = Array.of('1')    // 生成数组：['1']
items = Array.of(1, 2)    // 生成数组：[1, 2]
```

`Array.from()` 是为了简化把非数组对象转化为数组的方式：

```javascript
// 过去
const args = Array.prototype.slice.call(arrayLike)

// 现在
const args = Array.from(arrayLike)
```

其实 `Array.from()` 有三个参数。第一个是类数组对象（或者可迭代对象），第二个是映射方法，第三个是映射方法的上下文：

```javascript
const helper = {
  diff: 1,
  add(val) {
    return val + this.diff
  }
}

function translate() {
  return Array.from(arguments, helper.add, helper)
}

console.log(translate(1, 2, 3))  // 2, 3, 4
```

ES6 为数组新增的 `find()` 和 `findIndex()` 可以查找满足特定条件的元素，条件由回调参数指定：

```javascript
const items = [10, 20, 30]

console.log(items.find(n => n > 10))       // 20
console.log(items.findIndex(n => n > 10))  // 1
```

当然查找特定的元素值最好还是用 `indexOf()` 和 `lastIndexOf()`。

ES6 为数组新增的 `fill()` 可以用特定值来填充数组的一个或多个元素，参数分为是填充值、起始和结束位置：

```javascript
const items = [1, 2, 3, 4, 5]

items.fill(0)        // [0, 0, 0, 0, 0]
items.fill(3, 1)     // [0, 3, 3, 3, 3]
items.fill(5, 2, 4)  // [0, 3, 5, 5, 3]
```

ES6 为数组新增的 `copyWithin` 可以复制特定范围的数据并覆盖原数组，参数分为是覆盖的起始位置、复制的起始和结束位置：

```javascript
const items = [1, 2, 3, 4, 5, 6]

items.copyWithin(1, 0)        // [1, 1, 2, 3, 4, 5]
items.copyWithin(3, 1, 2)     // [1, 1, 2, 1, 4, 5]
```

JS 使用 64 位来存储一个数值的浮点数形式。这种形式同时用来表示整数和浮点数，存在位的浪费。如果数值发生变化，会频繁的发生整数与浮点数之间的格式转换。而类型化数组允许存储和操作 8 种数值类型：`int8`、`int16`、`int32`、`uint8`、`uint16`、`uint32`、`float32`、`float64`。

类型化数组是基于数组缓冲区（内存中包含一定数量字节的区域）构建的，使用类型化数组需要先创建一个数组缓冲区来存储数据：

```javascript
let buffer1 = new ArrayBuffer(10)  // 分配 10 个字节给 buffer1
console.log(buffer1.byteLength)    // 10

let buffer2 = buffer1.slice(4, 6)  // 提取 buffer1 的字节给 buffer2
console.log(buffer2.byteLength)    // 2
```

ArrayBuffer 代表了一块内存区域，view 是操作这块区域的接口。其中 DateView 是 ArrayBuffer 的通用视图，可以对 8 种数值类型进行操作：

```javascript
let buffer = new ArrayBuffer(10)

let view1 = new DataView(buffer)        // view1 可操作 buffer 的所有字节
let view2 = new DataView(buffer, 4, 2)  // view2 可操作 buffer 位置 4 开始的 2 个字节

console.log(view1.buffer === buffer)  // true
console.log(view2.buffer === buffer)  // true
console.log(view1.byteOffset)         // 0
console.log(view2.byteOffset)         // 4
console.log(view1.byteLength)         // 10
console.log(view2.byteLength)         // 2
```

视图创建后，就可以进行数据读写了。完整操作如下：

```javascript
let buffer = new ArrayBuffer(2)
let view = new DataView(buffer)

view.setInt8(0, 7)  // 0000 0111 0000 0000
view.getInt8(0)     // 7
view.getInt16(0)    // 1792
```

可以看出，一个视图可以有多种类型的书写读写。如果只针对一种数值类型，就可以选择特定类型视图，即类型化数组：

```javascript
// 使用 ArrayBuffer 创建类型化数组
let view1 = new Int16Array(new ArrayBuffer(2))
console.log(view1.length)      // 1
console.log(view1.byteLength)  // 2

// 使用 constructor 创建类型化数组
let view2 = new Int16Array(2)
console.log(view2.length)      // 2
console.log(view2.byteLength)  // 4
```

除了上述两种方式，还可以通过给构造器传递一个对象参数来创建类型化数组。可以传递的对象类型有：类型化数组、可迭代对象、数组以及类数组对象。

类型化数组与数组有很多相同点，比如具有 `length` 属性（但是只能读不能写），还可以用数值索引访问对应项。此外，还有很多相同的方法，也有相同的迭代器。但类型化数组并不是数组，它只能存储数值，而且长度固定无法伸缩，因此也不能使用会改变其长度的方法。

此外，类型化数组具有两个独有的方法：`set()` 方法从数组复制元素到类型化数组，`subarray()` 方法从自身提取一部分作为新的类型化数组。

### Promise and Asynchronous Programming

### Proxy and Reflection

目前用不到，暂时先跳过。（感觉比 Symbol 还麻烦）

### Module

JS 最糟糕的地方是所有的脚本文件都在全局作用域被共享。其他语言通常使用包（package）来确定代码的作用域，而 ES6 选择了模块（module）。

模块深刻的改变了 JS 代码的加载和执行，并具备了按需导入和导出代码的能力。主要有以下特点：

- 模块的代码自动运行在严格模式下，且无法更改
- 模块的顶级作用域的 `this` 值为 `undefined`
- 在模块顶级作用域创建的变量，不会被自动添加到共享的全局作用域
- 模块可以导出内容供其他模块使用，或者导入内容供自己使用

模块语法的基础是导出和导入，使用的关键字是 `export` 和 `import`。

导出语法既可以用于导出声明（变量、函数，或者类），也可以用于导出引用：

```javascript
// 导出声明
export let name = 'es6'
export function sum(n1, n2) { return n1 + n2 }
export class Rectangle { /* ... */ }

// 导出引用
function multiply(n1, n2) { return n1 * n2 }
export { multiply }
```

导入语法有两个部分：要导入的绑定和要导入的模块。

导入绑定的形式类似于对象解构。被导入的绑定表现的就像使用了 `const` 定义：不能导入另一个同名绑定，不能定义另一个同名变量，导入前不能使用这个标识符，导入后不能修改它的值。

导入模块的形式的是个模块说明符。如果是浏览器中导入模块，说明符必须包含文件扩展名；而 node.js 中有扩展名表示本地文件，无扩展名表示包（package）。另外需要显式的使用 `/`、`./` 或 `../` 来保证 web 浏览器和 node.js 的兼容性，但在 script 标签的 `src` 属性中并不需要（因为已经明确了环境）。

```javascript
// 单个导入
import { sum } from './demo.js'

// 多个导入
import { sum, multiply } from './demo.js'

// 完全导入
import * as demo from './demo.js'
```

完全导入后 `demo` 将作为 `demo.js` 所有导出成员的命名空间对象。如果一个模块被多次导入（无论是在同一个模块中被多次导入，还是在多个模块中被多次导入），实际的导入只会被执行一次，该模块被实例化后保留在内存中，供所有的 `import` 引用。

模块语法是需要被静态分析的。因此 `export` 和 `import` 必须在语句和表达式的外部使用，即只能在模块的顶级作用域内使用。

正常的变量引用通常是原始绑定，支持读和写。但 `import` 只创建只读绑定，不能修改绑定值。也就是说，导入的变量是一份只读引用，导出模块的任何变动都会反映到导入模块中。这种只读的绑定关系使用不会被破坏，即使使用特殊的技巧，比如导出模块为：

```javascript
export var name = 'es6'
export function setName(val) {
  name = val
}
```

导入模块为：

```javascript
import { name, setName } from './demo.js'

setName('es5')  // name 从 'es6' 变为了 'es5'
name = 'es4'    // 抛出错误
```

虽然导入模块不能修改绑定值（正如最后的语句抛错了），但在导出模块中确实可以的。`setName('es5')` 的实际执行地点是导出模块，并将这种变化反映到导入模块：导入模块的 `name` 作为一个只读引用，会忠实的反映导出模块的 `name` 的值。

尽管它们的名称相同，但确实是两个不同的 `name`。这也引出了另一个主题，导入和导出的重命名。

如果在导出时更改名称：

```javascript
let sum = (n1, n2) => n1 + n2
export { sum as add }  // 本地名称是 sum，导出名称是 add
```

那么导入这个模块时必须使用 `add`，除非导入也更改名称：

```javascript
import { add as sum } from './demo.js'  // 导入名称是 add，本地名称是 sum
sum(1, 1)
```

导入和导出也可以使用默认值。导出的默认值只能设置一个：

```javascript
// 声明的默认导出 (无需标识符)
export default function(n1, n2) { return n1 + n2}

// 引用的默认导出
export default sum

// 重命名的默认导出
export { sum as default }
```

导入时如果不写花括号就是使用默认导入，即导入一个被默认导出的成员：

```javascript
// 导入默认值
import sum from './demo.js'

// 混合导入默认值和其他值 (默认值必须在前)
import sum, { other } from './demo.js'

// 混合导入时对默认值重命名
import { default as sum, other } from './demo.js'
```

其实默认值的最大用途，在于提供一种连接形式：模块的提供者将常用成员默认导出，而模块的使用者又无需知道成员的精确名称。

被导入的模块也可以被再次导出，此时需要在使用 `export` 的同时使用 `from`。实际原理是进入指定模块查找标识符定义，然后将其导出。多个模块互相依赖时，导入再导出可以在语法层面简化依赖关系。

```javascript
import { sum } from './demo.js'

// 普通的导入再导出
export { sum } from './demo.js'

// 重命名的导入再导出
export { sum as add } from './demo.js'

// 完全的导入再导出（若 demo.js 含默认值，意味着当前模块不能再有默认导出）
export * from './demo.js'
```

模块语法的导出和导入，可以避免模块自身被其他模块修改，因为模块内的变量不会被自动纳入全局环境。尽管保护了自己，但无法保护全局：任何一个模块都可以直接修改全局环境，并且被其他模块直接感知到。

但有时候这也是需求：比如一个 JS 文件的目的就是通过扩展原生对象（比如 polyfill 和 shim）。如果它不含导入和导出，也没有作为脚本被加入到 html 中，那么使用它的唯一方式就是直接被导入：

```javascript
import './polyfill.js'
```

通过这种简单导入方式，就可以直接影响当前模块对全局的使用，并且所有的模块都能感知到全局的变化，但又不会再当前模块中引入任何不必要的绑定。

ES6 将更多的精力放在了模块的语法层面，但并未定义模块的加载机制，这将由 web 浏览器和 node.js 自行决定。

和使用脚本的方式一样，web 浏览器也是在 script 标签中使用模块的：

```html
<!-- 内联模块 -->
<script type="module">
// ...
</script>

<!-- 文件模块 -->
<script type="module" src="./demo.js"></script>
```

真正复杂的地方是加载的顺序和执行的顺序。复杂的原因是模块默认具备 `defer` 属性，并且模块可能会含有 `import` 语句。HTML 解析过程是按顺序的：

如果 HTML 解析到文件模块，会立即加载文件（从网络或缓存）并解析文件，如果解析到 `import` 语句，就继续加载语句想要导入的文件。需要注意的是，加载和解析是递归的，因为 `import` 语句导入的模块可能仍有 `import` 语句。

如果 HTML 解析到内联模块，就同理进行递归的加载和解析，区别在于少了第一次的加载。

当所有的模块（内联模块和文件模块）及其递归导入的资源全部被加载并解析完毕，并且整个 HTML 文档的解析也完毕后，才会开始脚本的执行过程。代码的执行是顺序的，可以理解为将所有加载到的代码拼接为内联形式后的代码。

相对于 `defer` 属性带来的特性（ 立即加载和解析，但文档解析完才执行），`async` 属性的特性是立即加载和解析，并且解析完就执行。如果将 `async` 用在模块上，带来的效果是：递归的加载和解析完所有的依赖，然后立即执行。

另外，模块也可以作为 worker 加载。

### Minor Improvements

JS 统一使用 number 类型来表示整型和浮点型，但它们的存储方式是不同的。ES6 的 `Number.isInteger()` 方法可判断一个值是否能表示整型，`Number.isSafeInteger()` 方法可判断一个值是否是安全整型（未超过安全边界）：

```javascript
console.log(Number.isInteger(25.0))                 // true
console.log(Number.isSafeInteger(Math.pow(2, 53)))  // false
```

`Math` 对象扩充很多方法，比如识别数值符号的 `Math.sign(x)` 和移除浮点数小数部分的 `Math.trunc()`：

```javascript
console.log(Math.sign(-10))    // -1
console.log(Math.trunc(9.99))  // 9
```

Unicode 字符可以作为合法的标识符了：

```javascript
let \u0061 = 'hello'
let \u{62} = 'world'

console.log(a)  // hello
console.log(b)  // world
```

对象可以使用 `__proto__` 来访问或设置该对象的原型，实际上类似于 `Object.getPrototypeOf()` 和 `Object.setPrototypeOf()` 两个方法的效果。

### ES7

ES7 有三项更新：

第一项是幂运算符 `**`，它的优先级是二元运算符中最高的。左侧是底数，右侧是指数：

```javascript
console.log((-5) ** 2 === Math.pow(-5, 2))  // true
```

第二项是 `Array.prototype.includes()` 方法，用来检查数组是否包含某项。它类似于 `String.prototype.includes()` 方法，同样是传入待检查项和起始位置：

```javascript
const arr = [1, 2, 3, 4]

console.log(arr.includes(1, 2))  // false
```

第三项是一个语法错误：只有是简单参数列表时，函数作用域内才能使用 `'use strict'`；如果是参数列表中存在解构或者默认值，将会报错。

### 参考
1.[Understanding ECMAScript 6 - Nicholas C. Zakas | translated by sagittarius-rev](https://sagittarius-rev.gitbooks.io/understanding-ecmascript-6-zh-ver/content/)
