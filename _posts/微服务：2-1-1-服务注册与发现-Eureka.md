---
title: 微服务：2-1-1 服务注册与发现-Eureka
date: 2020-10-25 23:23:34
tags:
---
### 1、Eureka基础知识
#### 1、什么是服务治理
  Spring Cloud 封装了Netflix公司开发的Eureka模块来实现服务治理。
  
  在传统的RPC远程调用框架中，管理每个服务与服务之家依赖关系比较复杂，管理比较复杂，所以需要使用服务治理，管理服务与服务之间的依赖关系，可以实现服务调用、负载均衡、容错等，实现服务发现与注册。

#### 2、什么是服务注册与发现
![eureka1](./eureka1.jpg)
![eureka2](./eureka2.jpg)

#### 3、Eureka两组件
![eureka3](./eureka3.jpg)

### 2、单机Eureka构建步骤
![eureka4](./eureka4.jpg)
![eureka18](./eureka18.jpg)

#### 1、IDEA生成eurekaServer端服务**注册中心**（类似物业中心）
1、建Module
![eureka5_module](./eureka5_module.jpg)
2、改POM
![eureka6_pom](./eureka6_pom.jpg)
3、写YML
![eureka7_yml](./eureka7_yml.jpg)
4、主启动
![eureka8_application](./eureka8_application.jpg)
5、测试
![eureka9_localhost](./eureka9_localhost.jpg)

#### 2、EurekaClient端cloud-provider-payment8001注册进EurekaServer成为**服务提供者**provider（类似于入驻该物业中心的奶茶店）
1、建Module

2、改POM
![eureka10_provider_pom](./eureka10_provider_pom.jpg)
3、写YML
![eureka11_provider_yml](./eureka11_provider_yml.jpg)
4、主启动
![eureka12_provider_application](./eureka12_provider_application.jpg)
5、测试
![eureka13_provider_localhost](./eureka13_provider_localhost.jpg)

6、yml配置文件中，spring的application的name就是Eureka启动后的application的name
![eureka14_provider](./eureka14_provider.jpg)

#### 3、EurekaClient端cloud-consumer-order80注册进EurekaServer成为**服务消费者**consumer（类似于奶茶店消费者）
1、建Module

2、改POM
与上面一致

3、写YML
![eureka15_consumer_yml](./eureka15_consumer_yml.jpg)
4、主启动
![eureka16_consumer_application](./eureka16_consumer_application.jpg)
5、测试
![eureka17_consumer_localhost](./eureka17_consumer_localhost.jpg)

### 3、集群Eureka构建步骤
#### 1、Eureka集群原理说明
![eureka20_cluster_theory](./eureka20_cluster_theory.jpg)
![eureka19_cluster](./eureka19_cluster.jpg)

#### 2、EurekaServer集群环境构建步骤
1、参考cloud-eureka-server7001，新建cloud-eureka-server7002
![eureka21_server7002_module](./eureka21_server7002_module.jpg)

2、改POM
![eureka22_server7002_pom](./eureka22_server7002_pom.jpg)

总工程的pom文件
![eureka23_server7002_parent_pom](./eureka23_server7002_parent_pom.jpg)

3、修改映射配置
![eureka24_server7002_host](./eureka24_server7002_host.jpg)

4、写YML（之前是单机，现在改成集群）
![eureka25_server7001&7002_pom](./eureka25_server7001&7002_pom.jpg)

5、主启动
![eureka26_server7001&7002_application](./eureka26_server7001&7002_application.jpg)

#### 3、将支付服务8001微服务发布到7001和7002两台Eureka集群配置中
1、8001配置7001和7002两台Eureka集群
![eureka27_server8001_yml](./eureka27_server8001_yml.jpg)

#### 4、将订单服务服务800微服务发布到7001和7002两台Eureka集群配置中
1、80配置7001和7002两台Eureka集群
![eureka28_server80_yml](./eureka28_server80_yml.jpg)

#### 5、测试
1、先启动EurekaServer，7001和7002服务
![eureka29_server7001](./eureka29_server7001.jpg)

2、再启动服务提供者provider，8001
![eureka30_server7002](./eureka30_server7002.jpg)
3、再启动服务消费者，80

4、访问
![eureka31_server80](./eureka31_server80.jpg)
![eureka32](./eureka32.jpg)

#### 6、支付服务提供者**集群环境**构建
服务提供者是一个集群：
![eureka39](./eureka39.jpg)

1、参考cloud-provider-payment8001，新建cloud-provider-payment8002
![eureka33_server8002_module](./eureka33_server8002_module.jpg)

2、改POM
复制8001的pom

3、写yml
复制8001的yml，将端口改成8002即可

5、主启动
复制8001

5、业务类
复制8001

5、修改8001和8002的Controller
简单标注该provider的端口号
![eureka34_server_controller](./eureka34_server_controller.jpg)

6、在80消费者controller中，修改访问服务提供者的路径，把写死的路径改成消费提供者**集群**对外暴露的名称：CLOUDER-PATMENT-SERVER
![eureka36_server80](./eureka36_server80.jpg)

7、**负载均衡**
在80消费者controller中，修改访问服务提供者的路径，把写死的路径改成消费提供者**集群**对外暴露的名称：CLOUDER-PATMENT-SERVER之后，当访问https://CLOUDER-PATMENT-SERVER路径时，服务器不知道你到底要访问CLOUDER-PATMENT-SERVER下的哪一台机器，所以要使用**@LoadBalance**注解赋予RestTemplate负载均衡的能力

修改ApplicationContextConfig
![eureka37_server80_config](./eureka37_server80_config.jpg)

8、启动7001和7002
![eureka35_server70017002](./eureka35_server70017002.jpg)

9、启动8001和8002

10、启动80
启动80端口的消费者，消费者访问服务提供者8001和8002时，服务提供者集群采用**轮询机制**给消费者提供服务。
![eureka38_80](./eureka38_80.jpg)

### 4、服务发现discovery
1、对于注册进eureka里面的微服务，可以通过服务发现来获得该服务信息

以8001为例：
2、修改cloud-provider-payment8001的controller
![eureka40_discovery_controller](./eureka40_discovery_controller.jpg)

3、8001主启动类
添加注解**@EnableDisCoveryClient**
![eureka41_discovery_application](./eureka41_discovery_application.jpg)

4、自测
![eureka42_discovery_run](./eureka42_discovery_run.jpg)
![eureka43_discovery_run2](./eureka43_discovery_run2.jpg)

### 5、Eureka自我保护
#### 1、故障现象
![eureka44](./eureka44.jpg)

#### 2、导致原因
1、某时刻，某一个微服务不可用了，Eureka不会立刻清理，依旧会对该服务的信息进行保存

2、属于CAP中的AP分支

![eureka45](./eureka45.jpg)

#### 3、怎么禁止自我保护
某时刻，某一个微服务不可用了，Eureka立刻清理不保存该服务的信息

以7001和8001为例
1、修改7001yml
![eureka46](./eureka46.jpg)

2、修改8001yml
![eureka47](./eureka47.jpg)