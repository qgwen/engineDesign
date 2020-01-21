### Eureka
SpringCloud Eureka是Springcloud基于Netflix Eureka封装的。  
#### Eureka中的角色：  
1. Eureka Server  
2. Eureka Client   

#### Eureka Client与Server之间的通讯的操作：  
1. Register(注册)  
2. Renew(续租)
3. Fetch Registry(获取注册表)  
4. Cancel(取消，告知Server剔除该实例) 

#### Eureka client端的serviceUrl的作用
eureka.client.serviceUrl是用来定位eureka server 

#### Eureka中的zone与region  
1. region：理解为地理分区  
2. zone：理解为同一个region下不同的机房  

在做架构设计的时候，让一个服务优先调用同一个机房内的服务，当一个机房的服务不可用的时候，再去调用其他机房的服务，达到减少延时的作用。  
服务注册：服务注册到同一zone的注册中心  
服务调用：服务调用同一zone的服务  
参考文章：  
https://juejin.im/post/5d68b73af265da03b12061be  
https://segmentfault.com/a/1190000014107639  
https://www.jianshu.com/p/2ca32773b3e5  

#### Registry（注册表）

#### 注册信息增量更新  

#### Eureka Server自我保护模式
**是什么**？  
是eureka server的一种自我保护措施，确保灾难性网络事件不会清除eureka server的注册表数据。  
例如：  
在Netfilix中，主要用于在一组客户端和Eureka Server之间存在网络分区的情况下的保护措施  
**什么情况下会进入自我保护模式？**  
eureka client结束它们注册的生命周期有两种情况：  
1. client执行明确的反注册(unregister)的操作  
2. client连续3次renew失败，等着eureka server剔除  

当server的注册表的数据多于15%（统计周期15分钟）处于第2种情况，将开启自我保护模式  
**如何从自我保护模式中退出？**  
1. client的心跳renew数又回到预期的阈值(默认为85%)上 eureka.renewalPercentThreshold=0.85   
2. 自我保护模式设置为禁用 eureka.enableSelfPreservation=false   

#### Eureka Server点对点通信  
**如何成为对等节点**
通过eureka.client.serviceUrl来配置
**节点和节点之间什么时候以及如何同步注册表数据？**  
1. 当执行某些操作（register、cancel、renew等）导致状态发生变化时，需要同步   
2. 把节点的状态变更广播给集群中其他节点  

分析文章:  
https://cloud.tencent.com/developer/article/1083131
http://www.iocoder.cn/Eureka/server-cluster/?vip  
http://www.iocoder.cn/Eureka/instance-registry-class-diagram/  

#### eureka client与server的配置参数
https://blog.csdn.net/u010647035/article/details/82870786



