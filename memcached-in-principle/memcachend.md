## Memcachend

##### 什么是Memcached ?
分布式内存对象的缓存系统。数据存在内存中，以kev-value形式存在。
Memcached 基于文本协议<br>

###### memcached 安装 & 启动 & 关闭
```
    ./memcached -m 32m -p 11211 -d -u root -P /var/run/memcached.pid -c 256

    -p 端口号 默认11211
    -m 最大内存大小 默认64MB
    -vv very brebose模式启动，将调试信息和错误输出到控制台
    -d 作为守护线程在后台运行
    -c 最大的并发连接数，默认是1024
    -P 设置pid文件
    -l 监听的服务器IP地址,如果有多个地址
    -u 运行用户

    kill memcached.pid
```

###### 基本使用 

* 添加
```
   add key flags expire bytes[noreply] / set key flags expire bytes[noreply]
   value

   flags 可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息
   exptime:在缓存中保存键值对的时间长度（以秒为单位）
   bytes:在缓存中存储的字节数 
   noreply（可选） 该参数告知服务器不需要返回数据
```
* 替换
```
    replace key flags exptime bytes[noreply]
    value
```
* 追加
```
    append key flags exptime bytes [noreply]
    value
```
* 删除
```
    delete key [noreply]
```
* 读取
```
    get key
    get key1 key2 key3
    gets key （CAS）
```
* incr/decr 
```
    incr key increment_value
    decr  key increment_value
```

