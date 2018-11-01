## redis基本使用(四):列表
>列表存储的值必须是字符串，最大存储的字段数量为2^32-1个，redis内部列表的实现是双向链表。list类型即可以作为双向列表使用个，也可以作为数组使用。列表保持的顺序是插入时的顺序。

* 向两端添加/获取/删除
```
    lpush key value...          队头添加
    rpush key value...          队尾添加
    lpop key                    队头获取并删除
    rpop key                    队尾获取并删除
    lrange key start stop       获取从start - stop 范围内的元素 start/stop正数从左边开始获取，负数从右边开始获取
    lrem key count value        删除前count个值=value的值。count>0,从左边开始删除；count<0，从右边开始删除;count=0,删除全部
    lindex key index            获取index上的元素
    lset key index value        设置index上的值=value
```

* 元素个数
```
    llen key
```

* 截取操作
```
    ltrim key start end          保留列表指定片段
```

* 插入
```
    linsert key after/before pivot value     在pivot后/前插入value
```