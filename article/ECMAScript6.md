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
