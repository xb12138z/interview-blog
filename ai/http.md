# http关键函数

Http核心分为四大类：HttpRequest类负责提供接口进行Http请求的具体读取和保存，HttpResponse类负责Http请求的回复，HttpContext类负责Http请求的读取，Httpserver负责

## HttpRequest类：

核心思想：设置Request状态，存储每次通信中的数据。因为每次使用后重置，所以内部储存数据不用对应相连，都是用于同一次通信  
setReceiveTime(muduo::Timestamp t)：记录请求开始时间t

### 请求行处理部分：

1.设置状态函数setMethod(const char *start, const char *end)：start、end中字符为状态

2.setPath(const char *start, const char *end)：URL中有？时，？前的作为路径，不带？整体是路径

3.setQueryParameters(const char *start, const char *end)：URL中有？时，？后的作为路径的参数

4.setVersion(std::string v)：根据传来的v设置HTTP版本

### 请求头处理部分：

1.addHeader(const char *start, const char *colon, const char *end)：colon为':'位置将一行请求内容记录为key：value

2.getHeader(const std::string &field) const： 根据传入的key返回对应的value

3.method：返回当前请求状态

4.setContentLength(uint64_t length)将读取到的length写入




## HttpContext类：  

核心思想：通过请求解析行、请求解析头、请求解析体、完成解析四个状态的切换完成Http消息的解析。  
持有一个Httprequest类对象（每次使用后重置清空），和重置状态函数（不写了）
```
POST /login HTTP/1.1

Host: localhost
Content-Type: application/json
Content-Length: 32

{"username":"a","password":"b"}
```

### http请求解析函数：bool HttpContext::parseRequest(Buffer *buf, Timestamp receiveTime):  

1.找到crlf（行结束标志）后，认为解析请求行完成，然后利用processRequestLine函数解析请求行。然后记录接受请求的时间，并且buf->retrieveUntil(crlf + 2);让buf指向请求头，切换状态为解析请求头  

2.切换为解析请求头函数后，开始找':'如果有的话，调用HttpRequest中的函数解析，然后继续循环解析。如果解析完毕如果request中请求为PUT或者POST，就去request中找contentLength，不为0就切换为解析请求体，为0切换为解析完成，找不到则为HTTP语法错误  

3.判断buf中的数据长度是否满足contentLength，满足的话更改状态为完成解析，不满足不更改状态持续循环。



### 解析请求行函数：bool HttpContext::processRequestLine(const char *begin, const char *end)：  
GET /user/info?id=123&name=tom HTTP/1.1  
找空格，按照空格解析，调用HttpRequest中的函数  
如例子URL为/user/info?id=123&name=tom，需要调用函数将/user/info设置为PATH，将id=123&name=tom设置为setPathParameters，便于数据库的查询


## HttpResponse类：

核心思想：设置返回消息状态，是否关闭连接参数closeconnection

### 响应行

1.构造函数：HttpResponse(bool close = true)：将消息设置为默认的kUnknown状态，并且默认关闭closecoonection

2.setVersion(std::string version)：设置消息的version

3.etStatusCode(HttpStatusCode code)：设置消息的statue（404）

4.setStatusMessage(const std::string message)：设置消息的statue message（no found）

### 响应头

5.setCloseConnection(bool on)：打开closeconnection，关闭连接

6.setContentType(const std::string& contentType)：写入文本类型

7.setContentLength(uint64_t length)设置响应内容类型

8.addHeader(const std::string& key, const std::string& value)，在map中记录key、value键值对。用于响应头中的参数记录。eg（addHeader("Content-Type", "application/json");）

### 响应体

9.setBody(const std::string& body)设置返回body响应体

10.setStatusLine(const std::string& version, HttpStatusCode statusCode,const std::string& statusMessage);：同时设置消息版本，状态，状态文本

### 响应对象输出

11.appendToBuffer(muduo::net::Buffer* outputBuf)，将所有存储的响应消息都append到outputBuf中。

## HttpServer

核心思想：
封装底层 Muduo 的 TCP 服务，负责接收连接、接收数据、解析 HTTP 请求、执行中间件与路由分发，并将处理结果封装成 HttpResponse 返回给客户端。在构造函数中埋好处理响应、建立链接、读取消息的回调函数，之后自动完成读取。

### 服务启动与基础配置

1.构造函数：HttpServer(int port, const std::string& name, bool useSSL = false, muduo::net::TcpServer::Option option = ...)：初始化监听地址、创建底层 TcpServer、设置是否启用 SSL、默认将 HTTP 请求处理回调绑定到 handleRequest(...)、调用 initialize() 注册底层连接/消息回调

2.setThreadNum(int numThreads)：设置底层服务器的工作线程数、用于提高并发处理能力

3.start()：启动底层服务器、进入事件循环，开始正式监听和处理客户端请求

4.getLoop()：获取底层事件循环对象 EventLoop

5.enableSSL(bool enable)：设置是否启用 SSL

6.setSslConfig(const ssl::SslConfig& config)：设置 SSL 配置、初始化 SSL 上下文

### HTTP请求处理回调

7.setHttpCallback(const HttpCallback& cb)：设置 HTTP 请求总处理回调函数、默认是 handleRequest(...)

### 路由注册接口

8.Get(const std::string& path, const HttpCallback& cb):注册 GET 静态路由，对应回调函数式处理

9.Get(const std::string& path, router::Router::HandlerPtr handler):注册 GET 静态路由，对应对象式 Handler 处理

10.Post(const std::string& path, const HttpCallback& cb):注册 POST 静态路由，对应回调函数式处理

11.Post(const std::string& path, router::Router::HandlerPtr handler):注册 POST 静态路由，对应对象式 Handler 处理

12.addRoute(HttpRequest::Method method, const std::string& path, router::Router::HandlerPtr handler):注册动态路由，对应对象式 Handler 处理

13.addRoute(HttpRequest::Method method, const std::string& path, const router::Router::HandlerCallback& callback):注册动态路由，对应回调函数式处理

### Session与中间件管理

14.setSessionManager(std::unique_ptr<session::SessionManager> manager):设置 Session 管理器\用于登录态和会话管理

15.getSessionManager()
作用：
获取当前 Session 管理器

16.addMiddleware(std::shared_ptr<middleware::Middleware> middleware):添加中间件到中间件链中\请求处理前后会依次执行\底层连接与消息处理

### 核心执行函数

#### initialize()
作用：
注册底层 TCP 连接回调 onConnection(...)
注册底层消息回调 onMessage(...)

#### onConnection(const muduo::net::TcpConnectionPtr& conn)
作用：
处理客户端连接建立与断开
如果启用 SSL，则创建对应的 SslConnection
给连接绑定一个 HttpContext

#### onMessage(const muduo::net::TcpConnectionPtr& conn, muduo::net::Buffer* buf, muduo::Timestamp receiveTime)
作用：
处理收到的网络数据
如果启用了 SSL，先解密数据
再交给 HttpContext 解析 HTTP 请求
当请求完整时调用 onRequest(...)

#### onRequest(const muduo::net::TcpConnectionPtr& conn, const HttpRequest& req)
作用：
创建 HttpResponse
调用 HTTP 请求总处理回调
将响应对象拼装成 HTTP 报文并发送回客户端
如果是短连接，则关闭连接

#### handleRequest(const HttpRequest& req, HttpResponse* resp)
作用：
执行中间件前置处理
调用 Router 匹配路由
找到对应 Handler 或回调函数进行业务处理
执行中间件后置处理
若未匹配路由则返回 404
