## Editor
> 摆脱重量级的 IDE，使用轻量级的 editor

现在看来， Sublime Text 的优势在于效率（速度极快），而 Atom 的优势在于扩展（插件生态）。

### 1. Atom

Atom 是通过 web 技术搭建起来的桌面应用。基于 Chromium 的 web 结合 Node.js 的 API 让 Atom 既强大又灵活。

#### 1.1 Getting Started

可以通过 `⌘ + Shift + P` 调出 Command Palette，这是 Atom 中最重要的命令。它是一个搜索驱动的菜单，可以查找到各种命令及其快捷键。比如要在 Command Palette 中打开设置，可以模糊输入 Settings View（当然也可以使用快捷键 `⌘ + ,`）

基本的文件和目录操作：打开文件（`⌘ + O`），关闭文件（`⌘ + W`），新建文件（`⌘ + N`），保存文件（`⌘ + S`），另存文件（`⌘ + Shift + S`）。在当前窗口打开一个目录（`⌘ + O`），在当前窗口叠加一个目录（`⌘ + Shift +O`）。打开的目录在窗口侧边会有一个 Tree View，此时可以通过右键进行各种目录操作。

Fuzzy Finder 用来搜索树或项目中文件，它通过 `⌘ + T`（Tree）或者 `⌘ + P`（Project）打开。如果将搜索限定在已打开文件，可以使用 `⌘ + B`，这里的 B 是指 buffer，即已打开文件的文本内容，它位于内存而非磁盘。如果将搜索限定在上次 commit 后的变化文件（新增的文件或者被修改的文件），可以使用 `⌘ + Shift + B`。

#### 1.2 Using Atom

Atom 本身是一个非常基本的功能核心。它的强大来自于其承载了大量的包（package）。Atom 默认就具有许多的包，比如 Settings View、Tree View 和 Fuzzy Finder。除了这些 Core 包，还可以安装更多的 Community 包、Development 包和 Git 包。每个包都是独立的，而且提供详细的包资料，支持配置和更新。

常用的移动（Move）操作：

- 跨行级别的上下移动分别是 `Ctrl + P` 和 `Ctrl + N`
- 行内级别的首尾移动分别是 `Ctrl + A` 和 `Ctrl + E`
- 字符级别的左右移动分别是 `Ctrl + B` 和 `Ctrl + F`
- 单词级别的左右移动分别是 `Alt + B` 和 `Alt + F`
- 文件级别的首尾移动分别是 `⌘ + Up` 和 `⌘ + Down`
- 指定行列的精确移动使用 `Ctrl + G`（输入 `row:column` 来跳转）
- 具体符号的查询移动使用 `⌘ + R`
- 单行书签的设置和获取使用 `⌘ + F2` 和 `Ctrl + F2`

常用的选择（Select）操作：

- 将光标移动操作的前 10 种加上 `Shift` 即可实现对应的文本选择
- 当前文件的文本选择使用 `⌘ + A`
- 当前行的文本选择使用 `⌘ + L`
- 当前单词的文本选择使用 `Ctrl + Shift + W`

常用的编辑和删除（Edit & Delete）操作：

- 改变当前词的大小写使用组合键 `⌘ + K` `⌘ + U` 和 `⌘ + K` `⌘ + L`
- 连接下一行 `⌘ + J`
- 复制当前行 `⌘ + Shift + D`
- 剪切至行尾 `Ctrl + K`
- 删除当前行 `Ctrl + Shift + K`
- 选中下一个与当前词相同的词 `⌘ + D`
- 选中所有的与当前词相同的词 `⌘ + Ctrl + G`
- space 与 tab 的互相转换需要在命令面板搜索 `whitespace` 再选中对应命令
- 为 tag 添加闭合标记 `⌘ + Alt + .`

常用的查找和替换（Find & Replace）操作：

- 当前文件中查找和替换：`⌘ + F`
- 当前项目中查找和替换：`⌘ + Shift + F`

其他一些有用的技巧：
- 输入部分字符后按 `tab` 键可扩展片段或者自动补全
- 在命令面板可搜索 `Snippets Available` 查看可用的 snippet
- 代码块的折叠和展开：`⌘ + Alt + [` 和 `⌘ + Alt + ]`
- 标签页的切分和回收：比起快捷键可能右键操作更加方便
- Markdown 预览的触发：`Ctrl + Shift + M`

另外，单击打开的标签页是预览状态，或者说它是个 pending 页面，它的特性是可以被下一个 pending 页面替换。通过双击 tab 或 tree，编辑或保存内容，都可以将其转化为打开状态。

最后，Atom 整合了 Git 的各种功能。在 Git 面板（`Ctrl + 9`）和 GitHub 面板（`Ctrl + 8`）面板均有相关操作，在命令面板输入 Github 也可以进行一些操作。


### 2. Sublime

目前已逐步转向 Atom，但是 Sublime 的清爽让人没法彻底抛弃。不如记录一些常用操作，以后可以随时切换回来。

#### 2.1 Install

- 安装 package control: 打开 console, 复制 [代码](https://packagecontrol.io/installation) 完成安装
- 安装包: 在命令面板，输入 `install` 匹配 `Package control: Install Package` 进行自动安装；输入 `add` 匹配 `Package Control: Add repository` 进行手动安装

#### 2.2 Setting & Customization

- 命令面板：Command Palette (`⌘ + shift + p`)
- 定位面板：Goto Anything Palette (`⌘ + p`)
- 各种定位：在定位面板可以直接定位文件；输入 `:` 可以行定位；输入 `@` 可以块定位；输入 `#` 可以模糊搜索
- 改变语法：在命令面板输入 `ss` 匹配 `Set Syntax` 列表
- 个性化设置：直接键入 `⌘ + ,` 并参考左侧的 Default 设置右侧的 User
- 设置定位：`binary_file_patterns` 属性
- 小地图的边框: `draw_minimap_border` 属性
- 侧边栏的开关: 键入 `⌘ + k` && `⌘ + b`

#### 2.3 Multiple Carets & Line Tricks

- 创造光标：按住 `⌘` 并在需要的位置单击鼠标
- 内容选中：先将光标置于起始位置，再 `ctrl + shift + L/R`
- 重复选中：先选中一个内容，然后 `⌘ + d` 选中下一处
- 跳过选中：先 `⌘ + d` 选中要跳过的内容，然后 `⌘ + k + d` 取消选中
- 行冒泡：`⌘ + ctrl + U/D`
- 行缩进：Edit → Line → Reindent
- 行跳转：`⌘ + L/R`
- 行选中：`⌘ + shift + L/R`

#### 2.4 Other

- 搜索：`enter` 下个匹配，`shift + enter` 上个匹配
- 多块窗口：`⌘ + opt + 1/2/3/4/5`
- 切换页面：`⌘ + opt + L/R`
- Data URIs：（将二进制文件如 PNG 和 JPG 转换为文本字符串）`ctrl + shift + d`

### 参考
1. [Flight Manual: Atom Documentation](http://flight-manual.atom.io/getting-started/)
2. [GitHub package](http://flight-manual.atom.io/using-atom/sections/github-package/)
3. [Sublime Text Power User](https://sublimetextbook.com/)
