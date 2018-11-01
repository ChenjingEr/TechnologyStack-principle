在mybatis 执行一个sql的整体流程

1. get SqlSessionFactory
2. get SqlSession
3. get Mapper
4. 'mapper' 方法执行
5. 取得结果


图


参数映射
    参数映射配置
    参数映射解析
    参数类型解析
sql解析
    sql获取
    sql解析
    动态sql
sql执行
    SimpleExecutor
    BatchExeccutor
    ReuseExecutro
结果映射
    结果映射配置
    结果类型转换
    结果数据拷贝

基础支持层： 链接管理，事物管理，配置加载，缓存处理， 配置框架







第1章　MyBatis快速入门
1.1　ORM简介
1.2　常见持久化框架
1.3　MyBatis示例
1.4　MyBatis整体架构
1.4.1　基础支持层
1.4.2　核心处理层
1.4.3　接口层
1.5　本章小结





第2章　基础支持层
2.1　解析器模块
2.1.1　XPath简介
2.1.2　XPathParser

2.2　反射工具箱
2.2.1　Reflector＆ReflectorFactory
2.2.2　TypeParameterResolver
2.2.3　ObjectFactory
2.2.4　Property工具集
2.2.5　MetaClass
2.2.6　ObjectWrapper
2.2.7　MetaObject


2.3　类型转换
2.3.1　TypeHandler
2.3.2　TypeHandlerRegistry
2.3.3　TypeAliasRegistry


2.4　日志模块
2.4.1　适配器模式
2.4.2　日志适配器
2.4.3　代理模式与JDK动态代理
2.4.4　JDBC调试


2.5　资源加载
2.5.1　类加载器简介
2.5.2　ClassLoaderWrapper
2.5.3　ResolverUtil
2.5.4　单例模式
2.5.5　VFS


2.6　DataSource
2.6.1　工厂方法模式
2.6.2　DataSourceFactory
2.6.3　UnpooledDataSource
2.6.4　PooledDataSource


2.7　Transaction


2.8　binding模块
2.8.1　MapperRegistry＆MapperProxyFactory
2.8.2　MapperProxy
2.8.3　MapperMethod


2.9　缓存模块
2.9.1　装饰器模式
2.9.2　Cache接口及其实现
2.9.3　CacheKey
2.10　本章小结


第3章　核心处理层
3.1　MyBatis初始化
3.1.1　建造者模式
3.1.2　BaseBuilder
3.1.3　XMLConfigBuilder
3.1.4　XMLMapperBuilder
3.1.5　XMLStatementBuilder
3.1.6　绑定Mapper接口
3.1.7　处理incomplete*集合
3.2　SqlNode＆SqlSource
3.2.1　组合模式
3.2.2　OGNL表达式简介
3.2.3　DynamicContext
3.2.4　SqlNode
3.2.5　SqlSourceBuilder
3.2.6　DynamicSqlSource
3.2.7　RawSqlSource
3.3　ResultSetHandler
3.3.1　handleResultSets()方法
3.3.2　ResultSetWrapper
3.3.3　简单映射
3.3.4　嵌套映射
3.3.5　嵌套查询＆延迟加载
3.3.6　多结果集处理
3.3.7　游标
3.3.8　输出类型的参数
3.4　KeyGenerator
3.4.1　Jdbc3KeyGenerator
3.4.2　SelectkeyGenerator
3.5　StatementHandler
3.5.1　RoutingStatementHandler
3.5.2　BaseStatementHandler
3.5.3　ParameterHandler
3.5.4　SimpleStatementHandler
3.5.5　PreparedStatementHandler
3.6　Executor
3.6.1　模板方法模式
3.6.2　BaseExecutor
3.6.3　SimpleExecutor
3.6.4　ReuseExecutor
3.6.5　BatchExecutor
3.6.6　CachingExecutor
3.7　接口层
3.7.1　策略模式
3.7.2　SqlSession
3.7.3　DefaultSqlSessionFactory
3.7.4　SqlSessionManager
3.8　本章小结
第4章　高级主题
4.1　插件模块
4.1.1　责任链模式
4.1.2　Interceptor
4.1.3　应用场景分析
4.2　MyBatis与Spring集成
4.2.1　Spring基本概念
4.2.2　Spring MVC介绍
4.2.3　集成环境搭建
4.2.4 Mybatis-Spring剖析
4.3　拾遗
4.3.1　应用＜sql＞节点
4.3.2　OgnlUtils工具类
4.3.3　SQL语句生成器
4.3.4　动态SQL脚本插件
4.3.5　MyBatis-Generator逆向工程
4.4　本章小结




mapper解析：

cache – Configuration of the cache for a given namespace.
cache-ref – Reference to a cache configuration from another namespace.

resultMap – The most complicated and powerful element that describes how to load your objects from the database result sets.

sql – A reusable chunk of SQL that can be referenced by other statements.
insert – A mapped INSERT statement.
update – A mapped UPDATE statement.
delete – A mapped DELETE statement.
select – A mapped SELECT statement.


sql解析

SqlSource sqlSource = langDriver.createSqlSource(configuration, context, parameterTypeClass);

public SqlSource parseScriptNode() {
    List<SqlNode> contents = parseDynamicTags(context);
    MixedSqlNode rootSqlNode = new MixedSqlNode(contents);
    SqlSource sqlSource = null;
    if (isDynamic) {
      sqlSource = new DynamicSqlSource(configuration, rootSqlNode);
    } else {
      sqlSource = new RawSqlSource(configuration, rootSqlNode, parameterType);
    }
    return sqlSource;
  }


getMapper:

执行流程：

1. parse configuration
2. get mapper (dynamic proxy，MapperProxy)
3. invoke(MapperProxy)
    -> final MapperMethod mapperMethod = cachedMapperMethod(method); //cache MapperMethod
       return mapperMethod.execute(sqlSession, args);

       mapperMethod.execute

       1.switch command.getType -> case insert, update, delete,, select
       2. method.convertArgsToSqlCommandParam(args)
            -> paramNameResolver 1. 没有参数，直接返回Null
                                 2. 1个参数，返回names.firstKey
                                 3. 多个，
                                 

