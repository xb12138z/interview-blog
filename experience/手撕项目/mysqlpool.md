# 数据库连接池

主要对外接口：  
1.提供对外的get_instance  
2.init函数构建数据库连接池，创建单个数据库连接接口
3.构造、析构、禁止拷贝构造赋值  
4.获取连接函数



```cpp
class DbConnectionPool {
public:
    static DbConnectionPool& getInstance() {
        static DbConnectionPool instance;
        return instance;
    }

    void init(std::string host, std::string user, std::string password, std::string database, size_t poolsize) {
        std::lock_guard<std::mutex> lock(mutex_);
        host_ = host;
        user_ = user;
        password_ = password;
        database_ = database;
        poolsize_ = poolsize;

        for (size_t i = 0; i < poolsize; ++i) {
            connections_pool.push(createConnection());
        }
    }

    std::shared_ptr<Dbconnection> getConnection() {
        std::unique_lock<std::mutex> lock(mutex_);
        cond_.wait(lock, [this]() { return !connections_pool.empty(); });

        auto conn = connections_pool.front();
        connections_pool.pop();

        if (!conn->ping()) {
            std::cout << "connection invalid, reconnecting\n";
            conn->reconnect();
        }

        return std::shared_ptr<Dbconnection>(conn.get(), [this, conn](Dbconnection*) {
            std::lock_guard<std::mutex> lock(mutex_);
            connections_pool.push(conn);
            cond_.notify_one();
        });
    }

private:
    DbConnectionPool() {
        checkThread_ = std::thread(&DbConnectionPool::checkConnection, this);
        checkThread_.detach();
    }

    ~DbConnectionPool() = default;

    DbConnectionPool(const DbConnectionPool&) = delete;
    DbConnectionPool& operator=(const DbConnectionPool&) = delete;

    std::shared_ptr<Dbconnection> createConnection() {
        return std::make_shared<Dbconnection>(host_, user_, password_, database_);
    }

    void checkConnection() {
        // 后台定时检测连接
        while (true) {
            std::this_thread::sleep_for(std::chrono::seconds(5));
            // 检测逻辑（略）
        }
    }

private:
    std::string host_, user_, password_, database_;
    size_t poolsize_;
    std::mutex mutex_;
    std::condition_variable cond_;
    std::thread checkThread_;
    std::queue<std::shared_ptr<Dbconnection>> connections_pool;
};
```