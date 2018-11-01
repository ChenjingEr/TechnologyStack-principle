## mongodb　聚合

mongodb的聚合框架可以对一组文档进行变换和组合输出。mongodb 中聚合分为 Pipline 和 MapReduce。Pipeline查询速度快于MapReduce，MapReduce更加强大，灵活。


##### Pipline

**$match** : 筛选符合条件的文档
```
	筛选年龄 <=20 的user :
	db.users.aggregate({$match:{age:{"$lte":20}}});
```

**$project**:投射，选择想要的字段 || 字段重命名。
```
	只显示 username字段，_id不显示(默认_id都显示)：
	db.users.aggregate({$project:{username:1,_id:0}});
	username 重命名：
	db.users.aggregate({$project:{"name":"$username"}});

	$project + 管道表达式：

		1. 数学表达式
			age+1 ($add)
				db.users.aggregate({$project:{age:{"$add":["$age",1]}}});
			age-1 ($subtract)
				db.users.aggregate({$project:{age:{"$subtract":["$age",1]}}});
			age*2 ($multiply)
				db.users.aggregate({$project:{age:{"$multiply":["$age",2]}}});
			age/2 ($divide)
				db.users.aggregate({$project:{age:{"$divide":["$age",2]}}});
			age%/2 ($mod)
				db.users.aggregate({$project:{age:{"$mod":["$age",2]}}});
		2. 字符串表达式
			子字符串($substr)
				db.users.aggregate({$project:{username:{"$substr":["$username",0,3]}}});
			连接 ($concat)
				db.users.aggregate({$project:{username:{"$concat":["$username","uu"]}});
			转大写($toUpper)
				db.users.aggregate({$project:{username:{"$toUpper":"$username"}});
			转小写($toLower)
				db.users.aggregate({$project:{username:{"$toLower":"$username"}});
		3. 日期表达式
			$year
				db.users.aggregate({$project:{created:{"$year":"$created"}}});
			$month
			$week
			$dayOfMonth
			$dayOfWeek
			$dayOfYear
			$hour
			$minute
			$second
		4. 逻辑表达式
			$cmp[]
			$strcasecmp[]
			$eq/$ne/$gt/$gte/$lt/$lte:[]
			$and
			$not
			$and
			$cond:[booleanExpre,ture,false]
			$isNull

```

**$group**：分组
```
	db.users.aggregate({"$group":{"_id":"$age"}});

```

**$unwind**：拆分
**$sort:**：排序



