## mysql 安装

1. 下载
2. 解压
3. 配置 my.ini
```
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8 
[mysqld]
#设置3306端口
port = 3306 
# 设置mysql的安装目录
basedir=D:\mysql-8.0.11-winx64
# 设置mysql数据库的数据的存放目录
datadir=F:\mysql-data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB

```
4. cmd命令下 mysqld install
5. cmd命令下 mysqld --initialize / mysqld --initialize-insecure 不生成密码
6. cmd命令下 net mysql start 

问题记录：Can't connect to MySQL server on 'localhost' (10061)

cmd命令下：mysql mysqld --skip-grant-tables
