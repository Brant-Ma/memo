## Git
> 关于 Git 的一切

### 1. 安装 & 配置

这些基本的配置，通常是一劳永逸的。

#### 安装 Git
- Mac 上可以通过 Homebrew 来安装 Git: `$ brew install git`
- 查看路径：`$ which git`
- 查看版本：`$ git --version`
- 更新版本：`$ brew upgrade git`

#### 配置 Git 账号
- 配置用户名字：`$ git config --global user.name Brant-Ma`
- 配置用户邮箱：`$ git config --global user.email brant.mazhiyuan@gmail.com`
- 当前用户（全局）的配置文件路径：`~/.gitconfig`
- 当前项目（局部）的配置文件路径：`.git/config`
- 查看配置文件：`$ git config --list`

#### 配置 SSH 密钥
- 查看旧密钥：`$ ls -al ~/.ssh`
- 生成新密钥：`$ ssh-keygen -t rsa -b 4096 -C brant.mazhiyuan@gmail.com`
- 开启代理：`$ eval "$(ssh-agent -s)”`
- 添加密钥至代理：`$ ssh-add ~/.ssh/id_rsa`
- 复制密钥：`$ pbcopy < ~/.ssh/id_rsa.pub`
- 添加密钥至 Github 账号后，测试密钥连接：`$ ssh -T git@github.com`

#### 配置 .gitignore
- 临时路径：`.idea/` `.sass-cache/` `.tmp/`
- 模块路径：`bower_components/` `node_modules/`

### 2. 概念与操作

主要是 4 个概念和 N 个操作，这是 Git work flow 的基础。

#### 核心概念
- Workspace: 工作区
- Index / Stage: 暂存区
- Local Repository: 本地仓库
- Remote Repository: 远程仓库

#### 操作图解
![Git 核心操作](../image/git-core.png)

#### 仓库操作
- 初始化当前目录为 Repository：`$ git init`
- 克隆 Remote 到当前目录：`$ git clone [url]`
- 查看当前 Repository 的状态：`$ git status `

#### 文件操作
- 首次追踪目录 或将已追踪目录的修改添加至 Index：`$ git add .`
- 首次追踪目录 或将已追踪目录的修改添加至 Index：`$ git add [dir]`
- 首次追踪文件 或将已追踪文件的修改添加至 Index：`$ git add [file1] [file2] ...`
- 删除 Workspace 中的指定文件并将删除操作添加至 Index：`$ git rm [file1] [file2] ...`
- 重命名 Workspace 中的指定文件并将重命名操作添加至 Index：`$ git mv [file-from] [file-to]`

#### 提交操作
- 查看最近的提交日志：`$ git log`
- 提交 Index 到 Repository：`$ git commit -m [message]`
- 提交 Index 中的指定文件到 Repository：`$ git commit [file1] [file2] ... -m [message]`

#### 分支操作
- 列出本地分支：`$ git branch`
- 删除本地分支：`$ git branch -d [branch]`
- 切换本地分支：`$ git checkout [branch]`
- 新建本地分支：`$ git checkout --track [origin/branch]`

#### 状态操作
- 恢复 Index 的指定文件到 Workspace：`$ git checkout [file]`
- 重置 Index 和 Workspace 至上次 commit：`$ git reset --hard`
- 将已追踪文件的 Workspace 保存为工作现场：`$ git stash`
- 还原工作现场到 Workspace 并删除 stash：`$ git stash pop`

#### 标签操作
- 列出已有的标签：`$ git tag`
- 新建轻量级标签：`$ git tag [tag]`
- 向远程推送标签：`$ git push origin --tags`

#### 同步流程
- 拉取远程分支到本地仓库：`$ git fetch`
- 合并指定分支到当前分支：`$ git merge [orgin/branch]`
- 推送当前分支到指定分支：`$ git push origin [branch]`

### 3. Git work flow

这部分是最重要的：使用 Git 的实际工作流程。





### 参考
1. [廖雪峰：Git 教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
2. [颜海镜：我的 Git 笔记](http://yanhaijing.com/git/2014/11/01/my-git-note/)
3. [阮一峰：常用 Git 清单](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)
4. [顾伟刚：Git 命令大全](https://gist.github.com/guweigang/9848271)
5. [geeeeeeeeek：git-recipes](https://github.com/geeeeeeeeek/git-recipes/wiki)
