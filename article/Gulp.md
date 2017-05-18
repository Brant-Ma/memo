## Gulp
> 尽管很少用了 但 gulp 真的挺棒

### 安装
全局安装：

	$ npm rm --global gulp
	$ npm install --global gulp-cli

本地安装：

	$ npm install --save-dev gulp

在项目根目录下创建 `gulpfile.js` 文件：

	var gulp = require('gulp');
	
	gulp.task('default', function() {
	  console.log('it works!');
	});

在项目的目录下使用命令行进行测试：

	$ gulp

### Glob 语法
node-glob 是 glob 的一种 JavaScript 实现，用来匹配文件。

基本的元字符：`*` 匹配 0 个以上的字符，`?` 匹配 1 个字符，二者不再是正则中的量词。

解析前，大括号 `a{/b/c,bcd}` 将被扩展为 `a/b/c` 和 `abcd`；中括号 `[]` 表示字符组，首字符为 `!` 或 `^` 表示任意一个非字符组内出现的字符。

小括号 `(a|b|c)` 表示的分支结构与其他元字符配合使用：`!(a|b|c)` 匹配任何括号内没出现的模式；`?(a|b|c)` 匹配 0 个或 1 个模式；`+(a|b|c)` 匹配 1 个模式以上；`*(a|b|c)` 匹配 0 个模式以上，`@(a|b|c)` 匹配 1 个模式。

点号在匹配路径时的默认机制：`a/.*/c` 可以匹配 `a/.abc/c`，但 `a/b/c` 无法匹配到它；星号在路径中连用时不再表示字符，而是路径，如 `**` 匹配 0 个以上的路径。

### API
`gulp.src()`：从指定路径读取数据，并输入到管道。通常参数只需要文件路径。

`gulp.dest()`：从管道读取数据，并输出到指定路径。通常参数只需要文件路径。

输入和输出需要配合使用，而且需要 pipe 和 plugin 的参与。一个任务中也可以多次输出：

	gulp.src(/*source*/)
	  .pipe(onePlugin())
	  .pipe(gulp.dest(/*one destination*/))
	  .pipe(anotherPlugin())
	  .pipe(gulp.dest(/*another destination*/));

`gulp.task(name[,deps][,fn])`：用于定义一个任务。参数分别为：任务名的字符串，任务的前置依赖（并行执行的任务数组），任务的具体操作函数。

任务默认是一次性全部并行执行（除非 `fn` 接收 callback，或者返回 promise 或 stream），如果需要特定的顺序，可以通过前置任务的形式来执行顺序，甚至在默认任务中也可以保证顺序：

	var gulp = require('gulp');
	
	gulp.task('task1', function() {
	  // do something
	});
	
	gulp.task('task2', ['task1'], function() {
	  // do something if task1 is done
	});
	
	gulp.task('default', ['task1', 'task2']);

`gulp.watch()`：监视文件，在文件发生改变时执行任务，并触发 `change` 事件。除了文件路径参数，还可以添加一个需要执行的任务数组，或者用于响应事件的回调函数。

如果传入的是任务数组：

	var watcher = gulp.watch('js/**/*.js', ['uglify', 'reload']);
	
	watcher.on('change', function(event) {
	  console.log('File ' + event.path + ' was ' + event.type);
	});

如果传入的是回调函数：

	gulp.watch('js/**/*.js', function(event) {
	  console.log('File ' + event.path + ' was ' + event.type);
	});

其中 `event.path` 是触发了事件的文件的路径，`event.type` 是触发的事件类型（`added` `changed` `deleted` `renamed`）

### CLI
执行默认任务：

	$ gulp

执行指定的单个任务，或者并发执行指定的多个任务：

	$ gulp <task> <task> ...

显示 gulp 版本：

	$ gulp -v

### 参考
1. [gulp documentation](https://github.com/gulpjs/gulp/tree/master/docs)
2. [node-glob documentation](https://github.com/isaacs/node-glob)
