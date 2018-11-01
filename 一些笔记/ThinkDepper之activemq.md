ThinkDepper 之 activemq

1. ConnectionFacory --create--> Connection
2. Connection --create--> Session
3. Session --create--> Destination
4. Session -- create --> Producer
5. producer -- send to --> destination
6. consumer -- receives from --> destination

#### ConnectionFacory --create--> Connection

```
ConnectionFactory cf = new ActiveMQConnectionFactory("tcp://localhost:61616");
con = cf.createConnection();
con.start();
```
1. ConnectionFactory cf = new ActiveMQConnectionFactory("tcp://localhost:61616");

参数是一个broker url,支持的shcema有：
```
<transportConnector name="auto" uri="auto://localhost:5671"/>
<transportConnector name="auto+ssl" uri="auto+ssl://localhost:5671"/>
<transportConnector name="auto+nio" uri="auto+nio://localhost:5671"/>
<transportConnector name="auto+nio+ssl" uri="auto+nio+ssl://localhost:5671"/>
```

new ActiveMQConenctionFactory做的工作是设置brokerUrL
this.brokerURL = data.toURI();

2. con = cf.createConnection();

->  Transport transport = createTransport();

    -> return TransportFactory.connect(connectBrokerUL); //transort通过TransportFactory生成
        -> TransportFactory tf = findTransportFactory(location);
            -> (TransportFactory)TRANSPORT_FACTORY_FINDER.newInstance(scheme) 通过FactoryFinder查找schema对应的TransportFactory(也就是最终是委托FactoryFinder查找的,创建的TransportFactory会放入TRANSPORT_FACTORYS缓存(ConcurrentMap))
              ->  return objectFactory.create(path+key);
                ->(private static ObjectFactory objectFactory = new StandaloneObjectFactory();)
                -> loadClass, clazz.newInstance() (通过path+schema 加载在 MATE_INFO下的信息，然后通过反射创建对应的Transport)

        -> tf.doConnect(location);
            -> WireFormat wf = createWireFormat(options); TODO
            -> Transport transport = createTransport(location, wf); tcp选项
                -> SocketFactory socketFactory = createSocketFactory();
                -> createTcpTransport(wf, socketFactory, location, localLocation);
            -> Transport rc = configure(transport, wf, options);
                -> new TcpTransport(wf, socketFactory, location, localLocation);
                   -> 底层通过java.net.SocketFactory提供的socketFactory.createSocket()创建socket
            -> IntrospectionSupport.extractProperties(options, "auto.");
            return rc;
       
->  ActiveMQConnection connection = createActiveMQConnection(transport, factoryStats);
    ->  ActiveMQConnection connection = new ActiveMQConnection(transport, getClientIdGenerator(),
                getConnectionIdGenerator(), stats);
-> 设置connection属性 ,connection setUserName,setPassword,大部分默认属性设置
-> transport.start(); ResponseCorrelator -> MutexTransport ->WireFormatNegotiator ->InactivityMonitor->ServiceSupport-> TcpTransport
    -> TcpTransport .doStart()
        -> socket.connect(remoteAddress, connectionTimeout) -> 30000
3. con.start();
    -> ensureConnectionInfoSent() //发送连接信息给broker
        -> synchronized(this.ensureConnectionInfoSentMutex) {} 同步，只能one by one.
            -> 设置clientId
            -> syncSendPacket(info.copy(), getConnectResponseTimeout()); (ConnectionInfo {commandId = 0, responseRequired = false, connectionId = null, clientId = null, clientIp = null, userName = null, password = *****, brokerPath = null, brokerMasterConnector = false, manageable = false, clientMaster = true, faultTolerant = false, failoverReconnect = false})
                -> ResponseCorrelator.request()
                    -> FutureResponse response = asyncRequest(command, null); ( ConnectionInfo类型)
                        -> next.oneway(command); 
                        MutexTransport(lock住， one by one 发送)
                        AbstractInactivityMonitor(send发送需要sendLock.readLock().lock(),空闲时是写锁)
                        最后的最后才是TcpTransaction
                    -> return response.getResult();

```
1.create ConnectionFactory
2.create Connection -> TCP连接建立
3.connection start -> 发送ConnectionInfo给Broker，有ClientID, ConnectionId等信息
通过socket通信， Transport封装，在Transport发送之前，通过一系列的Filter，中间会有各种synchronized,以及readLock


#### 2. Connection --create--> Session

```
session = con.createSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);
    -> 通过con创建1个ActiveMQSession，
```

#### 3. Session --create--> Destination
```
des = session.createQueue("queue-test");
    -> ActiveMQQueue(queueName)
des = session.createTopic("queue-test");
    -> ActiveMQTopic(topicName)
```

#### 4. Session -- create --> Producer
```
 MessageProducer producer = session.createProducer(des);
```

#### 5. producer -- send to --> destination
```
producer.send(msg);
session.commit();
```







