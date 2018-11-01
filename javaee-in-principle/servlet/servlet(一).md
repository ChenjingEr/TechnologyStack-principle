#### 什么是Servlet？
Servlet是基于Java技术的Web组件，运行在服务端，用于动态生成请求内容。

#### servlet的开发步骤(HttpServlet)

1. 实现一个Java类，继承HttpServlet
2. 实现service的方法(doGet,doPost...)
3. 实现init(), destory()方法，如果有必要的话
4. web.xml中配置"<servlet>" "<servlet-mapping>"

#### Servlet的生命周期
servlet的生命周期由servlet容器(调用servlet的程序)管理
1. 加载和实例化
    servlet容器启动或servlet第一次接收请求加载实例化
    容器利用反射实例化servlet,且使用默认构造函数
2. 初始化
    实例化之后，调用init()方法
    init()只是调用1次
3. 处理请求
    调用service()方法
4. 服务终止
    调用destory()

#### Servlet上下文(ServletContext)
ServletContext是Web应用程序的上下文表示。可以被所有Servlet访问，Servlet的init中注入的ServletConfig可以getServeltContext()获得ServletContext

