## ESLint
> 非强制的 style guide 或许并不能解决问题

ESLint 是什么？官网是这么描述的：ESLint 是一个用于识别和报告 JavaScript 代码中发现的模式的工具，通过 ESLint 可以使代码更加一致并避免错误。

### 1. 安装 ESLint

像所有 npm 包一样，ESLint 既可以全局安装也可以本地安装。但是代码检查其实是基于每个项目的不同配置来进行的，如果要全局安装的话，所有的插件也需要全局安装。所以 ESLint 或许更适合本地安装：

	$ npm install eslint --save-dev

然后就可以在项目中使用 `eslint` 命令了。先设置一个配置文件：

	$ ./node_modules/.bin/eslint --init

然后命令行会使用对话风格帮你作出配置选择，最简单的方法是从几个流行的 style guide 中选择一个。目前 ESLint 提供了三个内置的选项：Airbnb、Standard 和 Google（流行程度依次递减）。

Standard 是个相当不错的风格，但规则有些强硬。而且，函数名与括号之间需要加空格的规则多少有些让人怀疑人生。相对来说，Airbnb 灵活且理性的风格可能更对我的胃口。

### 2. 配置 ESLint

我们已经有了一个几乎是空白的 `.eslintrc.json` 配置文件。现在需要做的是，完整的将它配置好。我的一个基于 electron 的项目配置如下：

	{
	  "env": {
	    "es6": true,
	    "node": true
	  },
	  "extends": "airbnb-base"
	  "rules": {
	    "semi": "off",
	    "import/no-extraneous-dependencies": [
	      "error",
	      {
	        "devDependencies": true
	      }
	    ]
	  }
	}

其中，`env` 字段用于预定义一些全局变量。最常用的有：

- `browser`：浏览器的全局变量
- `es6`：所有的 ES6 特性（除了 modules）
- `node`：Node.js 全局变量和 Node.js 作用域（scoping）

而 `extends` 字段就是上面通过 ESLint 初始化得到的一系列规则。`rules` 字段可以精细的配置一些规则。比如 Airbnb Style Guide 默认不允许加分号，但我不太喜欢这个规则。

另外，当我使用 `require('electron')` 时会提醒我，被引入的模块需要放进 `package.json` 的 `dependencies` 字段而非 `devDependencies` 字段，这就属于误判了。

### 3. 使用 ESLint

类似于许多 npm 包，ESLint 可以使用命令行来完成基本的使用：

	$ eslint [file|dir|glob]

但这种方式毕竟很不灵活：满屏的 warn 和 error 会让你心态崩溃，而且有了 ESLint 这么棒的工具后还需要手动操作实在是暴殄天物。能无声无息的融入到项目中的 Lint 工具才是最好的。

比如，提前到代码书写阶段：在编辑器中（我喜欢 Atom）安装 `linter-eslint` 插件，可以对代码风格进行实时反馈和错误提醒，能够有效的防止积压大量的不规范代码。另外，貌似 `prettier-atom` 也挺不错的。

甚至，提前到代码提交阶段：git hooks 是存储在 `.git/hooks` 目录下的 shell 脚本，它们可以在特定的事件（certain points）触发后被调用。其实在 `git init` 命令调用后这些脚本就已经存在了，只不过因为包含了 `.sample` 后缀而不能生效。也就是说，将脚本修改一下再去掉后缀就可以使之生效了。那么选择哪个钩子比较好呢？个人觉得在越早的阶段进行代码风格审查就越有意义，所以选择 pre-commit 钩子可能更加合适。另外，使用诸如 [husky](https://github.com/typicode/husky) 的模块会让 hook 更加好用。

### 参考
1. [Configuring - ESLint](http://eslint.org/docs/user-guide/configuring)
2. [airbnb/javascript: JavaScript Style Guide](https://github.com/airbnb/javascript)
3. [5-step quick start guide to ESLint](https://codeutopia.net/docs/eslint/)
4. [Integrations - ESLint](http://eslint.org/docs/user-guide/integrations)
5. [基于git hooks的前端代码质量控制解决方案](https://github.com/kuitos/kuitos.github.io/issues/28)
6. [typicode/husky: Git hooks made easy](https://github.com/typicode/husky)
