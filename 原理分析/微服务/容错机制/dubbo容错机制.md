### dubbo容错机制
容错机制是发生在客户端（消费端）  
dubbo中常用的容错方式有：  
**1、Failover Cluster（失败自动切换）**  
FailoverClusterInvoker在调用失败时，会自动切换invoker进行重试，直到调用成功或达到重试次数。这是dubbo的默认容错策略。
**2、Failback Cluster(失败自动恢复)** 
FailbackClusterInvoker在调用失败时，会返回一个空结果，然后会通过定时任务对失败调用进行重发。
**3、FailFast Cluster(快速失败)**  
FailFastClusterInvoker只会进行一次调用，失败后抛出异常。
**4、FailSafe Cluster(失败安全)**  
FailsafeClusterInvoker 是一种失败安全的 Cluster Invoker。所谓的失败安全是指，当调用过程中出现异常时，FailsafeClusterInvoker 仅会打印异常，而不会抛出异常。返回一个空结果。
**5、Forking Cluster(并行调用多个服务提供者)**   
ForkingClusterInvoker 会在运行时通过线程池创建多个线程，并发调用多个服务提供者。只要有一个服务提供者成功返回了结果，doInvoke 方法就会立即结束运行。ForkingClusterInvoker 的应用场景是在一些对实时性要求比较高读操作（注意是读操作，并行写操作可能不安全）下使用，但这将会耗费更多的资源
**6、Broadcast Cluster(广播给每个服务调用者)**
BroadcastClusterInvoker 会逐个调用每个服务提供者，如果其中一台报错，在循环调用结束后，BroadcastClusterInvoker 会抛出异常。该类通常用于通知所有提供者更新缓存或日志等本地资源信息。