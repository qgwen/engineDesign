





**注册中心为zk的情况下**  
**registryUrl(注册url)** 
registry://127.0.0.1:2181/org.apache.dubbo.registry.RegistryService?application=dubbo-demo-api-provider&dubbo=2.0.2&pid=60654&registry=zookeeper&timestamp=1579663240895

**url:**  
dubbo://192.168.3.54:20880/org.apache.dubbo.demo.DemoService?anyhost=true&application=dubbo-demo-api-provider&bind.ip=192.168.3.54&bind.port=20880&deprecated=false&dubbo=2.0.2&dynamic=true&generic=false&interface=org.apache.dubbo.demo.DemoService&methods=sayHello,sayHelloAsync&pid=60654&release=&side=provider&timestamp=1579663467310

**log：**   
[22/01/20 11:28:00:749 CST] main  INFO config.ServiceConfig:  [DUBBO] Register dubbo service org.apache.dubbo.demo.DemoService url dubbo://192.168.3.54:20880/org.apache.dubbo.demo.DemoService?anyhost=true&application=dubbo-demo-api-provider&bind.ip=192.168.3.54&bind.port=20880&deprecated=false&dubbo=2.0.2&dynamic=true&generic=false&interface=org.apache.dubbo.demo.DemoService&methods=sayHello,sayHelloAsync&pid=60654&release=&side=provider&timestamp=1579663467310 to registry registry://127.0.0.1:2181/org.apache.dubbo.registry.RegistryService?application=dubbo-demo-api-provider&dubbo=2.0.2&pid=60654&registry=zookeeper&timestamp=1579663240895, dubbo version: , current host: 192.168.3.54

**serviceKey:**
org.apache.dubbo.demo.DemoService:20880

