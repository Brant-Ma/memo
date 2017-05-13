## MV\* Pattern
> 理解 MVC MVP MVVM

### 问题与抽象
- 问题：GUI 应用程序为用户提供了可视化的操作界面，用户行为会执行一些应用逻辑。而应用逻辑可能会触发业务逻辑对数据的变更，且数据的变更需要同步至用户界面以提供准确的信息
- 抽象：基于职责分离，对应用程序进行分层，用户界面的层次称为 View，数据抽象的层次称为 Model
- MV\* 模式：为了解决 GUI 应用程序的复杂性管理问题的架构模式。从 View 到 Model 的逻辑和从 Model 到 View 的同步，是 MV 模式需要解决的问题

### MVC
- 调用关系：View 捕获到用户的操作后，将控制权移交给 Controller；Controller 执行相应的应用逻辑，如对来自 View 的数据进行预处理、决定调用哪个 Model 的接口；Model 执行相应的业务逻辑，对数据进行处理，但不直接操作 View；View 通过观察者模式收到 Model 变更的消息后，向 Model 请求数据并更新视图

### MVP
- 调用关系：View 捕获到用户的操作后，将控制权移交给 Presenter；Presenter 执行相应的应用逻辑，并对 Model 进行相应的操作；Model 执行相应的业务逻辑后，通过观察者模式把自己变更的消息传递给 Presenter；Presenter 获取到 Model 变更的消息后，通过 View 提供的接口更新视图

### MVVM
- 调用关系：View 捕获到用户的操作后，将控制权移交给 ViewModel；ViewModel 执行相应的应用逻辑，并对 Model 进行相应的操作；Model 执行相应的业务逻辑；ViewModel 的 Binder 通过双向数据绑定来自动同步 View 和 Model 之间的数据更新


### 参考
1. [界面之下：还原真实的 MV\* 模式](https://github.com/livoras/blog/issues/11)
2. [前端 MVC 已死？](https://blog.yongyuan.us/articles/2016-10-15-mvc-dead/)
