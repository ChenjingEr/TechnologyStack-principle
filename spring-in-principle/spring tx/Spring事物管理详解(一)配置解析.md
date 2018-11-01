## Spring事务管理(一)： 配置解析

事物管理配置:
```
<bean id="dataSource"
    class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="user" value="${jdbc.user}"></property>
    <property name="password" value="${jdbc.password}"></property>
    <property name="driverClass" value="${jdbc.driver}"></property>
    <property name="jdbcUrl" value="${jdbc.url}"></property>
    <property name="minPoolSize" value="3"></property>
    <property name="maxPoolSize" value="10"></property>
</bean>

<bean id="jdbcTemplate"
    class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"></property>
</bean>

<bean id="transactionManager"
    class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>

<tx:annotation-driven transaction-manager="transactionManager" />
```
<tx:annotation-driven transaction-manager="transactionManager" /> 其中没有配置transaction-manager，则默认取name=transactionManager的配置。tx解析器配置：
```
http\://www.springframework.org/schema/tx=org.springframework.transaction.config.TxNamespaceHandler

@Override
public void init() {
    registerBeanDefinitionParser("advice", new TxAdviceBeanDefinitionParser());
    registerBeanDefinitionParser("annotation-driven", new AnnotationDrivenBeanDefinitionParser());
    registerBeanDefinitionParser("jta-transaction-manager", new JtaTransactionManagerBeanDefinitionParser());
}
``` 

AnnotationDrivenBeanDefinitionParser解析:
```
public BeanDefinition parse(Element element, ParserContext parserContext) {
        registerTransactionalEventListenerFactory(parserContext);
        String mode = element.getAttribute("mode");
        if ("aspectj".equals(mode)) {
            // mode="aspectj"
            registerTransactionAspect(element, parserContext);
        }
        else {
            // mode="proxy"
            AopAutoProxyConfigurer.configureAutoProxyCreator(element, parserContext);
        }
        return null;
    }
```

AopAutoProxyConfigurer.configureAutoProxyCreator(element, parserContext); 

1. 注册代理生成器：
```
public static BeanDefinition registerAutoProxyCreatorIfNecessary(BeanDefinitionRegistry registry, Object source) {
    return registerOrEscalateApcAsRequired(InfrastructureAdvisorAutoProxyCreator.class, registry, source);
}
```
2. 注册与事物管理相关的几个bean
```
RootBeanDefinition sourceDef = new RootBeanDefinition(
                        "org.springframework.transaction.annotation.AnnotationTransactionAttributeSource");

RootBeanDefinition interceptorDef = new RootBeanDefinition(TransactionInterceptor.class);
    pvs     transactionManagerBeanName -> element.getAttribute(TRANSACTION_MANAGER_ATTRIBUTE)
            transactionAttributeSource ->  sourceDef

RootBeanDefinition advisorDef = new RootBeanDefinition(BeanFactoryTransactionAttributeSourceAdvisor.class);
    pvs     adviceBeanName -> interceptorDef
            transactionAttributeSource -> sourceDef
```
