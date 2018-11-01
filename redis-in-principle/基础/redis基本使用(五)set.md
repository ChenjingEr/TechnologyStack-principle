## redis基本使用(五)：集合
>集合中的元素无序，不存在相同元素

* 添加/删除/获取
```
    sadd key value...      添加，重复添加不会成功
    srem key value...      删除
    smembers key           获得所有值
```

* 是否存在
```
    sismember key value    value是否存在在key中
```

* 集合运算
```
    sdiff key...           差集 k1-k2
    sinter key...          交集 
    sunion key...          并集
```

* 元素个数
```
    scard key 
```