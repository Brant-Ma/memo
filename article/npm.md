## npm
> npm 不是 Node Package Manager 的缩写，是 npm is not an acronym 的简写

### 1. 安装 npm

作为包管理工具，npm 可以帮你复用别人的代码，分享自己的代码，以及管理不同版本的代码。npm 伴随 Node.js 的安装（可通过 Homebrew 安装）而出现，但最好升级一下：

```
$ sudo npm install npm@latest -g
$ npm -v
```

通常 npm 目录的路径会是 `/usr/local`，可以测试一下：

```
$ npm config get prefix
```

### 2. 管理 package

管理 package 的基本操作类型无外乎：package 的安装、更新和卸载。

#### 2.1 安装 npm 包

本地安装（locally）可通过 `require()` 引入到其他包；全局安装（globally）可直接在用命令行中使用。安装命令为：

```
$ npm install <package>           # 将包放在 ./node_modules 下
$ npm install  -g <package>       # 将包放在 /usr/local 下
```

本地项目使用 `package.json` 来管理依赖包，必须有的字段为 `name` 和 `version`。创建命令为：

```
$ npm init             # 命令行问答式的创建文件
$ npm init --yes       # 直接创建默认的配置文件
```

其中 `dependencies` 字段列出生产环境所依赖的包，`devDependencies` 字段列出仅用于开发环境和测试环境的包。通常不必手动列出，只需在安装一个包时明确即可：

```
$ npm install <package> --save           # 安装并添加至运行时依赖
$ npm install <package> --save-dev       # 安装并添加至开发时依赖
```

特别需要注意几点：

- `$ npm install` 命令会安装 `dependencies` 和 `devDependencies` 中的所有依赖项
- 更新到 npm 5 后，一个新改进是：`$ npm install <package>` 将 `--save` 作为了默认行为

如果 `package.json` 文件的依赖字段明确了某个包的版本，那么 npm 的安装或更新命令将按此版本安装或更新此包；如果没有 `package.json` 文件，或者文件的依赖字段未明确某个包的版本，则默认安装或更新此包的最新版本。

#### 2.2 更新 npm 包

包发生更新时，项目可能需要与之保持同步。本地更新的相关命令：

```
$ npm outdated [<package>]       # 检查全部的包（指定的包）是否过时
$ npm update [<package>]         # 更新全部（指定）的包
```

对应的全局更新的相关命令：

```
$ npm outdated -g  --depth=0 [<package>]       # 检查全部的包（指定的包）是否过时
$ npm update -g [<package>]                    # 更新全部（指定）的包
```

#### 2.3 卸载 npm 包

可以将包从项目目录中删除。本地删除的相关命令：

```
$ npm uninstall <package>                  # 仅从 ./node_modules/ 中删除
$ npm uninstall --save <package>           # 删除并从 package.json 的 dependencies 中移除
$ npm uninstall --save-dev <package>       # 删除并从 package.json 的 devDependencies 中移除
```

对应的全局删除的相关命令：

```
$ npm uninstall -g <package>       # 从全局移除包
```

### 3. 共享 module

包是可以共享的代码单元，而共享的关键在于如何发布包，以及如何解决版本控制问题。

#### 3.1 包与模块

包（package）和模块（module）是一对容易混淆的概念：

- 包：被 `package.json` 文件描述的文件或目录。可以是压缩包形式，或者是 url 形式，甚至是 npm 标识符形式
- 模块：可以被 Node.js 的 `require()` 加载的文件或目录。可以是一个包（但其 `package.json` 文件必须含有 `main` 字段），或者包含 `index.js` 文件的目录，甚至是一个 JS 文件

大部分的 npm 包都是模块且包含有模块。其中，`package.json` 文件定义了包，`node_modules` 目录是 Node.js 寻找模块的场所。

#### 3.2 发布模块

发布前，需要确认三件事。一个是你是否为注册用户，可以通过 `https://npmjs.com/~` 检查；一个是模块是否存在冗余文件，可以通过 `.gitignore` 文件来忽略冗余；一个是模块名是否被占用，可以通过 `https://npmjs.com/package/<package>` 检查。

```
$ npm publish       # 发布包
```

发布后，可以更新模块后再次发布。更新命令会根据更新类型自动修改 `package.json` 文件中的 `version` 字段。

```
$ npm version <update_type>       # 更新模块
$ npm publish                     # 再次发布
```

#### 3.2 版本控制

语义化版本（Semantic Versioning）用于告知某次 release 的更改所属的类型。

- 主版本（major）：允许不再兼容的大更新
- 次版本（minor）：必须向前兼容的小更新
- 修订版本（patch）：用于 bug 修复，或者其他小的修改

完整的版本格式为：`<major>.<minor>.<patch>`，首次发布的版本应该为 `1.0.0`。

### 4. 构建项目

npm 除了管理 package，还可以利用 npm scripts 来构建项目。

#### 4.1 npm scripts 原理

npm scripts 是定义在 `package.json` 文件的 `scripts` 字段中的脚本。`scripts` 字段的每个属性名对应一个命令，每个属性值对应一个脚本。

比如下面的例子，如果在命令行中执行 `$ npm run clean` 将等同于执行 `$ rm -r dist/*`。

```
// package.json
{
  ...
  "scripts": {
    "clean": "rm -r dist/*"
  }
  ...
}
```

可以看出，npm 最大的好处在于可以将项目的脚本集中在一起（全部放在 `script` 对象中），并提供统一的对外接口（执行 `$ npm run <property>`）。具体而言，每次执行 `npm run` 命令都会在一个新建的 shell 里执行指定的脚本。命令对脚本的类型并无要求，只要是能在 shell 中运行的可执行文件就行。

那么 shell 是如何寻找命令路径的呢？其实 `npm run` 命令在执行的时候，会临时将当前目录的 `node_modules/.bin` 目录加入到 `PATH` 环境变量中。最大的好处是，项目的本地依赖所提供的可执行二进制文件可以直接被使用，比如：

```
// 不必这么写：
"test": "./node_modules/.bin/mocha test"

// 可以简写为：
"test": "mocha test"
```

#### 4.2 使用 npm 脚本

npm 默认为生命周期内的一些内置命令提供了前后两个钩子。比如执行 `$ npm run build` 时，实际上会按顺序运行三个命令：

```
npm run prebuild && npm run build && npm run postbuild
```

最常用的四个脚本（`start`、`stop`、`test`、`restart`） 具有简写形式，可以省掉 run 一词来更快的书写。比如 `$ npm run test` 可以简写为 `$ npm test`。

常用的 npm 脚本如下：

```
// 清空发布目录
"clean": "rimraf ./dist/*"

// 搭建本地服务器
"serve": "http-server -p 7777 ./dist"

// 打开浏览器页面
"open:dev": "opener http://localhost:7777"

// 页面实时刷新
"livereload": "live-reload --port 7777 ./dist"
```

### 参考
1. [npm Documentation](https://docs.npmjs.com/)
2. [npm scripts 使用指南 - 阮一峰](http://www.ruanyifeng.com/blog/2016/10/npm_scripts.html)
3. [论版本号的正确打开方式](http://taobaofed.org/blog/2016/08/04/instructions-of-semver/)
4. [Difference Between Update and Upgrade](http://www.differencebetween.net/technology/difference-between-update-and-upgrade/)
