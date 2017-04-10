## RESTful 
> RESTful 是一种架构风格，目标是构建可扩展的 web 服务

### 理解
- 极简：用 URL 定位资源（名词），用 HTTP 描述操作（动词）
- REST: 表现层状态转移（REpresentational State Transfer）的主语是 Resource, 因此整体的意思是资源在网络中以某种表现形式进行状态转移，而 client 和 server 之间传递的是某个资源的表现形式
- RESTful API: server 为各种 client (web, iOS, Android) 提供一套统一的服务接口
- 架构特征：面向资源（Resource Oriented），可寻址（Addressability），连通性（Connectedness），无状态（Statelessness），统一接口（Uniform Interface），超文本驱动（Hypertext Driven）

### 关键词
- 资源（Resource）：资源是服务器上的抽象概念；URI 既是资源的名称，也是资源的地址
- 表述（Representation）：资源的表述是对资源状态的描述；表述可以有多种格式，且可以通过协商机制确定
- 状态转移（State Transfer）：在客户端和服务器之间转移代表资源状态的表述；通过转移和操作资源的表述，来间接实现操作资源的目的
- 统一接口（Uniform Interface）：对资源的操作必须通过统一的接口，包括 7 个 HTTP 方法，HTTP 头信息，HTTP 响应状态码，内容协商机制，缓存机制，客户端身份认证机制；操作语义必须由 HTTP 消息体之前的部分完全表述
- 超文本驱动（Hypertext Driven）：web 应用是由很多状态组成的有限状态机；资源之间通过超链接相互关联，超链接既代表资源之间的关系，也代表可执行的状态迁移，超媒体中不仅包含数据，还包含了状态迁移的语义；以超媒体作为引擎，驱动 web 应用的状态迁移（HATEOAS）

### 架构约束
- 客户端-服务器（Client-Server）：通信只能由客户端单方面发起，表现为请求-响应的形式
- 无状态（Stateless）：通信的会话状态（Session State）应该全部由客户端负责维护
- 可缓存（Cashable）：响应内容可以在通信链的某处被缓存，以改善网络效率
- 统一接口（Uniform Interface）：通信链的组件之间通过统一的接口相互通信，以提高交互的可见性
- 分层系统（Layered System）：通过限制组件的行为（每个组件只能看到与其交互的紧邻层），将架构分解为若干等级的层
- 支持按需代码（Code-On-Demand，可选）：支持通过下载并执行一些代码（如JavaScript），对客户端的功能进行扩展

### 设计细节
- URL root 和 API version: `example.com/api/v1/*` 或者 `api.example.com/v1/*`
- URL 使用嵌套结构；每个路径段里，使用名词而非动词，且使用复数而非单数：`/friends/12345`
- URL 中反映资源之间的关系：`/tickets/12345/messages/67890`
- 尽可能将不太符合 CRUD 的操作构造为 RESTful 风格：文章收藏（`PUT /articles/123/star`）与取消收藏（`DELETE /articles/123/star`）
- 对资源的抽象要有合适的粒度，可以根据网络的效率，表述的大小，客户端使用时的易用程度来评判
- 需要针对返回资源的大小考虑进行分页（pagination）或加入限制（limit）
- 安全性：身份认证（考虑 HTTPS 或 OAuth2）；数据加密（考虑 HTTPS 或加盐）；访问授权（考虑代码实现）
- 输入 JSON 而非 URL 编码，因为后者缺乏数据类型和层次结构的支持；输出 JSON 而非 XML，因为核心需求是数据的序列化而非扩展性

### 简单示例
- 整体获取：`GET /messages`
- 单个获取：`GET /messages/7`
- 单个创建：`POST /messages`
- 单个更新：`PUT /messages/7`
- 部分更新：`PATCH /messages/7`
- 单个删除：`DELETE /messages/7`

### 查询组件
- 过滤：`GET /tickets?state=open`（过滤出状态为打开的票）
- 排序：`GET /tickets?sort=-priority`（按照优先级降序排序）
- 搜索：`GET /tickets?q=return`（包含 return 的资源）
- 综合使用：`GET /tickets?q=return&state=open&sort=-priority`

### 成熟度模型
- 第 0 级：使用 HTTP 作为传输方式
- 第 1 级：引入 Resource 概念
- 第 2 级：支持 HTTP 动词
- 第 3 级：使用超媒体作为应用状态引擎（HATEOAS）

### 安全性和幂等性
(img)

### 参考
- [理解本真的 REST 架构风格]
- [Best Practices for Designing a Pragmatic RESTful API]
