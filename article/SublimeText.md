## Sublime Text
> 摆脱重量级的 IDE，使用轻量级的 editor

### Install
- 安装 package control: 打开 console, 复制 [代码](https://packagecontrol.io/installation) 完成安装
- 安装包: 在命令面板，输入 `install` 匹配 `Package control: Install Package` 进行自动安装；输入 `add` 匹配 `Package Control: Add repository` 进行手动安装

### Palette
- 命令面板：Command Palette (`⌘ + shift + p`)
- 定位面板：Goto Anything Palette (`⌘ + p`)
- 各种定位：在定位面板可以直接定位文件；输入 `:` 可以行定位；输入 `@` 可以块定位；输入 `#` 可以模糊搜索
- 改变语法：在命令面板输入 `ss` 匹配 `Set Syntax` 列表
- 选用片段：在命令面板输入 `snip` 匹配 `Snippet` 列表

### Setting & Customization
- 个性化设置：直接键入 `⌘ + ,` 并参考左侧的 Default 设置右侧的 User
- 设置定位：`binary_file_patterns` 属性
- 小地图的边框: `draw_minimap_border` 属性
- 侧边栏的开关: 键入 `⌘ + k` && `⌘ + b`

### Multiple Panes
- 多块窗口：`⌘ + opt + 1/2/3/4/5`
- 切换页面：`⌘ + opt + L/R`

### Multiple Carets
- 创造光标：按住 `⌘` 并在需要的位置单击鼠标
- 内容选中：先将光标置于起始位置，再 `ctrl + shift + L/R`
- 重复选中：先选中一个内容，然后 `⌘ + d` 选中下一处
- 跳过选中：先 `⌘ + d` 选中要跳过的内容，然后 `⌘ + k + d` 取消选中

### Search
- 搜索：`⌘ + f` 进行搜索，`⌘ + shift + f` 全局搜索，`enter` 下个匹配，`shift + enter` 上个匹配

### Fold
- 折叠 / 展开：`⌘ + opt + [` / `⌘ + opt + ]`

### Line Tricks
- 行冒泡：`⌘ + ctrl + U/D`
- 行缩进：Edit → Line → Reindent
- 行连接：`⌘ + j`
- 行复制：`⌘ + shift + d`
- 行删除：`ctrl + shift + k`
- 行包裹：`ctrl + shift + w`
- 行跳转：`⌘ + L/R`
- 行选中：`⌘ + shift + L/R` 

### Emmet Tricks
- Data URIs are a way to convert binary files (usually PNGs and JPGs) into a single string of text. `ctrl + shift + d`

### 参考
1. [Sublime Text Power User](https://sublimetextbook.com/)
