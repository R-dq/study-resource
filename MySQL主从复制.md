### 一、创建两个数据库

数据库创建这里不做说明，请查看网络上其他资源。

### 二、主数据库配置

#### 1.修改my.cnf

进入其中一个数据库（选择的数据库就是主数据库）修改my.cnf文件，在其中添加内容：
```    
    #mysql 服务ID,保证整个集群环境中唯一
    server-id=1
    #mysql binlog 日志的存储路径和文件名
    log-bin=/var/lib/mysql/mysqlbin
    #设置logbin格式
    binlog_format=STATEMENT
    #是否只读，1代表只读，0代表读写
    read-only=0
```

修改完成以后记得重启刷新配置，重启以后查看my.cnf确保修改成功。

#### 2.创建用户及授权

进入MySQL内部执行命令：

①创建一个名为itcast的用户，密码为root。都可以自定义

`CREATE USER 'itcast'@'%' IDENTIFIED WITH mysql_native_password BY 'root';`

②为itcast用户授权，允许多方登录

`GRANT REPLICATION SLAVE ON *.* TO 'itcast'@'%';`

创建完成以后可以执行`SELECT user, host FROM mysql.user;`查看是否创建成功

#### 3.查看状态

`show master status;`

记录File和Position字段的值，用作从数据库连接主数据库配置。

ps：```File字段表示当前正在写入的二进制文件（binlog）的名称，Position字段表示二进制日志文件中当前写入位置的偏移量```

### 三、从数据库配置

#### 1.修改my.cnf

在其中添加内容：

```    
    #mysql 服务ID,保证整个集群环境中唯一
    server-id=2
    #mysql binlog 日志的存储路径和文件名
    log-bin=/var/lib/mysql/mysqlbin
```

修改完成以后记得重启刷新配置，重启以后查看my.cnf确保修改成功。

#### 2.开启主从复制

##### mysql是 8.0.23 之前的版本：

```
-- 停止从节点复制进程
STOP SLAVE;

-- 重置节点复制数据
RESET SLAVE;

-- 配置从节点连接主节点的信息
CHANGE MASTER TO
  MASTER_HOST='192.168.10.230',
  MASTER_PORT=32314,
  MASTER_USER='itcast',
  MASTER_PASSWORD='root',
  MASTER_LOG_FILE='mysqlbin.000002',
  MASTER_LOG_POS=2941;

-- 启动从节点复制进程
START SLAVE;

-- 查看节点复制情况。当查询结果的Slave_IO_Running和Slave_SQL_Running字段值都为yes时可以视为成功。为保证正确性查看 三、测试
SHOW SLAVE STATUS
```

##### mysql是 8.0.23 之后的版本：

执行如下命令：

``````
-- 停止从节点复制进程
STOP REPLICA;

-- 重置节点复制数据
RESET REPLICA;

-- 配置从节点连接主节点的信息
CHANGE REPLICATION SOURCE TO SOURCE_HOST='192.168.10.230', 
SOURCE_PORT=3306,
SOURCE_USER='itcast', 
SOURCE_PASSWORD='root', 
SOURCE_LOG_FILE='binlog.000001', 
SOURCE_LOG_POS=656;

-- 启动从节点复制进程
START REPLICA;

-- 查看节点复制情况。当查询结果的Replica_IO_Running和Replica_SQL_Running字段值都为yes时可以视为成功。为保证正确性查看 三、测试
SHOW REPLICA STATUS
``````

### 三、测试

主库建库建表，查看从库是否能更新



