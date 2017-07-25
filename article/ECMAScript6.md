## ECMAScript 6
> ES6 跟 ES5 比起来就像是另一门语言

尼古拉斯的 Understanding ES6 都快第二版了，但国内第一版还没出来。那就不等啦。

笔记目录

1. [Block Binding](#block-binding)
2. [String and Regular Expression](#string-and-regular-expression)
3. [Function](#function)
4. [Object](#object)
5. [Destructuring](#destructuring)

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
