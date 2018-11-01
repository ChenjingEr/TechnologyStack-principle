1. 服务暴露
ServiceConfig -> 
    1. afterPropertiesSet 设置 provider,applicatioin, module, registries,monitor,protocol, path
        -> 发布设置(是否延迟发布) dubbo:provider delay="-1"  没有设置delay或delay=-1表示延迟发布
    2. 暴露方法 -> export 
        public synchronized void export() {
        if (provider != null) {
            if (export == null) {
                export = provider.getExport();
            }
            if (delay == null) {
                delay = provider.getDelay();
            }
        }
        if (export != null && !export) {   // @1 已经发布就不在发布
            return;
        }

        if (delay != null && delay > 0) {    // @2 延迟发布方法
            delayExportExecutor.schedule(new Runnable() {
                @Override
                public void run() {
                    doExport();
                }
            }, delay, TimeUnit.MILLISECONDS);
        } else {
            doExport();    //@3
        }
    }

    -> doExport
    -> doExportUrls

      private void doExportUrls() {
        List<URL> registryURLs = loadRegistries(true); -> true表示服务的提供者,false表示服务的消费者
        for (ProtocolConfig protocolConfig : protocols) { ->设置的protocol发布
            doExportUrlsFor1Protocol(protocolConfig, registryURLs);
        }
    }

    url -> registry://127.0.0.1:2181/com.alibaba.dubbo.registry.RegistryService?application=demo-provider&dubbo=2.0.0&pid=7072&qos.port=22222&registry=zookeeper&timestamp=1527308268041 

    -> doExportUrlsFor1Protocol
        protocolConfig.getName -> 为空则使用默认的协议 dubbo
        用Map存储该协议的所有配置参数，包括协议名称、dubbo版本、当前系统时间戳、进程ID、application配置、module配置、默认服务提供者参数(ProviderConfig)、协议配置、服务提供Dubbo:service的属性。 

    -> injvm 协议，不向注册中心发布服务 map(notify,false)

    -> 根据scope配置发布协议，scope没有配置，远程和本地都发布，
        scope = remote -> 先在本地发布(injvm),再remote
        scope = 不是local,远程发布
        scope = remote




2. 服务引用
3. 服务消费