# 中间件部分

该部分主要写了四个类：CorsConfig（主要来写cors配置文件，允许的HTTP方法列表，允许的请求头列表）、CorsMiddleware（cors中间件）、Middleware（中间件模板）、MiddlewareChain（中间件链）

虽然目前只有一个中间件，但还是实现了链式，这样加新功能的时候不需要改底层架构。这就是“开闭原则”的一种体现：对扩展开放对修改关闭

## Middleware

主要进行中间件的模板化，包括中间前处理，后处理和下一个中间件的记录
```cpp
class Middleware 
{
public:
    virtual ~Middleware() = default;
    
    // 请求前处理
    virtual void before(HttpRequest& request) = 0;
    
    // 响应后处理
    virtual void after(HttpResponse& response) = 0;
    
    // 设置下一个中间件
    void setNext(std::shared_ptr<Middleware> next) 
    {
        nextMiddleware_ = next;
    }

protected:
    std::shared_ptr<Middleware> nextMiddleware_;
};
```

## MiddlewareChain

基于链式保存中间序列，并且按顺序对接受的消息进行中间件处理。注意：processBefore时是正向读取中间件中的消息，而processAfter需要往消息中写中间件信息，所以需要反向的写。（一层一层的处理）
```cpp
class MiddlewareChain 
{
public:
    void addMiddleware(std::shared_ptr<Middleware> middleware);
    void processBefore(HttpRequest& request);
    void processAfter(HttpResponse& response);

private:
    std::vector<std::shared_ptr<Middleware>> middlewares_;
};
```
## CorsMiddleware、CorsConfig

先说背景：CORS 是什么  当前端页面和后端不是同源时，浏览器会检查后端是否允许跨域。  

常见会有两类请求：  

1.普通跨域请求  比如前端直接发：  
```http
POST /chat/send
Origin: http://localhost:3000
```
这种请求希望后端返回时带上：
```http
Access-Control-Allow-Origin: ...
```

2.预检请求:浏览器在真正发 POST、带自定义 Header、带 Cookie 的请求前，可能会先发一个：
```http
OPTIONS /chat/send
Origin: http://localhost:3000
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Content-Type
```
这个 OPTIONS 请求不是业务请求，它是浏览器在问：“我等会儿能不能跨域发这个真正的请求？”

情况 1：浏览器发预检请求 OPTIONS流程是：

1.请求进来  
2.CorsMiddleware::before() 发现是 OPTIONS  
3.直接构造预检响应  
4.throw response  
5.后面的 Router 和 Handler 不再执行  
6.浏览器收到允许跨域的信息  

情况 2：浏览器发真正的 POST /chat/send流程是：

1.请求进来  
2.before() 发现不是 OPTIONS  
3.放行，进入 Router 和 Handler  
4.ChatSendHandler 处理业务，生成 JSON 响应  
5.after() 给这个响应补上 CORS 头  
6.返回客户端  

before() 的作用是：专门拦截并直接处理 CORS 预检 OPTIONS 请求  
after() 的作用是：给正常业务响应统一补充 CORS 相关响应头  

