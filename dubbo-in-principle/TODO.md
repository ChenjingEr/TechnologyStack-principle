1. 服务发布
ServiceConfig -> Proxy -> Exporter -> Remoting -> ServiceRegistry

1. ServiceConfig

<dubbo:application name="demo-provider" />   -> ApplicationConfig
<dubbo:registry address="zookeeper://192.168.1.46:2181"/> -> RegistryConfig
<dubbo:protocol name="dubbo" port="20880" />  -> ProtocolConfig
<dubbo:service interface="pro.jing.api.DemoService" ref="demoService" /> -> ServiceBean

[demo-provider, com.alibaba.dubbo.config.RegistryConfig, dubbo, demoService, pro.jing.api.DemoService]

2. Proxy

proxyFactory -> wapper

proxyFactory.getInvoker

3. Exporter

exportLocal(url)

protocol.export

4. Remoting

/dubbo/pro.jing.api.DemoService/providers/


dubbo%3A%2F%2F192.168.56.1%3A20880%2Fpro.jing.api.DemoService%3Fanyhost%3Dtrue%26application%3Ddemo-provider%26dubbo%3D2.6.2%26generic%3Dfalse%26interface%3Dpro.jing.api.DemoService%26methods%3DsayHello%26pid%3D8096%26side%3Dprovider%26timestamp%3D1535456284775

5. ServiceRegistry


2. 服务引用

