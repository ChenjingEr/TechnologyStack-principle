
#### namesrv 注册中心 ->

1. 保存集群中各个组件的信息
2. 各个角色定期向NameServer上报状态，超时认为不可用，会移除
3. nameser 可以部署多个，各个namesvr本身无状态，各个组件自动上报，放到Broker，Topic中

RouteInfoManager保存集群中的状态：
    -> private final ReadWriteLock lock = new ReentrantReadWriteLock();
    -> private final HashMap<String/* topic /, List<QueueData>> topicQueueTable;
        topic -> master broker的个数，QueueData broker的名称，读写queue的数量，同步标识等
    -> private final HashMap<String/* brokerName /, BrokerData> brokerAddrTable;
        brokerName -> master + slave的地址
    -> private final HashMap<String/* clusterName /, Set<String/* brokerName />> clusterAddrTable;
        clusterName -> borker name 集合
    -> private final HashMap<String/* brokerAddr /, BrokerLiveInfo> brokerLiveTable;
        brokerAddr -> broker的实时信息
    -> private final HashMap<String/* brokerAddr /, List<String>/* Filter Server /> filterServerTable;
        过滤服务器

1. namesrv 状态维护

BrokerHousekeepingService -> ChannelEventListener
    -> onChannelClose()
        -> this.namesrvController.getRouteInfoManager().onChannelDestroy(remoteAddr, channel);
    -> onChannelException()
    -> onChannelIdle()

定期检查时间戳逻辑
```
 this.scheduledExecutorService.scheduleAtFixedRate(new Runnable() {

    @Override
    public void run() {
        NamesrvController.this.routeInfoManager.scanNotActiveBroker();
    }
}, 5, 10, TimeUnit.SECONDS);

延迟5秒，每隔10秒执行一次

 public void scanNotActiveBroker() {
    Iterator<Entry<String, BrokerLiveInfo>> it = this.brokerLiveTable.entrySet().iterator();
    while (it.hasNext()) {
        Entry<String, BrokerLiveInfo> next = it.next();
        long last = next.getValue().getLastUpdateTimestamp();
        if ((last + BROKER_CHANNEL_EXPIRED_TIME) < System.currentTimeMillis()) {
            RemotingUtil.closeChannel(next.getValue().getChannel());
            it.remove();
            log.warn("The broker channel expired, {} {}ms", next.getKey(), BROKER_CHANNEL_EXPIRED_TIME);
            this.onChannelDestroy(next.getKey(), next.getValue().getChannel());
        }
    }
}

超过2分钟移除

```

2. 各个组件的交互


1. Namesrv 启动
    -> NamesrvStartup.main
        -> NamesrvController controller = createNamesrvController(args);
            -> final NamesrvConfig namesrvConfig = new NamesrvConfig();
            -> final NettyServerConfig nettyServerConfig = new NettyServerConfig();
            -> nettyServerConfig.setListenPort(9876);



        -> start(controller);

#### remoting 模块 ->

#### producer 

同步发送
异步发送
延迟发送
发送事务消息

1. 启动一个producer
```
//producerGroup
DefaultMQProducer producer = new DefaultMQProducer("rmq-group");
    -> set producerGroup
    -> new DefaultMQProducerImpl(this, rpcHook)

producer.setNamesrvAddr("127.0.0.1:9876");
producer.setInstanceName("rmq-instance");
producer.start();
    -> serviceState CREATE_JUST -> START_FAILED
    -> create MQClientInstance mQClientFactory
    -> ConcurrentHashMap<String/* topic */, TopicPublishInfo> topicPublishInfoTable =
            new ConcurrentHashMap<String, TopicPublishInfo>();
    -> mQClientFactory.start()
        -> this.mQClientAPIImpl.start();
            -> this.remotingClient.start();
        -> this.startScheduledTask();
        -> this.pullMessageService.start();
        -> this.rebalanceService.start();
        -> this.defaultMQProducer.getDefaultMQProducerImpl().start(false);
    -> this.serviceState = ServiceState.RUNNING;
```

2. send to 

发送模式 CommunicationMode-
>SYNC, 同步 (默认)
ASYNC, 异步
ONEWAY, one way

构建发送 RemotingCommand
SendMessageRequestHeader -> producerGroup, topic, defaultTopic, defaultTopicQueueNums,
    queueId, sysFlag, bornTimestamp, flag, properties,

 

3. broker 接收处理

NettryRemotingServer.processMessageReceived
    -> cmd.getType()
    -> processRequestCommand()

持久化消息->

MessageExtBrokerInner 
    -> PutMessageResult putMessageResult = this.brokerController.getMessageStore().putMessage(msgInner);
    ->  PutMessageResult result = this.commitLog.putMessage(msg); //添加消息到CommitLog






