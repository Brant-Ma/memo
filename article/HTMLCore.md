## HTML Core
> 留个备份，防止遗忘

本文档用于快速复习 HTML 的基础与核心。目录结构如下：

1. [Concept](#concept)
2. [Summary](#summary)
3. [Document Level Element](#document-level-element)
4. [Text Level Element](#text-level-element)
5. [Group Level Element](#group-level-element)
6. [Section Level Element](#section-level-element)
7. [Table Level Element](#table-level-element)
8. [Form Level Element](#form-level-element)
9. [Embed Level Element](#embed-level-element)

### Concept

元素（element）包含标签和内容。元素分为元数据元素、短语元素和流元素。空元素没有内容，如 `<code></code>`，或写作 `<code/>`；虚元素不能有内容，如 `<hr>`，或写作 `<hr />`。

属性（attribute） 包含属性名和属性值。属性分为全局属性和局部属性。布尔属性只需要写属性名，如 `disabled`；自定义属性以 `data-` 开头。全局属性如下：

- `accesskey`：定义元素的快捷键
- `class`：对元素进行分类
- `contenteditable`：使元素内容可编辑
- `contextmenu`：为元素添加快捷菜单
- `dir`：指定元素内容的布局方向
- `draggable`：声明元素可拖动
- `dropzone`：声明其他元素可拖放至该元素上
- `hidden`：表示无需关注该元素及其内容
- `id`：为元素分配独有标识符
- `lang`：声明元素内容所用语言
- `spellcheck`：检查元素内容的拼写错误
- `style`：直接定义元素样式
- `tabindex`：指定元素的tab键次序
- `title`：为元素提供额外信息

文档（document）是通过本地磁盘或 web 服务器载入到用户代理的文本文件。文档外层结构由 `<!DOCTYPE>` 和 `<html>` 决定。元数据位于 `<head>` 中，文档内容位于 `<body>` 中，注释位于 `<!--` 和 `-->` 之间。

实体（entity）是在文档中替代特殊字符的代码。常用的字符-实体对有：

- `<`：`&lt;`
- `>`：`&gt;`
- `&`：`&amp;`

### Summary

HTML5 元素的基本理念是将语义与呈现分离。元素的选用原则大致有以下几条：

- 别多用：用少量的元素解决问题
- 别错用：按照语义来选择合适的元素
- 别泛用：尽量选择具体的而非通用的元素
- 别乱用：相同元素的使用上要前后保持一致

所有元素的快速分类预览：

- 文档类：`base`, `body`, `DOCTYPE`, `head`, `html`, `link`, `meta`, `noscript`, `script`, `style`, `title`.
- 文本类：`a`, `abbr`, `b`, `bdi`, `bdo`, `br`, `cite`, `code`, `del`, `dfn`, `em`, `i`, `ins`, `kbd`, `mark`, `q`, `rp`, `rb`, `ruby`, `s`, `samp`, `small`, `span`, `strong`, `sub`, `sup`, `time`, `u`, `var`, `wbr`.
- 分组类：`blockquote`, `dd`, `div`, `dl`, `dt`, `figcaption`, `figure`, `hr`, `li`, `ol`, `p`, `pre`, `ul`.
- 分节类：`address`, `article`, `aside`, `details`, `footer`, `h1~h6`, `header`, `hgroup`, `nav`, `section`, `summary`.
- 表格类：`caption`, `col`, `colgroup`, `table`, `tbody`, `td`, `tfoot`, `th`, `thead`, `tr`.
- 表单类：`button`, `datalist`, `fieldset`, `form`, `input`, `keygen`, `label`, `legend`, `optgroup`, `option`, `output`, `select`, `textarea`.
- 嵌入类：`area`, `audio`, `canvas`, `embed`, `iframe`, `img`, `map`, `meter`, `object`, `param`, `progress`, `source`, `svg`, `track`, `video`.
- 未实现的元素：`command`, `menu`.

### Document Level Element

与文档结构相关的元素有：

- `DOCTYPE`：作为文档的开头，用于说明文档的类型和版本，HTML5 中写法固定为 `<!DOCTYPE HTML>`
- `html`：作为文档的根元素，包含 `head` 和 `body`，可使用 `manifest` 属性定义离线应用缓存
- `head`：包含文档的元数据，必须包含 `title`
- `body`：包含文档的内容

与元数据相关的元素有：

- `title`：设置文档的标题。能够显示在标签页上
- `base`：设置文档的解析基准
  - 使用 `href` 属性为文档中的相对 URL 设置解析基准（默认基准为当前文档的 URL）
  - 使用 `target` 属性为文档的链接设置打开方式
- `meta`：定义文档的元数据。每个 `meta` 元素只能用于一种用途
  - 使用 `name` 属性和 `content` 属性声明元数据键值对
  - 使用 `charset` 属性声明字符编码（通常是 `utf-8`）
  - 使用 `http-equiv` 属性和 `content` 属性改写 HTTP 头部字段
- `style`：定义文档的内嵌样式
  - 使用 `type` 属性指定样式类型（只能是 `text/css`）
  - 使用 `scoped` 属性指定样式作用范围（默认是整个文档）
  - 使用 `media` 属性指定样式适用的媒体
    - 可以指定设备，如 `screen` 和 `print`（默认是 `all`）
    - 可以指定特性，如 `width`、`max-width`、`min-width`
    - 设备和特性之间可以使用 `AND`、`NOT`、`,` 来组合条件
- `link`：指定文档的外部资源。通常是样式表
  - 使用 `rel` 属性说明文档与资源的关联关系，如 `stylesheet` 和 `icon`
  - 使用 `type` 属性指定资源的 MIME 类型，如 `text/css` 和 `image/x-icon`
  - 使用 `sizes` 属性指定资源大小（通常用于 `icon`）
  - 使用 `href` 属性指定资源的 URL（通常用于 `stylesheet`）
  - 使用 `hreflang` 属性指定资源使用的语言
  - 使用 `media` 属性指定资源适用的媒体

与脚本相关的元素有：

- `script`：指定需要加载的脚本并控制其执行。每个 `script` 只能用于一种用途（文档内嵌脚本或外部引用脚本）
  - 使用 `type` 属性指定脚本类型（可以省略）
  - 使用 `src` 属性指定外部引用脚本的 URL
  - 使用 `charset` 属性指定外部引用脚本的字符编码
  - 使用 `defer` 属性和 `async` 属性设定脚本的执行方式
    - 有 `async`：异步加载完成后立即执行，执行顺序将不同于脚本定义的顺序
    - 无 `async` 有 `defer`：异步加载后等待页面解析结束，再按定义的顺序执行
    - 无 `async` 无 `defer`：暂停文档解析，按定义的顺序同步的加载和执行脚本
- `noscript`：规定不支持或禁用脚本时的处理方法
  - 可以提供额外的内容（比如提示启用脚本）
  - 可以使用诸如 `<meta http-equiv="refresh" content="0; http://github.com">` 重定向页面

### Text Level Element

`a` 是最重要的文本元素。它用于内容导航（不同页面之间，或者同一文档中），可以使用的局部属性有 `href`、`hreflang`、`media`、`rel`、`target`、`type`，但常用的只有 `href` 和 `target`：

- 外部超链接：将 `href` 属性设置为以已知协议（通常是 `http://`）的 URL 来链接到外部资源，或者将其视为相对 URL 并结合文档的解析基准来链接到外部资源
- 内部超链接：将 `href` 属性设置为 `#<id>` 来查找并滚动到对应 `id` 属性值的页面元素处，查询失败则尝试元素的 `name` 属性值
- 使用 `target` 来指定浏览环境，即如何显示链接资源，默认为 `_self`，其他值有`_blank`、`_parent`、`_top`、`<frame>`

基本的文本元素有：

- `b`：表示关键词和产品名称，通常将内容加粗
- `i`：表示外文词语和科技术语，或人物的心理活动，通常是斜体样式
- `em`：表示强调的内容，读到此处时才注意到其提供的语境，因而通常是斜体样式
- `strong`：表示重要的内容，看到页面时就立刻凸现此处的内容，因而通常将内容加粗
- `s`：表示内容不再准确或校正，通常在内容上加删除线
- `u`：表示读者难以理解的中文姓名或错误拼写的内容，通常在内容下加下划线，但易与超链接混淆
- `small`：表示小号字体内容，如免责声明或澄清声明
- `sub`：表示下标内容，如语言或数学中
- `sup`：表示上标内容，如语言或数学中

用于换行或输入输出的文本元素有：

- `br`：强制换行，仅用于换行也是内容的一部分的情况，如诗歌和代码
- `wbr`：建议换行，为浏览器指出内容中适合换行的位置，如很长的单词
- `code`：代码，通常为等宽字体
- `var`：变量，通常为斜体样式
- `samp`：程序输出，通常为等宽字体
- `kbd`：用户输入，通常为等宽字体

用于文献或语言的文本元素有：

- `abbr`：表示缩写，其 `title` 属性对应缩写的全称
- `dfn`：定义术语，详细解释跟在 `dfn` 后面，简略解释在其 `title` 属性中，如果 `dfn` 嵌套有 `abbr`，则 `abbr` 的 `title` 属性可充当其简略解释
- `q`：引用具体的内容，其 `cite` 属性用来说明内容出处的URL，通常引文前后会生成引号
- `cite`：引用作品的标题，通常是斜体样式
- `ruby`：用于为汉语和日语等表意语言注音，内含文字、`rp` 元素、`rt` 元素
- `rp`：为不支持注音的浏览器显示括号，内含 `(` 或 `)`
- `rt`：表示注音，内含文字的注音
- `bdo`：明确指定元素内的字符层面的文字方向，全局属性 `dir` 的值可以是 `ltr` 或 `rtl`
- `bdi`：当浏览器自动处理文字方向时，将方向未知的文本包含其中，防止这段文本扰乱其后内容的处理

剩余的文本元素有：

- `span`：一般性的内容，含义通常取决于 `class` 或 `id`
- `mark`：突出显示的内容，通常采用黄色背景和黑色前景
- `ins`：添加的内容，可用属性有 `cite` 和 `datetime`，前者表示修改原因的页面 URL，后者表示修改时间
- `del`：删除的内容，可用属性有 `cite` 和 `datetime`，前者表示修改原因的页面 URL，后者表示修改时间
- `time`：表示时间或日期，可用属性有 `pubdate` 和 `datetime`，前者可声明在文档中查找发布时间（是一个布尔属性），后者用标准格式来指定日期或时间

### Group Level Element

分组相关的元素用于组织相关的内容：

- `p`：表示段落
- `div`：一般性的结构，含义通常取决于 `class` 或 `id`
- `pre`：保留内容预先编排好的格式，阻止浏览器合并空白字符，通常会嵌套 `code`
- `blockquote`：引用较多的内容，局部属性 `cite` 用来说明内容出处的 URL
- `hr`：表示分隔，用于向另一个相关主题的转换
- `ol`：内含 `li` 的有序列表，可用属性有 `start`、`type` 和 `reversed`，分别用于首项编号，编号类型，降序编号
- `ul`：内含 `li` 的无序列表（可使用 CSS 属性 `list-style-type`）
- `li`：列表项，父元素可以是 `ol`、`ul` 和 `menu`，当父元素是 `ol` 时，`value` 属性可更改列表项的计数器
- `dl`：内含 `dt` 和 `dd` 的定义列表，其中 `dt` 与 `dd` 是并列关系，每个 `dt` 可搭配多个 `dd`
- `dt`：定义的标题
- `dd`：定义的描述
- `figure`：插图，可包含 `figcaption`
- `figcaption`：插图的标题，必须是 `fugure` 的第一个或最后一个元素

### Section Level Element

分节相关的元素用于分隔无关的内容：

- `h1 ~ h6`：一级至六级标题，构成文档的大纲
- `hgroup`：内含`h1 ~ h6`，通过组织标题来区分子标题和节标题，`hgroup` 中只有第一个标题子元素会被纳入大纲
- `section`：明确定义的小节，内含应列入大纲或目录的内容，通常可以有一个标题和若干个段落，在多层嵌套的小节中，各个标题元素的 `font-size` 会根据它们之间的相对层级来确定，即使它们使用同级别的标签，如都使用 `h1`
- `header`：文档或某节的首部，通常包含 `h1 ~ h6`、`hgroup` 和 `nav`
- `footer`：文档或某节的尾部，通常包含作者介绍，版权信息，相关链接
- `nav`：规划文档的主要导航区域，不限于 `header` 内，也可以置于文档末尾
- `article`：表示独立成篇的内容，一个文档可以有多个 `article`，且可以进行嵌套
- `aside`：表示与页面主体相关的内容
- `address`：表示文档或文章的联系信息，具体根据它是 `body` 还是 `article` 的子元素
- `details`：可以折叠和展开内容，内含 `summary` 和其他元素，折叠时只显示 `summary`，展开时显示所有内容。可用属性有`open`，使`details`默认呈展开状态
- `summary`：作为详情区域的简略说明

### Table Level Element

表格相关的元素用于显示二维数据。基本型的表格元素有：

- `table`：表格
  - 使用 `border` 属性来设置表格和单元格的边框，同时明确目的：呈现表格数据而非页面布局
  - 浏览器会根据最宽最高的单元格，自动调整行与列的尺寸并维持表格的形式
- `tr`：行，HTML的表格是基于行的
- `td`：单元格，局部属性有 `colspan`、`rowspan`、`headers`
  - 使用 `colspan` 和 `rowspan` 使单元格占据更多的网格，最终的大网格会覆盖其他元素，只保留左上角元素
  - 使用 `headers` 与相关的 `th` 元素相关联
- `th`：表头单元格，用于区分数据和对数据的说明，局部属性有与 `td` 相同
  - `colspan` 和 `rowspan` 属性与 `td` 用法相同
  - 使用 `headers` 与相关的 `th` 元素相关联（所有 `th` 与表格左上角的 `th` 关联）
  - 使用 `id` 属性来被相关的 `td` 或 `th` 关联

结构化的表格元素有：

- `tbody`: 表格主体，包含组成主体的行，即使 `table` 中只有 `tr`，浏览器也会自动包裹一层 `tbody`
- `thead`: 表头行，包含组成表头的行
- `tfoot`: 表脚行，包含组成表脚的行，`tfoot` 出现在 `tbody` 或 `tr` 的前后都可以
- `caption`: 表格标题，无论定义在什么位置，都将在表格的上方显示
- `colgroup`: 创建一组列，用于实现基于列的样式，局部属性有 `span`
- `col`: 在一组列中创建一组列，在 `colgroup` 内使用，可以更加精细的实现基于列的样式，局部属性有 `span`

### Form Level Element

有点复杂，稍后总结。

### Embed Level Element

与嵌入图像相关的元素有：

- `img`：在文档中嵌入图像
  - 使用 `src` 属性指定图像的 URL
  - 使用 `alt` 属性指定图像的备用内容，该内容将在图片无法显示时呈现（如地址或格式错误，浏览器或设备问题）
  - 使用 `width` 属性和 `height` 属性声明图像的尺寸（提供给浏览器来预留空间，而非用来缩放图像）
  - 使用 `ismap` 属性创建 server 端的分区响应图
    - 需要在 `a` 元素中嵌套 `img`，点击图片将跳转至 `a` 元素指定的地址，URL 将携带点击时的地址信息
  - 使用 `usemap` 属性创建 client 端的分区响应图
    - 需要使用 `usemap` 属性指定 hash 引用，指向 `map` 元素的 `name` 属性值
- `map`：接受 `img` 的 `usemap` 指向，将请求导向 `area` 子元素
  - 使用 `name` 属性指定 client 端分区标识符
- `area`：代理 `img` 的点击分发，响应并导向新的 URL
  - 使用 `src` 和 `alt` 代理 `img` 的功能
  - 使用 `target`、`rel`、`media`、`hreflang`、`type` 描述相关说明
  - 使用 `shape` 指定分区形状（`rect`、`circle`、`poly` 或 `default`，默认是整张图像）
  - 使用 `coords` 指定分区坐标（默认形状无需坐标）

与嵌入插件内容相关的元素有：

- `embed`：一般很少用了
- `object`：可以用于插件内容，也可以用于浏览器能直接处理的内容
  - 使用 `width` 和 `height` 指定嵌入内容的尺寸
  - 使用 `src` 和 `type` 指定嵌入内容的地址和 MIME 类型
  - 使用 `usemap`、`name`、`form` 完成其他内容的处理
  - 内部可嵌套 `param` 属性指定传入插件的参数，还可以传入各种标签形式的备用内容
- `param`：通过 `name` 和 `value` 指定参数

与嵌入数值表现形式相关的元素有：

- `progress`：显示进度
  - 使用 `value` 指定当前进度
  - 使用 `max` 指定最大值，默认为 1（此时范围是 0 到 1）
- `meter`：显示范围内的可能值（很少用）

`iframe` 元素可以嵌入新的文档：

- 使用 `name` 属性创建浏览上下文，`a` 等元素可以将 `target` 指向该上下文的值（无需 hash），从而将自身的 `href` 地址导入 `ifame` 并显示内容
- 使用 `width` 和 `height` 指定嵌入文档的尺寸
- 使用 `src` 指定要载入的初始 URL，使用 `srcdoc` 指定要显示的 HTML 文档内容
- 使用 `seamless` 取消嵌入文档的边框，使其融入文档上下文的整体部分
- 使用 `sandbox` 禁用脚本、表单、插件和指向其他浏览上下文的链接，用作布尔属性则全部禁用，也可以单项禁用

另外，`audio`、`video` 可以嵌入音频和视频。`source` 可以指定视频源，`track` 可以指定视频字幕。`canvas` 可以嵌入动态图形。

### 参考
1. [HTML5 权威指南 - Adam Freeman](https://book.douban.com/subject/25786074/)
