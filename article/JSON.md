## JSON
> 深入了解一下不是什么坏事

### 语法
JSON 和 XML 一样，属于一种数据交换格式，即用于两个实体（应用程序、服务器、语言）之间的通信。

JSON 的构建基于两种基本的数据结构：对象和数组。

object 是无序的键值对的集合：

![object](../image/json-object.gif)

array 是有序的值的集合：

![array](../image/json-array.gif)

object 和 array 中的值，可以是基本数据类型，也可以是 object 和 array，各个结构也支持嵌套：

![value](../image/json-value.gif)

string 必须使用双引号来表示：

![string](../image/json-string.gif)

number 不能是八进制或十六进制：

![number](../image/json-number.gif)

其他的隐式约束还有：对象的键也是字符串，因此也只能使用双引号；对象和数组的最后一个值后不允许使用逗号；值的类型并不支持 undefined 和 function；不支持注释语法。

另外，JSON 中的有效空白字符仅限于：空格符、制表符、换行符、回车符。

### JSON 与 JavaScript
道格拉斯创造的 JSON 几乎是 JavaScript 的子集，但存在例外：

行分隔符 `\u2028` 和段分隔符 `\u2029` 在 JSON 中是合法的，但在 JavaScript 中是非法的。这个历史遗留问题（而且已经无法再挽回）可以通过简单的字符替换来规避风险：

	function safeStringify(obj) {
	  return JSON.stringify(obj)
	  .replace(/\u2028/g, '\\u2028')
	  .replace(/\u2029/g, '\\u2029')
	}

尽管有一处疏漏，但整体上可以认为， JSON 是语法严格版的 JavaScript 对象。

### JSON 的使用
JSON 通常用于前后端的数据交互，也常常用于配置文件。由于 JSON 基于文本，因此符合 JSON 格式的字符串又叫做 JSON 字符串。

JavaScript 对象和 JSON 字符串之间的转换可以通过以下两个 API：

- `JSON.parse(text)`：解析 JSON 字符串参数，返回 JavaScript 对象的值

- `JSON.stringify(value)`：序列化 JavaScript 对象的值，返回 JSON 字符串

实际上，完整的方法签名还包括其他的参数，但很少用到。

### 参考
1. [JSON](http://json.org/)
2. [JSON - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON)
