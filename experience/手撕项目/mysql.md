# 单个数据库连接

主要需要实现：

1.构造、析构、禁止拷贝构造、拷贝赋值。  
2.是否可以正常连接的判断函数  
3.查询数据库  
4.写入数据库(与查询逻辑相同)  



```cpp

class Dbconnection {
public:
    //构造、析构、禁止拷贝构造、拷贝赋值。
    Dbconnection(const std::string& host,
                 const std::string& user,
                 const std::string& password,
                 const std::string& database);

    ~Dbconnection();

    Dbconnection(const Dbconnection&) = delete;
    Dbconnection& operator=(const Dbconnection&) = delete;

    bool ping();
    bool reconnect();
    bool is_valid();

    // 查询
    template<typename...Args>
    std::unique_ptr<sql::ResultSet> executeQuery(const std::string& sql, Args&&... args) {
        std::lock_guard<std::mutex> lock(mutex_);//先上锁
        try {
            std::unique_ptr<sql::PreparedStatement> stmt(conn_->prepareStatement(sql));//准备sql语句
            bindParams(stmt.get(), 1, std::forward<Args>(args)...);//用args中的参数填充sql语句中的？？？
            return std::unique_ptr<sql::ResultSet>(stmt->executeQuery());//读取sql中的数据
        }
        catch (const std::exception& e) {
            std::cerr << "[Query Error] " << e.what() << '\n';
            return nullptr;
        }
    }

    // 更新（insert/update/delete）
    template<typename...Args>
    int executeUpdate(const std::string& sql, Args&&... args) {
        std::lock_guard<std::mutex> lock(mutex_);
        try {
            std::unique_ptr<sql::PreparedStatement> stmt(conn_->prepareStatement(sql));
            bindParams(stmt.get(), 1, std::forward<Args>(args)...);
            return stmt->executeUpdate();//写入sql
        }
        catch (const std::exception& e) {
            std::cerr << "[Update Error] " << e.what() << '\n';
            return -1;
        }
    }

private:
    // 参数绑定
    template<typename T, typename... Args>
    void bindParams(sql::PreparedStatement* stmt, int idx, T&& val, Args&&... args) {
        if constexpr (std::is_same_v<T, int>) {
            stmt->setInt(idx, val);
        }
        else if constexpr (std::is_same_v<T, std::string> || std::is_same_v<T, const char*>) {
            stmt->setString(idx, val);
        }
        else if constexpr (std::is_same_v<T, double>) {
            stmt->setDouble(idx, val);
        }

        if constexpr (sizeof...(args) > 0) {
            bindParams(stmt, idx + 1, std::forward<Args>(args)...);
        }
    }

    std::shared_ptr<sql::Connection> conn_;
    std::string host_, user_, password_, database_;
    std::mutex mutex_;
};

```