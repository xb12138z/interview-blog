# AI接入工具

负责整个项目中的ai应用

## AIconfig

负责从配置文件中加载提示词模板和工具列表。并且保存在此类中。

1.loadFromFile(const std::string& path)：从path中传入的路径中读取提示词模板和工具详情。并且存储在AIconfig类中。

2.AIConfig::buildToolList() const：将构建工具列表字符串，格式为：工具名(参数1, 参数2) ¡ú 工具描述信息

3.buildPrompt(const std::string& userInput) const 构建提示词字符串，将用户输入和工具列表填充到prompt_template中，返回最终的提示词字符串

4.parseAIResponse(const std::string& response) const 解析AI的响应，判断是否包含工具调用的信息，如果包含则提取工具名和参数，返回一个AIToolCall结构体，如果不包含则返回一个isToolCall为false的AIToolCall结构体

5.buildToolResultPrompt 构建工具结果提示词字符串，将用户输入、工具调用信息和工具结果填充到一个新的提示词中，返回最终的提示词字符串

## AIStrategy

定义了单个模型的基础架构，同时通过读取环境变量中的apikey等来进行模型的创建。有将历史消息列表补充发消息的角色，并且转化为一行的函数和将接受Json提取出消息的函数。

```cpp
class AIStrategy {
public:
    virtual ~AIStrategy() = default;// 定义接口

    
    virtual std::string getApiUrl() const = 0;// 获取API URL

    // API Key
    virtual std::string getApiKey() const = 0;// 获取API Key


    virtual std::string getModel() const = 0;// 获取模型名称


    virtual json buildRequest(const std::vector<std::pair<std::string, long long>>& messages) const = 0;// 构建请求体，参数是消息列表，返回值是JSON格式的请求体//longlong是时间戳


    virtual std::string parseResponse(const json& response) const = 0;// 解析响应，参数是JSON格式的响应体，返回值是AI的回答字符串

    bool isMCPModel = false;

};
```
## AIFactory

AI工厂实例，负责AIStrategy类的创建。作用是：工厂记录每个模型的名称和对应的类。创建时候只需要给工程名称，工厂会在表中查询对应的类，进行对应模型的创建。

```cpp
#include"AIStrategy.h"

class StrategyFactory {

public:
    using Creator = std::function<std::shared_ptr<AIStrategy>()>;//定义一个函数类型 Creator，表示创建 AIStrategy 实例的函数

    static StrategyFactory& instance();//单例模式，获取工厂实例

    void registerStrategy(const std::string& name, Creator creator);//注册策略，参数是策略名称和创建函数

    std::shared_ptr<AIStrategy> create(const std::string& name);//创建策略实例，参数是策略名称，返回值是对应的 AIStrategy 实例，如果找不到则抛出异常

private:
    StrategyFactory() = default;
    std::unordered_map<std::string, Creator> creators;//存储策略名称和创建函数的映射
};


//模板类

template<typename T>
struct StrategyRegister {//模板结构体，用于注册策略，参数是策略类 T，构造函数会调用工厂的 registerStrategy 方法注册策略
    StrategyRegister(const std::string& name) {
        StrategyFactory::instance().registerStrategy(name, [] {
            std::shared_ptr<AIStrategy> instance = std::make_shared<T>();
            return instance;
            });
    }
};
//作用如下
/*
如果没有它，你每加一个策略，都得手动写类似代码：

StrategyFactory::instance().registerStrategy("1", [] {
    return std::make_shared<AliyunStrategy>();
});
有了 StrategyRegister<T> 后，就可以简化成：

static StrategyRegister<AliyunStrategy> regAliyun("1");
*/
//StrategyRegister<NewModelStrategy> 类型的对象 regNew，然后调用了这个类型对应的构造函数，并把 name 传进去。
```

## AIToolRegistry

负责工具的注册，和调用(curl写回调函数)

```cpp
class AIToolRegistry {
public:
    using ToolFunc = std::function<json(const json&)>;//工具函数类型，接受json参数，返回json结果

    AIToolRegistry();//构造函数，注册内置工具

    void registerTool(const std::string& name, ToolFunc func);//注册工具接口
    json invoke(const std::string& name, const json& args) const//调用工具接口
    bool hasTool(const std::string& name) const;//检查工具是否存在接口

private:
    std::unordered_map<std::string, ToolFunc> tools_;

    
    static size_t WriteCallback(void* contents, size_t size, size_t nmemb, std::string* output);//curl回调函数，写入响应数据
    static json getWeather(const json& args);//内置工具：获取天气
    static json getTime(const json& args);//内置工具：获取时间
};

```
## AISessionIdGenerator

获取当前时间戳+一个随机数获得一个随机的sessionid

## aihelper

