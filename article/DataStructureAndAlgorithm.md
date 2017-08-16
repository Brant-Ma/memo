## Data Structure and Algorithm
> 它们应该是最基本的抽象了吧

笔记目录：

- [Data Structure](#data-structure)
  - [Array](#array)
  - [Enum](#enum)
- [Algorithm](#algorithm)
  - [Analysis](#analysis)
  - [Sort](#sort)
  - [Search](#search)

### Data Structure

常见的数据结构有：Array、Stack、Queue、LinkedList、Set、Map、Tree 和 Graph。

#### Array

判断数组的方式有很多，常用的有：

- `arr instanceof Array`（如果 arr 来自其他 iframe 则会失败）
- `Array.isArray(arr)`（可能说是很优雅了）
- `Object.prototype.toString.call(arr).slice(8, -1) === 'Array'`（万能的 toString）

数组有几个静态方法：

`Array.from(arrayLike, mapFn, thisArg)` 可以从类数组对象或者可迭代对象中创建一个数组：

```javascript
Array.from({length: 3}, (val, index) => index)  // [0, 1, 2, 3]
Array.from('abc')                               // ['a', 'b', 'c']
```

`Array.isArray(obj)` 可判断一个对象是否是一个数组：

```javascript
Array.isArray(Array)            // false
Array.isArray(Array.prototype)  // true
```

`Array.of(item1, ..., itemN)` 使用参数创建一个数组实例：

```javascript
Array.of(1)  // [1]
Array(1)     // []
```

数组还有许多的实例方法。有一些是会改变数组本身的 mutator 方法：

- `unshift(item1, ..., itemN)` 添加元素到头部，返回新长度
- `shift()`：移除头部的元素，返回该元素
- `push(item1, ..., itemN)` 添加元素到尾部，返回新长度
- `pop()` 删除尾部的元素，返回该元素
- `sort(compareFn)` 为数组排序，返回修改过的数组。默认顺序是按照字符串代码点，回调函数具有两个参数
- `reverse()` 反转数组顺序，返回新数组
- `splice(start, delCount, item1, ..., itemN)` 从指定位置开始删除并添加元素，返回被删除的元素组成的数组。默认 delCount 是从开始位置到最后，默认无 item
- `fill(value, start, end)` 用一个固定值对数组的某个范围进行填充，返回修改过的数组。默认 start 是 0，默认 end 是 this.length
- `copyWithin(target, start, end)` 浅拷贝数组某个范围的元素并从目标位置开始粘贴，返回修改过的数组。默认 start 是 0，默认 end 是 this.length

有一些是不改变数组本身，而是返回新数组的 accessor 方法：

- `indexOf(target, start)` 查找元素在数组中的索引，返回第一个匹配到的索引或者 -1。默认 start 是 0
- `lastIndexOf(target, start)` 同上，但从后向前查找
- `toString()` 返回数组元素的字符串形式，通常分隔符是逗号。该方法覆盖（override）了对象的实例方法
- `slice(start, end)` 返回数组某部分的浅拷贝。默认 start 是 0，默认 end 是 this.length
- `join(separator)` 返回数组元素的字符串形式。默认分隔符是逗号
- `concat(item1, ..., itemN)` 返回由该数组和其他数组或元素连接而成的新数组
- `includes(target, start)` 返回数组是否包含某个值。默认 start 是 0

有一些是通常需要 callback 作为参数的 iteration 方法，通常不建议在逻辑中改变数组本身。callback 的参数通常是（value, index, array）

- `forEach(callback, thisArg)` 为每个元素执行一次 callback，返回 undefined
- `every(callback, thisArg)` 测试是否每个元素的 callback 都返回 true，最终返回 true 或 false
- `some(callback, thisArg)` 测试是否存在元素的 callback 能返回 true，最终返回 true 或 false
- `map(callback, thisArg)` 返回一个由每个元素的 callback 返回值构成的新数组
- `filter(callback, thisArg)` 返回一个由通过 callback 测试的元素所构成的新数组
- `find(callback, thisArg)` 查询满足 callback 测试条件的元素，返回匹配到的第一个元素或 undefined
- `findIndex(callback, thisArg)` 查询满足 callback 测试条件的元素，返回匹配到的第一个元素的索引或 -1
- `reduce(callback, initial)` 对累加器和当前元素调用 callback 并返回值作为下个元素的累加器，最终使数组缩减为一个值并返回。initial 默认是第一个元素，callback 的参数与其他 iteration 方法不同：（sum, value, index, array）
- `reduceRight(callback, initial)` 同上，但从后向前进行 reduce
- `keys()` 返回一个包含数组元素 index 的可迭代对象
- `values()` 返回一个包含数组元素 value 的可迭代对象
- `entries()` 返回一个包含数组元素键值对的可迭代对象
- `[Symbol.iterator]()` 返回一个可迭代对象，默认采用 `values()` 方法的结果

#### Enum

ES6 引入了 const 来声明常量。平时开发也确实会遇到常量，比如项目的配置参数和应用状态。极端的开发者甚至会选择这样一种变量的声明方案：首选 const，少用 let，不用 var。（Airbnb 就采用了这样的代码风格）

但如果存在大量的常量，其中一些常量之间有关联呢？可以选择进行分组：

```javascript
const Size = {
  SMALL: 0,
  MEDIUM: 1,
  LARGE: 2
}

let mySize = Size.SMALL
```

这样似乎是够用了。但如果每个常量还有对应的一些属性呢？可以再加一个属性来统一存储：

```javascript
const Size = {
  SMALL: 0,
  MEDIUM: 1,
  LARGE: 2,
  props: {
    0: { name: 'small', value: 0, label: 'S' },
    1: { name: 'medium', value: 1, label: 'M' },
    2: { name: 'large', value: 2, label: 'L' }
  }
}

let mySize = Size.SMALL
let myLabel = Size.props[mySize].label
```

目前已经可以将常量的值和相关属性完美的关联在一起了。但现在的常量和属性需要自己去手动关联，如果能用对象的方法去构造和访问就更好了：

```javascript
class Size {
  constructor(name, label) {
    this.name = name
    this.label = label
  }

  getName() { return this.name }

  getLabel() { return this.label }
}

Size.SMALL = new Color('small', 'S')
Size.MEDIUM = new Color('medium', 'S')
Size.LARGE = new Color('large', 'S')

Object.freeze(Size)

let mySize = Size.SMALL
let myLabel = Size.SMALL.getLabel()
```

上述过程实际上是通过类模拟了枚举类型。

每个枚举值都是类的一个实例，并且分配给了类的一个静态属性。每个枚举值都具有实例方法，可以访问相关的属性。枚举类在实例创建和分配完毕后被冻结了，意味着不能再被修改。

而这也符合了枚举类型的特点：枚举值是常量（冻结后不能修改），所有枚举值具有相同的枚举类型（都是同一个类的实例），每个枚举值具有自己的属性和方法（实例具有属性和方法）。

### Algorithm

最基本的算法是排序算法和搜索算法。

#### Analysis

对于一个具体的算法，最基本的指标是健壮性和可读性。但算法作为完整程序的一部分，其性能或者效率会显得更更加重要，因此需要分析算法的运行成本。算法的运行成本是指对系统资源的占用，主要包括对内存的占用和所需的运行时间。

简单的讲，内存占用方面需要关注的是引用类型的数据的规模，尤其是在递归调用的情况下。空间复杂度通常并不需要过多的关注，有些情况下甚至会考虑牺牲空间来换取时间。

运行时间方面需要关注的是内循环代码的组成。时间复杂度通常使用大 O 表示法来描述，它表述了运行时间对于问题规模 n 的增长数量级。常见的几种增长数量级函数如下：

- O(1)：常数级别，比如一条仅有基本操作的普通语句
- O(logN)：对数级别，比如使用二分策略的 binary 搜索
- O(N)：线性级别，比如使用单层循环的 linear 搜索
- O(NlogN)：线性对数级别，比如使用分治方法的 merge 排序和 quick 排序
- O(N^2)：平方级别，比如使用双层循环的 bubble 排序、selection 排序和 insertion 排序
- O(N^3)：立方级别，比如使用三层循环的三元组求和
- O(2^N)：指数级别，比如使用穷举方法检查所有的子集

优化一个算法其实就是去除循环和优化逻辑，让增长数量级尽量是对数级别、线性级别和线性对数级别。

毫无疑问，关注算法的性能是必要的，因为很多暴力算法在解决大规模问题时运行的很慢。但过于关注性能也未必是好事，很多时候写出清晰正确的代码才是首要目的。优化代码所占用的时间成本有可能会降低整个生产效率，甚至是毫无意义的：如果问题规模并不大，那么将运行时间为 100 毫秒的算法优化至 10 毫秒，可能也无法被直接感知到。

#### Sort

最常用的排序算法有：冒泡排序、选择排序、插入排序、归并排序和快速排序。

冒泡排序和选择排序需要一个共同的 swap 方法，用于交换两个数组元素：

```javascript
const swap = (former, latter) => {
  const temp = former
  former = latter
  latter = temp
}
```

冒泡（bubble）排序的原理是每轮比对相邻元素，将较大者冒泡至尾端，每轮冒泡得出一个当前的最大值。实现如下：

```javascript
const bubbleSort = (arr) => {
  const len = arr.length
  // 共 len - 1 轮
  for (let i = 0; i < len - 1; i += 1) {
    // 已冒泡出 i 个最大值，本轮从第 1 个开始，到第 len - i 个
    for (let j = 0; j < len - i - 1; j += 1) {
      if (arr[j] > arr[j + 1]) {
        swap(arr[j], arr[j + 1])
      }
    }
  }
  return arr
}
```

选择（selection）排序的原理是每轮选定一个最小值，把实际更小的与之交换，每轮选择得出一个当前的最小值。实现如下：

```javascript
const selectionSort = (arr) => {
  const len = arr.length
  // 共 len - 1 轮
  for (let i = 0; i < len - 1; i += 1) {
    // 假设第 1 + i 个是最小值
    let min = i
    // 已选择出 i 个最小值，本轮从第 2 + i 个开始，到最后一个
    for (let j = i + 1; j < len; j += 1) {
      if (arr[j] < arr[min]) {
        min = j
      }
      swap(arr[j], arr[min])
    }
  }
  return arr
}
```

插入（insertion）排序的原理是构建一个有序子数组，每轮插入一个元素，最终形成一个完整的有序数组。实现如下：

```javascript
const insertionSort = (arr) => {
  const len = arr.length
  // 共 len - 1 轮
  for (let i = 1; i < len; i += 1) {
    // 对于第 i + 1 个带插入的元素，遍历已排好序的前 i 个
    for (let j = 0; j < i; j += 1) {
      if (arr[i] < arr[j]) {
        arr.splice(j, 0, arr[i])
        arr.splice(i + 1, 1)
      }
    }
  }
  return arr
}
```

归并（merge）排序的原理是将一个大数组拆分为无数的小数组，然后再合并为完整的有序数组。递归实现如下：

```javascript
const mergeSort = (arr) => {
  // 拆分
  if (arr.length < 2) return arr.slice()
  const mid = Math.floor(arr.length / 2)
  const left = mergeSort(arr.slice(0, mid))
  const right = mergeSort(arr.slice(mid))
  // 合并
  const merge = (a, b) => {
    const ret = []
    while (a.length && b.length) {
      ret.push(a[0] < b[0] ? a.shift() : b.shift())
    }
    return ret.concat(a.length ? a : b)
  }
  return merge(left, right)
}
```

快速（quick）排序的原理是选定一个元素作为中心基点，然后将小值放其左侧，大值放其右侧，最后形成完整的有序数组。递归实现如下：

```javascript
const quickSort = (arr) => {
  if (arr.length < 2) return arr.slice()
  const one = arr[0]
  const left = arr.slice(0).filter(item => item < one)
  const right = arr.slice(0).filter(item => item >= one)
  return quickSort(left).concat(one, quickSort(right))
}
```

#### Search

最常用的搜索算法是线性搜索和二分搜索。

线性（linear）搜索是指从头到尾的遍历数组。实现如下：

```javascript
const linearSearch = (arr, item) => {
  const len = arr.length
  for (let i = 0; i < len; i += 1) {
    if (arr[i] === item) return i
  }
  return -1
}
```

二分（binary）搜索是指先将数组排序，然后不断的折半搜索。实现如下：

```javascript
const binarySearch = (arr, item) => {
  const sort = quickSort(arr)
  let low = 0
  let high = sort.length - 1
  while (low <= high) {
    const mid = Math.floor((low + high) / 2)
    if (sort[mid] < item) {
      low = mid + 1
    } else if (sort[mid] > item) {
      high = mid - 1
    } else {
      return mid
    }
  }
  return -1
}
```

### 参考

1. [学习 JavaScript 数据结构与算法 - Loiane Groner](https://book.douban.com/subject/26639401/)
2. [Array - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)
3. [JavaScript 中的枚举](http://kingzez.com/2016/05/23/Enums-in-javascript-%E8%AF%91/)
4. [谈谈 JavaScript 枚举类型](https://zhuanlan.zhihu.com/p/24880652)
