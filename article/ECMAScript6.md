## ECMAScript 6
> ES6 跟 ES5 比起来就像是另一门语言

尼古拉斯的 Understanding ES6 都快第二版了，但国内第一版还没出来。那就不等啦。

### 1. Block Binding

使用 `var` 声明的变量，其声明部分会被提升至所在作用域的顶部。在没有块级作用域的情况下，变量提升（hoisting）会带来很多麻烦。因此 ES6 引入了块级作用域来改进。

在块内声明的变量无法在块外访问。块级作用域在两种情况下被创建：在一个函数内，或者在一个代码块内。使用 `let` 声明的变量只存在于当前代码块中，并且不存在变量提升。另外，也不能用 `let` 重复声明一个当前作用域已存在的标识符：

	var str = 'hello'

	let str = 'world'  // 语法错误

与 `let` 声明一样，`const` 声明的变量也是块级声明，因此同样不能再块外被访问，也不存在变量提升，标识符也不能被重复声明。唯一的区别是，使用 `const` 声明的变量被认为是一个常量，它不能被再次赋值，因此必须在声明时就完成初始化。但如果声明的是对象，则稍有不同：

	const js = { edition: 'es5' }

	js.edition = 'es6'       // 工作正常

	js = { edition: 'es6' }  // 抛出错误

可以看出，`const` 只是阻止对变量绑定的修改，并不阻止对成员值的修改。

由于不存在变量提升，使用 `let` 和 `const` 声明的变量会被放在暂时性死区（temporal dead zone）内，在声明位置之前无法被访问。但在块级作用域外是可以被访问的：

	console.log(typeof hi)    // 返回 'undefined'

	{
	  console.log(typeof hi)  // 引用错误
	  let hi = 'world'
	}

如果 `var` 声明在 `for` 循环内会很糟糕：计数器变量在循环外部仍能被访问。更糟糕的是，如果循环内创建了引用它的函数，所有的函数只能共享循环结束后的计数值：

	var funcs = []

	for (var i = 0; i < 10; i++) {
	  funcs.push(() => { console.log(i) })
	}

	funcs.forEach((func) => { func() })  // 输出 10 次数值 '10'

在没有块级绑定的时候，我们会使用 IFFE 来解决这个问题，即将变量的副本（而非引用）传入函数：

	for (var i = 0; i < 10; i++) {
	  funcs.push(((val) => {
	    return () => {
	      console.log(val)
	    }  
	  }(i)))
	}

有了块级绑定后，只需将 `var` 改为 `let` 即可实现预期。循环内的 `lei` 声明具有特殊的行为：每次迭代都会创建一个新的 `i` 变量并对它进行初始化，保证每次迭代的 `i` 都是一个独立的 `i` 副本。在 `for-in` 和 `for-of` 循环中，`let` 声明也具有相同的行为。

与循环内的 `let` 声明不同的是，`const` 声明只能工作在只能在 `for-in` 和 `for-of` 循环中，而不能工作在 `for` 循环中。原因很简单，前者在每次迭代中创建了新的变量绑定，而后者试图修改已绑定的变量的值：

	for (const key in obj) { /* ... */ }          // 工作正常

	for (const i = 0; i < 10; i++) { /* ... */ }  // 抛出错误

块级绑定在全局作用域也与 `var` 声明不同。在全局作用域使用 `var` 声明不仅会创建一个全局变量，也会成为全局对象的一个属性，这可能会无意间覆盖一个全局属性。而块级绑定只会创建全局变量，不会被添加到全局对象上。这意味着全局变量只会屏蔽全局属性，而不会覆盖它：

	let hello = 'hello'
	console.log('hello' in window)  // false

	var world = 'world'
	console.log('world' in window)  // true

块级绑定的最佳实践是不使用 `var`，默认情况下使用 `const`，只有在知道变量值需要被更改时才使用 `let`。

### 2. String & Regular Expression

JS 的字符串一直以来都是以 16 位字符编码方式为基础的：每 16 bit 作为一个码元（code unit），用以表示一个字符。Unicode 为世界上所有的字符提供全局的唯一标识符，即代码点（code point）。由于 Unicode 在多语言基本平面外增加了扩展平面，单个 16 位的码元已经无法表示所有的代码点。

解决方式是允许两个 16 位码元来表示单个代码点。比如 Unicode 字符 `𠮷` 实际上是两个 16 位的字符，所以原来针对 16 位的操作，将出现很多正确但不合理的情况：

	var text = '𠮷'
	console.log(text.length);         // 2
	console.log(/^.$/.test(text));    // false
	console.log(text.charAt(0));      // ''
	console.log(text.charAt(1));      // ''
	console.log(text.charCodeAt(0));  // 55362
	console.log(text.charCodeAt(1));  // 57271

ES6 针对这些问题，提供了很多新的针对 32 位的字符串方法来应对。`codePointAt()` 方法接收字符串中 16 位码元的位置并返回代码点。如果该位置本身就是 16 位字符，或者是 32 位字符的后半部分，则返回值与 `charCodeAt()` 一致；如果该位置是 32 位字符的前半部分，则返回完整的代码点：

	const is32Bit = (c) => c.codePointAt(0) > 0xffff

	console.log(is32Bit('𠮷'))        // true
	console.log('𠮷'.codePointAt(0))  // 134071
	console.log('𠮷'.codePointAt(1))  // 57271

另外，`String.fromCodePoint()` 可以视为 `String.fromCharCode()` 的完善版本，或者说是 `codePointAt()` 的相反操作。

