# 用户登录管理

负责单个用户信息的注册、读取过程

## Session类：

核心思想：存储单个会话的数据，每个 session 对应一个 sessionId，内部保存用户登录状态等键值对数据，并记录过期时间。通过 SessionManager 统一管理。

### session基础信息处理部分：

1.构造函数：Session(const std::string& sessionId, SessionManager* sessionManager, int maxAge = 3600):作用：初始化一个 session 对象，设置 sessionId、所属的 SessionManager、最大存活时间 maxAge，并在创建时调用 refresh() 设置过期时间

2.getId() const：返回当前 session 的唯一标识 sessionId

3.isExpired() const：判断当前 session 是否过期，本质是将当前时间与 expiryTime_ 比较

4.refresh()：刷新 session 过期时间，将 expiryTime_ 更新为“当前时间 + maxAge 秒”

5.setManager(SessionManager* sessionManager)：为当前 session 设置所属的 SessionManager

6.getManager() const：返回当前 session 所属的 SessionManager

### session数据存储部分：

1.setValue(const std::string& key, const std::string& value)：向 session 内部的 data_ 中写入键值对。若当前 session 绑定了 SessionManager，写入后会自动调用 updateSession(...) 进行保存

2.getValue(const std::string& key) const：根据 key 从 data_ 中取出对应 value，若不存在则返回空字符串

3.remove(const std::string& key)：删除 session 中指定 key 对应的数据

4.clear()：清空当前 session 中保存的所有键值对数据

## SessionManager类：

核心思想：统一管理多个 session，负责根据 HTTP 请求中的 Cookie 获取 sessionId，从存储层中加载已有 session，或者创建新的 session，并把新的 sessionId 写回响应头中的 Cookie。

### SessionManager基础处理部分：

1.构造函数：SessionManager(std::unique_ptr<SessionStorage> storage)
作用：初始化 SessionManager，并接入底层的 session 存储器 storage_，同时初始化随机数生成器 rng_，用于生成新的 sessionId

### session获取与创建部分：

2.getSession(const HttpRequest& req, HttpResponse* resp)：
核心函数。先从请求头 Cookie 中提取 sessionId，再到 storage_ 中查找对应 session。  
如果找到且未过期，则返回已有 session  
如果没找到或已过期，则创建新 session  
创建新 session 时会调用 setSessionCookie(...) 把新的 sessionId 写入响应头  
最后调用 refresh() 刷新过期时间，并保存到存储器中  

3.generateSessionId()：生成新的 sessionId。当前实现是生成一个 32 位十六进制字符串，保证 session 标识尽量唯一

### session销毁与维护部分：

4.destroySession(const std::string& sessionId)：根据 sessionId 从存储器中删除对应 session

5.cleanExpiredSessions()：用于清理过期 session。当前代码中只是预留接口，真正清理逻辑依赖具体存储实现

6.updateSession(std::shared_ptr<Session> session)：更新 session，本质是调用 storage_->save(session) 将修改后的 session 保存回存储层

### Cookie处理部分：

7.getSessionIdFromCookie(const HttpRequest& req)：
从请求头的 Cookie 字段中解析出 sessionId= 后面的值，用于后续匹配 session

8.setSessionCookie(const std::string& sessionId, HttpResponse* resp)：
将新的 sessionId 写入响应头中的 Set-Cookie 字段，格式为：

9.sessionId=xxxx; Path=/; HttpOnly
这样浏览器后续请求时就会自动带上这个 Cookie

## SessionStorage类：

核心思想：作为 session 存储层接口，负责 session 的保存、读取和删除。当前项目中实现的是基于内存的 MemorySessionStorage。

### SessionStorage接口部分：

1.save(std::shared_ptr<Session> session)：保存 session 到存储层中

2.load(const std::string& sessionId)：根据 sessionId 从存储层中读取对应 session

3.remove(const std::string& sessionId)：根据 sessionId 从存储层中删除 session

### MemorySessionStorage内存实现部分：

4.save(std::shared_ptr<Session> session)：
将 session 以 sessionId -> session对象 的形式保存到内存中的 sessions_ 哈希表

5.load(const std::string& sessionId)：
根据 sessionId 从 sessions_ 中查找对应 session。

如果找到且未过期，则返回该 session  
如果找到但已过期，则先删除再返回 nullptr  
如果没找到，也返回 nullptr  

6.remove(const std::string& sessionId)：
根据 sessionId 从 sessions_ 哈希表中移除对应 session

