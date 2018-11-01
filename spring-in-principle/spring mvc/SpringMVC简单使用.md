#### SpringMVC的简单使用

1. 配置web.xml的ServletDispatcher，作为分发器。
```
	<servlet>
		<servlet-name>springmvc</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		
		<load-on-startup>1</load-on-startup>
	</servlet>

	<!-- Map all requests to the DispatcherServlet for handling -->
	<servlet-mapping>
		<servlet-name>springmvc</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>

```
2. 用@Contorller注解配置控制器，@RequestMapper配置请求路径

3. 配置视图解析器InternalResourceViewResource
```
	<!- 包扫描路径 ->
	<context:component-scan base-package="pro.jing.web"></context:component-scan>
	
	<!-- 视图解析器 -->
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/view/"></property>
		<property name="suffix" value=".jsp"></property>
	</bean>
```

#### RequestMapping
1. RequestMapping注解用于设置请求路径,可以用于类，方法