内容相同的字符可以有不同的代码点，比如 `æ` 和 `ae`，但是直接比较并不会相等。必须先用`normalized()` 方法可以将字符标准化为相同的形式。

正则表达式也是基于 16 位码元来表示单个字符的。使用 `u` 标志可以将工作模式切换到针对代码点而非码元：

	const codePointLength = (text) => {
	  var result = text.match(/[\s\S]/gu)
	  return result ? result.length : 0
	}

	console.log(codePointLength('𠮷'))  // 1

使用 `u` 标志前提前检查可用性也很有必要（稍微修改后也可以用于检测新的 `y` 标志）：

	const hasRegExpU = () => {
	  try {
	    var pattern = new RegExp('.', 'u')
	    return true
	  } catch (ex) {
	    return false
	  }
	}

甚至还可以分别获取正则表达式的模式和标志：

	const reg = /abcde/imguy

	console.log(reg.source)  // 'abcde'
	console.log(reg.flags)   // 'gimuy'

使用 `str.indexOf(substr) !== -1` 来判断 `str` 包含 `substr` 很不语义。新的方式是使用 `includes()` 方法，`startsWith()` 方法和 `endsWith()` 方法。它们的第一个参数是子串，第二个参数是首次匹配的索引位置，返回布尔值。

只有需要确切位置，或者需要使用正则表达式时才使用 `indexOf()` 方法和 `lastIndexOf()` 方法。另外，新的 `repeat()` 方法可以将字符串重复固定的次数，返回一个新字符串。

ES6 在字符串方面引入的最大改进是模板字面量（template literal）。它提供了领域专用语言（domain-specific language）的语法，用于解决一些特殊的问题。模板字面量使用一对 ````` 来包裹内容，且无需对 `'` 或 `"` 转义（但 ````` 肯定是需要转义的）。

此前如果想要创建多行字符串，需要一些奇怪的方式。

	const msg = ['multiline', 'string'].join('\n')

	const msg = 'multiline\n' + 'string'

现在可以直接在模板字面量中换行了。不过要注意缩进的问题：

	const msg = `multiline
	string`

	const msg = `multiline\nstring`

	const html = `
	<div>
	  <p>paragragh</p>
	</div>`.trim()

模板字面量的特点在于可以在 `${` 和 `}` 之间放入 JS 表达式，这被称为替换位。无论是变量、计算、函数调用，都可以放进替换位中。甚至可以嵌入另一个模板字面量（因为模板字面量本身也是表达式）：

	const name = 'world'
	const msg = `hello, ${name}`

模板字面量最强大的功能是模板标签（template tag）。标签是在模板的第一个 ````` 之前被指定的函数。标签函数在调用时会接收模板字面量数据，并划分为独立片段，然后组合这些信息片段来输出最终的字符串。对于一个模板标签，其标签函数的形态如下：

	const msg = tag`hello, world!`

	function tag(literals, ...substitutions) { /* ... */ }

其中，`literals` 是个数组，包含了被替换位隔开的字面量字符串，`substitutions` 是剩余参数数组，包含了替换位的解释值。可以得知，`literals.length === substitutions.length + 1`。如果用标签函数来模拟模板字面量默认的转换操作，模拟实现将大致如下：

	function defaultTag(literals, ...substitutions) {
	  let result = ''

	  for (let i = 0; i < substitutions.length; i++) {
	    result += literals[i]
	    result += substitutions[i]
	  }

	  result += literals[literals.length - 1]
	  return result
	}

可以看出模板字面量的默认标签行为是：交替拼接两个数组中的元素，最终创建出完整的结果字符串。需要注意的是，如果替换位是位于模板字面量的开始或结束，那么 `literals[0]` 或 `literals[literals.length - 1]` 的值是空字符串。此外，`literals` 数组元素的值一定是字符串，但 `substitutions` 数组元素的值则由替换位的解释结果决定。

	const product = 'AirPods'
	let price = 159
	let taxRate = 0.17
	let exchangeRate = 6.925
	const rmb = price * (1 + taxRate) * exchangeRate

	let msg = `The ${product} costs ￥${parseInt(rmb)} in China.`
	console.log(msg)  // 'The AirPods costs ￥1288 in China'

针对上面的例子，`literals` 数组的元素值分别为：`'The '`、`' costs ￥'` 和 `' in China.'`，而 `substitutions` 数组的元素值分别为 `'AirPods'` 和 `1288`。

其实模板标签也可以访问字符串的原始信息，即转义之前的形式。具体而言，原始信息是存放在 `literal.raw` 属性里，即 `literal[i]` 的原始信息存放在 `literal.raw[i]` 中。内置的 `String.raw()` 标签默认具有获取原始字符串值的功能，模拟实现如下：

	function rawTag(literals, ...substitutins) {
	  let result = ''

	  for (let i = 0; i < substitutions.length; i++) {
	    result += literals.raw[i]
	    result += substitutions[i]
	  }

	  result += literal.raw[literal.length - 1]
	  return result
	}

具体的使用上，`rawTag()` 与 `String.raw()` 是等价的：

	const msgOne = rawTag`multiline\nstring`
	const msgTwo = String.raw`multiline\nstring`

	console.log(msgOne)  // 'multiline\\nstring'
	console.log(msgTwo)  // 'multiline\\nstring'

可以看出，转义字符是以原始形式返回的：作为换行字符的 `\n` 的代码形式是一个 `\` 字符和一个 `n` 字符，即 `\\n` 的原始形式。
