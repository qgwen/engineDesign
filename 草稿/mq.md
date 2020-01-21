#### 思考
1、每个Broker可以存储多个Topic的消息，每个Topic的消息也可以分片存储于不同的Broker  
如何分片存储于不同的broker中？
- 根据topic下的所有QueueData的brokerName和WriteQueueNums来构造MessageQueue列表  
- 发送消息时，会从MessageQueue列表中选出一个Queue进行发送，这样就会导致同一topic下的消息发送到不同broker上

2、消息发送有同步发送、异步发送、顺序发送、单向发送  
重点了解异步发送和顺序发送的实现逻辑？（已看完）  
broker的处理逻辑：  
- broker收到消息后，broker会把消息写入到commitlog里面
- broker会启动一个后台线程(ReputMessageService)根据commitLog去构建ConsumeQueue  

3、消费消息分为拉取式消费和推动式消费  
推动式消费(push)的过程，主要分为两大部分：  
1、有个一个后台线程(pullService)一直从队列中获取pullRequest进行处理
2、reblanceService和拉取失败或者没有结果之后会把pullRquest放入队列中
参考文章：  
http://silence.work/2019/03/03/RocketMQ%20%E6%B6%88%E8%B4%B9%E6%B6%88%E6%81%AF%E8%BF%87%E7%A8%8B%E5%88%86%E6%9E%90/  
4、broker对于消费者组、消费进度偏移、主题和队列信息是如何维护的？  
5、生产者组的作用？  
- 提升消费性能 
- 支持容错（单个消费者down之后，可以由其他的消费者接管）

6、消费者组消费消息  
集群消费：同一个消费者组中的消费者平摊消费消息，如何实现的？  
广播消费：同一个消费者组中的每个消费者获取的是全量消息，如何实现？  
消费者批量消费消息如何commit consume offset？  
- 广播模式下，同一个消费者组下的每个消费者消费的是单个topic的全量数据，offset在本地维护(offset.json)
- 集群模式下，同一个消费组下的每个消费者消费的是单个topic的部分数据（整个消费者组消费全量），offset维护在broker(以group的维度维护)  
offset的提交与持久化：
- 每次消费消息成功后，如果消息不是可丢弃的，会更新offset，但是此时只更新内存态的offset
- 消费者启动的时候，会启动一个定时持久化offset（每10s）的任务，如果是广播模式，则持久化到本地文件，如果是集群模式，则发送持久化消息到broker中  

7、普通顺序消息和严格顺序消息  
普通顺序消息：同一个消费队列消息是有序的，整体是无序的  
严格顺序消息：消费者收到的消息整体是有序的，如何实现的？  
8、rocketMQ主从架构，消费者可以从slave broker上消费，kafka的架构是基于partition的，每个分区都有leader和同步副本，那么消息者消费分区的消息时，是否只能从leader节点上消息？？  
9、消费者既可以从master消费消息、也可以从slave消费消息，在向master拉取消息时，master服务器会根据一定的规则建议下次拉取是从master还是slave上拉取，这个规则是什么？？为什么要这样实现??  
10、发送消息时，选择队列的规则是什么？  
11、关于topic路由信息的上报和拉取（已看完）  
- 通过定时上报和拉取topic路由信息，默认都是30s（确定）    

但是上述方式会出现延时的问题，新创建的topic信息可能没有及时同步，导致发送消息和消费消息报错，如何解决该问题？
- 创建topic的时候会在broker中缓存当前topic的信息，同时把该信息发送到namesrv集群的所有节点中。（确定）
- producer发送消息时，根据toipc在本地缓存中获取路由信息，当不存在时，去namesrv中同步路由信息，consumer类似（确定）  

12、消费者组中的负载均衡是如何完成的？  
13、broker如何通过half消息和op消息进行事务消息的回查？  
是否需要间隔一段时间进行回查？ 需要，默认通过transactionMsgTimeout设置多长时间后进行回查，也可以通过设置用户属性CHECK_IMMUNITY_TIME_IN_SECONDS来改变这个限制，该参数优先于transactionMsgTimeout参数。    



