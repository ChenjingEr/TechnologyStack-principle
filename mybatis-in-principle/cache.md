## mybatis 缓存机制

1. settings 

mybatis-config.xml
```
<settings>
  <!-- 全局设置缓存，default=true -->
  <setting name="cacheEnabled" value="true"/>   
</settings>
```

mapper-config.xml
```
<select
  id="selectPerson"
  useCache="true"
>
```
```
<insert
  id="insertAuthor"
  flushCache="true"
>

<update
  id="updateAuthor"
  flushCache="true"
>

<delete
  id="deleteAuthor"
  flushCache="true"
>
```

