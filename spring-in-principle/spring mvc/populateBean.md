populateBean


---

1. HandlerMapper解析过程
2. <mvc:default-servlet-handler/>	-> DefaultServletHttpRequestHandler.class
									beanName = org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler#0
			default-servlet-handler urlMap -->/**  
			生成的handlerMapping的BeanDefination class = SimpleUrlHandlerMapping.class
			urlMap 作为BeanDefination的pv
			beanName是默认生成规则
									
3.<mvc:annotation-driven></mvc:annotation-driven>
	会注册一个 ContentNegotiationManagerFactoryBean.class BeanDefination
	注册一个 RequestMappingHandlerMapping.class 的BeanDefination
	注册一个 ConfigurableWebBindingInitializer.class BeanDefination
	注册一个 RequestMappingHandlerAdapter.class BeanDefination
	CompositeUriComponentsContributorFactoryBean.class BeanDefination
	ConversionServiceExposingInterceptor.class
	MappedInterceptor.class
	ExceptionHandlerExceptionResolver.class
	ResponseStatusExceptionResolver.class
	DefaultHandlerExceptionResolver.class
	
	<mvc:interceptors>
		<bean class="pro.jing.springmvc.interceptor.AccessInterceptor"></bean>
	</mvc:interceptors>