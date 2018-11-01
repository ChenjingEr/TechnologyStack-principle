## mongodb使用

#### 基本概念：
* 文档：是一组键值对的集合。其中键/值不区分大小，不可以有重复的键，值可以是不同的数据类型。
* 集合：一组文档的集合成为集合。是动态模式，也就是说集合中的文档可以是各式各样的。
* 数据库：一组结合组成一个数据库(集合存在数据库中)

###### 安装
解压mongodb压缩包，启动命令在bin中

###### 启动关闭
启动服务器：<br>
```
./mongod //默认启动
./mongod --dppath=path --logpath=filt_path -f conf_file_path   //启动服务器，配置dppath，logpath 
./mongod --shutdown  //关闭服务器 
```
默认启动方式，用的dbpath=/data/db,port=27017

###### mongo shell 操作(info,curd)

* 数据库：mongodb中多个集合组成数据库。一个mongodb实例有0-n个实例
```
    use demo          隐式创建数据库，没有insert不显示(第一次insert才会生成文件)
    db;               显示当前正在使用的db name
    show dbs;         显示db列表
    db.dropDatabase() 删除db
```

* 集合：mongodb集合中存放0-n个文档
```
    show collections;                       显示当前数据库中的集合
    db.createCollection(name,optins);       创建集合
    db.COLLECTION_NAME.drop();              删除集合

```

* 文档：mongodb以文档为存储单位，文档以BSON格式存储
```
    插入：
        db.collection_name.insert(name:"",age:"");              插入 name:"",age:"" 文档。没有_id,mongodb自动生成
        db.collection_name.insert([{},{}]);                     批量插入文档，没有_id,mongodb自动生成
        db.collcetion_name.insert(name:["","",""]);             插入数组
        db.collection_name.insert({name:"",addr:{num:""}});     插入嵌套文档
    查询 :
        db.collections_name.find();                             查询所有的文档
        db.collections_name.find({},{"age"：1});                查询所有的文档且只返回age的键
        db.collections_name.find({name:"",field:value});        查询 name="" and field=value...的文档
        db.collections_name.findOne();                          查询1个文档
        db.collections_name.find({age:{"$gte":18,"$lte":30}});  查询age>=18 and age <=30的文档，$lt < , $lte <= , $gt > , $gte >= 。
        db.collections_name.find({name:{"$ne":""}});            查询name != "" 的文档
        db.collections_name.find({"age":{"$in":[10,20,30]});    查询age=10 || age = 20 || age = 30 的值
        db.collections_name.find({"$or":[{},{}]);               or 条件
    更新：
        db.collection_name.update({});                          可以设置修改器
    删除：
        db.collection_name.remove()                             删除元素，不会删除集合
        db.collection_name.remove({"age":{"$in":[10,20,30]})    有条件删除元素
        db.collection.drop()                                    不能指定删除条件，集合也会删除                      

```

###### 数据类型
```
    String  : 存储的有效的UTF-8
    Integer : 32/64位
    Boolean : true/false
    Double  : 64位
    Arrays  : 数组
    Null    : 空

```
