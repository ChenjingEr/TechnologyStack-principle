## reids 基本使用(一)：key

##### 关于key命令

* 获取符合规则的key列表

```
    keys pattern 

    通配符说明：
        ?  匹配1个字符
        *  匹配0-任意个字符
        [] 匹配括号里的任一个字符
        \  转义
```

* 判断是否存在

```
    exists key_name  

    返回(integer)：1-存在，0-不存在
```

* 删除key
 
```
    del key //删除1个
    del key1 key2 key3...
```

* key对应的value的类型

```
    type key

    redis支持的数据类型: string,hash,list,set,zset
```
