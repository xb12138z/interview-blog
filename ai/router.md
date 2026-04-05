# 路由

## router

核心思想：名字叫路由，但本质上其实是Http指令读取完毕后，处理指令的异步调用接口，分为了回调函数、静态路由、动态路由三种。路由与回调函数本质一致但是由于回调函数不适合长代码，所以分开，路由指向单独的类来执行。而动态体现在指令中存在变量的情况

### 数据结构

1.struct RouteKey（路由键）： 负责存储请求回复的enum类型，和请求的处理路径。同时重写了==，辅助判断路由键是否相等

正则路由键:额外存储对应执行的函数
```cpp
 struct RouteCallbackObj
    {
        HttpRequest::Method method_;
        std::regex pathRegex_;
        HandlerCallback callback_;
        RouteCallbackObj(HttpRequest::Method method, std::regex pathRegex, const HandlerCallback &callback)
            : method_(method), pathRegex_(pathRegex), callback_(callback) {}
    };

    struct RouteHandlerObj
    {
        HttpRequest::Method method_;
        std::regex pathRegex_;
        HandlerPtr handler_;
        RouteHandlerObj(HttpRequest::Method method, std::regex pathRegex, HandlerPtr handler)
            : method_(method), pathRegex_(pathRegex), handler_(handler) {}
    };
```

2.struct RouteKeyHash（自定义哈希散列函数）：

```cpp
size_t operator()(const RouteKey &key) const
        {
            size_t methodHash = std::hash<int>{}(static_cast<int>(key.method));//enum类型转化为size_t
            size_t pathHash = std::hash<std::string>{}(key.path);
            return methodHash * 31 + pathHash;
        }
```

3.路由和回调函数的注册
```cpp
    using HandlerPtr = std::shared_ptr<RouterHandler>;
    using HandlerCallback = std::function<void(const HttpRequest &, HttpResponse *)>;
```

### 注册

4.registerHandler(HttpRequest::Method method, const std::string &path, HandlerPtr handler)：静态注册路由

5.registerCallback(HttpRequest::Method method, const std::string &path, const HandlerCallback &callback)：注册回调函数

6.addRegexHandler(HttpRequest::Method method, const std::string &path, HandlerPtr handler)：注册动态路由，先将path转化成正则表达式

7.addRegexCallback(HttpRequest::Method method, const std::string &path, const HandlerCallback &callback)：注册动态回调


8.route(const HttpRequest &req, HttpResponse *resp)：路由处理函数，先在静态路由表中搜索是否存在对应路径的静态路由，而后查找回调函数，最后查找动态路由和动态路由回调函数



## routerHandler

RouterHandler 就像一个接口协议：

它不负责具体业务  
它只规定“所有路由处理器都必须会 handle(req, resp)”  