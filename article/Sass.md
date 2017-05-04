## Sass
> Sass 是将编程特性引入到样式书写中，从而让 CSS 更加简洁的 CSS 预处理器。类似的预处理器有 LESS 和 Stylus，

### 1. 基本使用
使用命令 `gem install sass` 安装 Sass，然后通过 `sass -v` 检查是否安装成功
- 基本转换：`sass demo.scss`（将 demo.scss 转化为 demo.css）
- 完整转换：`sass --style [style] demo.scss demo.css`（编译风格有 4 种可选，见下文）
- 样式监听：`sass --watch [src]:[dist]`（监听对象可以是文件或目录）

### 2. 编译风格
- `nested`：（默认值）嵌套风格，能够呈现 selector 的嵌套结构
- `expanded`：扩展风格，标准的 CSS 书写方式
- `compact`：紧凑风格，能够根据 selector 来快速扫视
- `compressed`：压缩风格，能够最大限度的压缩规模，除了后代选择器和文件末尾，不存在任何的空白

### 3. 编程特性

#### 变量
Sass 使用 `$` 标识变量，变量名中可以用中划线或下划线。任意合法的属性值均可作为变量值，变量也可以是变量值的一部分。变量引用有块作用域的限制。

	$primary-color: #42a7e7;
	div {
	  $basic-border: 1px solid $primary-color;
	  border: $basic-border;
	}
	
	// 编译后
	div { border: 1px solid #42a7e7; }

#### 嵌套
Sass 的规则块（`{}`）既可以包含属性，也可以嵌套其他规则块，编译时父选择器和子选择器会以后代选择器的方式连接。使用伪类等需要指代容器本身的情况时，使用 `&` 标识符来指代父选择器。群组选择器（`,`）、直接后代选择器（`>`）、兄弟选择器（`~`）、相邻兄弟选择器（`+`）在嵌套块中同样可以被很好的编译。在根属性后加 `:` 后，子属性也可以嵌套。

	body {
	  background-color: #eee;
	  .wrapper { color: #333; }
	  &:hover { box-shadow: 0 2px 4px 0 #000; }
	  h1, h2 {
	    span { font-weight: bold; }
	  }
	  > div {
	    article + blockquote {
	      border: {
	        top: 5px;
	        bottom: 1px;
	      }
	      margin: 20px { right: 10px; }
	    }
	  }
	}
	
	// 编译后
	body { background-color: #eee; }
	body .wrappper { color: #333; }
	body:hover { box-shadow: 0 2px 4px 0 #000;}
	body h1 span, body h2 span { font-weight: bold; }
	body > div article + blockquote {
	  border-top: 5px;
	  border-bottom: 1px;
	  margin: 20px;
	  margin-right: 10px;
	}

#### 导入
不同于 CSS 的 `@import` 规则（只有执行到该行时才去下载要导入的样式），Sass 的 `@import` 规则在编译前就将相关的样式导入。Sass 中的 `@import` 规则默认用于导入 Sass 文件而非原生用法，除非导入文件是显式的 CSS 文件。

仅用于 `@import` 规则的样式导入而无需被编译为原生样式的 Sass 文件被称为局部文件，局部文件以下划线开头从而告知编译器，但书写时可以简写，如导入 `layout/_flex.scss` 时书写 `@import "layout/flex"`。此外，`@import` 规则支持嵌套导入。

	// 局部文件（theme.scss）
	article {
	  color: #333;
	  background-color: #eee;
	}
	
	// 当前文件
	.content { @import "theme"; }
	
	// 编译后
	.content {
	  article {
	    color: #333;
	    background-color: #eee;
	  }
	}

局部文件可以被多个文件引用，而不同的使用场景可能存在细微差异。可以通过 `!default` 在局部文件中定义默认变量值来实现这种区别，类似于 `!important` 的对立面。

	// 局部文件
	$primary-color: #42a7e7 !default;
	$basic-border: 1px solid #eee !default;
	
	// 当前文件
	$primary-color: #d75893;
	.nav {
	  color: $primary-color;
	  border: $basic-border;
	}
	
	// 编译后
	.nav {
	  color: #d75893;
	  border: 1px solid #eee;
	}

#### 混合
变量只能解决简单的值复用问题，而 `@mixin` 和 `@include` 可以解决代码段的复用问题。前者声明一段代码，后者调用一段代码。

	@mixin clearfix {
	  &::after {
	    display: block;
	    content: "";
	    clear: both;
	  }
	}
	.parent { @include clearfix; }
	
	// 编译后
	.parent::after {
	  display: block;
	  content: "";
	  clear: both; 
	}

通过传参可以让混合更加的定制化。更加灵活的是，`@mixin` 声明时可以为参数指定默认值，`@include` 调用时可以明确的指定参数值。

	@mixin link-colors($normal, $visited: $normal, $hover, $active: $normal) {
	  color: $normal;
	  &:visited { color: $visited; }
	  &:hover { color: $hover; }
	  &:active { color: $active; }
	}
	a {
	  @include link-colors(
		$normal: blue;
	    $hover: green;
	  );
	}

#### 继承
如果说混合是基于呈现的复用，继承则是基于语义的复用。`@extend` 可以使选择器 a 继承选择器 b 的所有样式，以及所有与 b 相关的样式。

原理上，混合是将声明好的代码块替换到调用的区域，即复制完整代码；而继承仅仅是将由继承双方构成的群组选择器替换被继承的选择器，即只复制选择器。

	.block {
	  display: block;
	}
	.shadow {
	  box-shadow: 0 2px 4px 0 #000;
	}
	.warn {
	  font-size: 2rem;
	  font-weight: bold;
	}
	// 继承
	.error {
	  @extend .warn;
	  color: #d75893;
	  background-color: #f9f2f4;
	}
	// 链式继承
	.serious {
	  @extend .error;
	  border: 1px solid #d75893;
	}
	// 多重继承
	.block-shadow {
	  @extend .block;
	  @extend .shadow;
	}
	
	// 编译后
	.block, .block-shadow {
	  display: block;
	}
	.shadow, .block-shadow {
	  box-shadow: 0 2px 4px 0 #000;
	}
	.warn, .error, .serious {
	  font-size: 2rem;
	  font-weight: bold;
	}
	.error, .serious {
	  color: #d75893;
	  background-color: #f9f2f4;
	}
	.serious {
	  border: 1px solid #d75893;
	}

### 参考
1. [Sass 基础教程](http://www.sasschina.com/guide/)
2. [Sass Documentation](http://sass-lang.com/documentation/file.SASS_REFERENCE.html)
