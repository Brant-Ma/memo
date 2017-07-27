## Mocha
> 原来 mocha 既可以是一种 flavor，也可以是一种 coffee...

选择 Mocha 而不是 Jasmine 可能更多的是因为抹茶比茉莉更有趣吧。

### 单元测试

单元测试（unit testing）是针对程序单元进行的正确性检验。单元（unit）指的是程序中的最小可测试部件，比如一个函数、一个类或者一个模块。

单元测试的目标是隔离程序部件并证明这些单个部件是正确的。因此使用单元测试的最大好处就是准确的反映代码变更后的程序表现，从而在开发的早期就发现问题所在。

### 使用 Chai

使用测试框架都需要一个断言库（assertion library）。Chai 是个能与任何 JS 测试框架配合的断言库，而且既可以用在 node.js 中，也可以用在 browser 中。当然通常是安装在 node.js 环境：

```javascript
$ npm install chai --save-dev
```

然后需要选择一种断言风格。TDD 风格使用传统的 `assert` 接口，BDD 风格使用更接近自然语言的 `expect` 或 `should` 接口。BDD 风格的两种 flavor 大同小异，个人更倾向于 `expect` 风格，因为 `expect` 关键字前置显得更整洁：

```javascript
import { expect } from 'chai'

expect(foo).to.be.a('string')
expect(foo).to.equal('bar')
expect(foo).to.have.lengthOf(3)
expect(foo).to.have.property('flavors').with.lengthOf(3)
```

Chai 提供了一些关键字来作为 chain，从而增强断言的可读性：to、be、been、is、that、which、and、has、have、with、at、of、same、but、does。

Chai 提供了完整的 [BDD Assertion API](http://chaijs.com/api/bdd/)，不过常用的并不多：

```javascript
// .equal() 会断言目标与给定参数 strict 相等
expect('foo').to.equal('foo')

// .deep 会让链中跟随的断言使用 deep 相等
expect({a: 1}).to.deep.equal({a: 1})

// .a() 或 .an() 会断言目标的类型等于给定的字符串类型
expect(new Error).to.be.an('error')

// .include() 或 .contain() 会断言目标包含给定的子集
expect([1, 2, 3]).to.include(3)

// .property() 会断言目标具有给定的参数的属性
expect({a: 1}).to.have.property('a')

// .lengthOf() 会断言目标的 length 属性等于给定值
expect('12345').to.have.lengthOf(5)
```

### 使用 Mocha

（终于回到了正题）Mocha 是一个运行在 node.js 和 browser 环境的 JS 测试框架，优点是支持异步并且特性丰富。

由于 Mocha 默认执行 `/test` 目录下的脚本，因此最好将测试脚本放在该路径下。通常测试脚本名与待测模块名相同，并以 `.test.js` 为后缀。如果只有一个测试脚本可以简单命名为 `test.js`：

```javascript
$ npm install mocha --save-dev
$ mkdir test
$ touch test/test.js
```

测试脚本中包含至少一个 `describe` 块，每个 `describe` 块包含至少一个 `it` 块。其中，`describe` 块表示 test suite，即一组相关的测试，`it` 块表示 test case，即一个最小的测试单元。二者的第一个参数是 suite 或 case 的名称，第二个参数是实际执行的函数。

假如要测试的模块是项目根目录的 `friendlyURL.js`：

```javascript
// friendlyURL.js
const friendlyURL = (url) => {
  const noHead = url.replace(/^https?:\/\//, '')
  return noHead.replace(/\/$/, '')
}

module.exports = friendlyURL
```

那么测试脚本的基本结构如下：

```javascript
// test.js
const { expect } = require('chai')
const getURL = require('../friendlyURL.js')

describe('friendlyURL', () => {
  it('should generate nice URL', () => {
    const url = getURL('https://apple.com/')
    expect(url).to.equal('apple.com')
  })
})
```

然后在 `package.json` 中加入测试脚本：

```javascript
"scripts": {
  "test": "mocha"
}
```

最后用 npm script 的方式来运行测试脚本：

```javascript
$ npm test
```

每个 `describe` 块都具有四个钩子，它们可以在指定时间执行回调函数：

```javascript
describe('test suite', () => {

  before('some description', () => {
    // runs before all tests in this block
  })

  after('some description', () => {
    // runs after all tests in this block
  })

  beforeEach('some description', () => {
    // runs before each test in this block
  })

  afterEach('some description', () => {
    // runs after each test in this block
  })

  // test cases
})
```

### 参考
1. [单元测试 - 维基百科](https://zh.wikipedia.org/wiki/单元测试)
2. [Chai Assertion Library](http://chaijs.com/)
3. [Mocha - the fun, simple, flexible JavaScript test framework](http://mochajs.org/)
4. [测试框架 Mocha 实例教程 - 阮一峰](http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html)
