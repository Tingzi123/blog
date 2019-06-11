title: SpringBoot与消息
---
1、JMS&AMQP简介

### 异步处理

同步机制：
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg1.png?raw=true)
用户发送注册消息后，服务器依次完成注册、发送注册邮件、发送注册短信，总共耗时150ns

并发机制：
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg2.png?raw=true)
用户发送注册消息后，服务器完成注册，然后使用多线程分别给用户发送注册邮件和发送注册短信，总共耗时100ns

注：注册成功后用户并不需要立刻收到邮件和短信，只需接受到注册结果的消息即可。

异步消息队列机制：
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg3.png?raw=true)
用户发送注册消息后，服务器完成注册，然后写入消息队列后立即通知用户注册成功，再异步发送注册邮件和发送注册短信，总共耗时55ns

### 应用解耦
使用中间件，将两个服务解耦，一个写入，一个订阅
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg4.png?raw=true)
将订单系统和库存系统单独抽取出来（解耦），在订单系统下单后将信息写入消息队列，库存系统通过订阅消息队列（只要队列中有消息）获取消息，再进行后续操作。

3、流量削锋

例如消息队列的FIFO，限定元素的长度，防止出现多次请求导致的误操作
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg5.png?raw=true)
例如秒杀场景，消息队列中有10000个位置，一共十万个请求，前10000个用户请求抢占位置，后90000个请求直接pao返回失败信息，在处理抢占成功的请求的后续操作。

概述

1、大多数应用，可以通过消息服务中间件来提升系统的异步通信、拓展解耦能力

2、消息服务中的两个重要概念：

消息代理（message broker）和目的地（destination），当消息发送者发送消息以后，将由消息代理接管，消息代理保证消息传递到指定的目的地。

3、消息队列主要的两种形式的目的地

1）、队列（queue）：点对点消息通信【point-to-point】，取出一个移除一个，一个发布，多个接受者。（例A、B、C、D同时想要去接受，但是只要一个人接受成功消息，消息就会被删除，其他人不可能再拿到）

2）、主题（topic）:发布（publish）/订阅（subscribe）消息通信，多人【订阅者】可以同时接到消息

4、JMS(Java Message Service) Java消息服务：
基于JVM消息规范的代理。ActiveMQ/HornetMQ是JMS的实现

5、AMQP(Advanced Message Queuing Protocol)
高级消息队列协议，也是一个消息代理的规范，兼容JMS
RabbitMQ是AMQP的实现
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg6.png?raw=true)

6、SpringBoot的支持
spring-jms提供了对JMS的支持
spring-rabbit提供了对AMQP的支持
需要创建ConnectionFactory的实现来连接消息代理
提供JmsTemplate,RabbitTemplate来发送消息

@JmsListener(JMS).@RabbitListener(AMQP)注解在方法上的监听消息代理发布的消息

@EnableJms,@EnableRabbit开启支持

7、SpringBoot的自动配置
JmsAutoConfiguration
RabbitAutoConfiguration

2、RabbitMQ简介
AMQP的实现
1、核心概念
Message:消息头和消息体组成，消息体是不透明的，而消息头上则是由一系列的可选属性组成，属性：路由键【routing-key】,优先级【priority】,指出消息可能需要持久性存储【delivery-mode】

Publisher:消息的生产者，也是一个向交换器发布消息的客户端应用程序

Exchange:交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列

Exchange的4中类型：direct【默认】点对点，fanout,topic和headers, 发布订阅，不同类型的Exchange转发消息的策略有所区别

Queue:消息队列，用来保存消息直到发送给消费者，它是消息的容器，也是消息的终点，一个消息可投入一个或多个队列，消息一直在队列里面，等待消费者连接到这个队列将数据取走。

Binding:绑定，队列和交换机之间的关联，多对多关系

Connection:网络连接，例如TCP连接

Channel:信道，多路复用连接中的一条独立的双向数据流通道，信道是建立在真是的TCP链接之内的虚拟连接AMQP命令都是通过信道发送出去的。不管是发布消息，订阅队列还是接受消息，都是信道，减少TCP的开销，复用一条TCP连接。

Consumer:消息的消费者，表示一个从消息队列中取得消息的客户端的 应用程序

VirtualHost:虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个 vhost 本质上就是一个 mini 版的 RabbitMQ 服务器，拥有自己的队列、交换器、绑定和权限机制。vhost 是 AMQP 概念的基础，必须在连接时指定，RabbitMQ 默认的 vhost 是 / 。

Broker:消息代理，表示消息队列 服务实体
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg7.png?raw=true)

2、RabbitMQ的运行机制

Exchange分发消息时根据类型的不同分发策略有区别，目前共四种类型：direct、fanout、topic、headers 。headers 匹配 AMQP 消息的 header 而不是路由键， headers 交换器和 direct 交换器完全一致，但性能差很多，目前几乎用不到了，所以直接看另外三种类型：

direct：根据路由键直接匹配，一对一（点对点，单播模式）
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg8.png?raw=true)

fanout：不经过路由键，直接发送到每一个队列（广播模式，速度很快，发布订阅的一个参考实现）
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg9.png?raw=true)

topic：类似模糊匹配的根据路由键，来分配绑定的队列（根据路由键的匹配有选择性的广播）
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg10.png?raw=true)

3、RabbitMQ安装测试

1、打开虚拟机，在docker中安装RabbitMQ
```java
#1.安装rabbitmq，使用镜像加速
docker pull registry.docker-cn.com/library/rabbitmq:3-management
[root@node1 ~]# docker images
REPOSITORY                                     TAG                 IMAGE ID            CREATED             SIZE
registry.docker-cn.com/library/rabbitmq        3-management        e1a73233e3be        11 days ago         149 MB
#2.运行rabbitmq
##### 端口：5672 客户端和rabbitmq通信 15672：管理界面的web页面

docker run -d -p 5672:5672 -p 15672:15672 --name myrabbitmq e1a73233e3be

#3.查看运行
docker ps
```

2、打开网页客户端并登陆，账号【guest】,密码【guest】，登陆
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg11.png?raw=true)

3、添加 【direct】【faout】【topic】的绑定关系等

1）、添加Exchange,分别添加exchange.direct、exchange.fanout、exchange.topic
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg12.png?raw=true)

2）、添加 Queues,分别添加atct、atct.news、atct.emps、atctxzz.news
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg13.png?raw=true)

3）、点击【exchange.direct】添加绑定规则
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg14.png?raw=true)

4）、点击【exchange.fanout】添加绑定规则
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg15.png?raw=true)

5）、点击【exchange.topic】添加绑定规则
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg16.png?raw=true)
/*: 代表匹配1个单词
/#：代表匹配0个或者多个单词

4、发布信息测试

【direct】发布命令，点击 Publish message
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg17.png?raw=true)

查看队列的数量
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg18.png?raw=true)

点击查看发送的信息，点击Get Message
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg19.png?raw=true)

【fanout】的发布消息
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg20.png?raw=true)

队列信息，每个队列都添加了一条

【topic】发布信息测试
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg21.png?raw=true)

队列的值
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg22.png?raw=true)
hello.news通过“*.news”匹配到atct.news和atctxzz.news

4、创建工程整合

1、RabbitAutoConfiguration 
2、自动配置了连接工厂 ConnectionFactory 
3、RabbitProperties封装了 RabbitMQ 
4、RabbitTemplate:给RabbitMQ发送和接受消息的 
5、AmqpAdmin：RabbitMQ的系统管理功能组件

### RabbitTemplate
1、新建SpringBoot工程，SpringBoot1.5+Integeration/RabbitMQ+Web

2、RabbitAutoConfiguration文件

3、编写配置文件application.yml
```java
spring:
  rabbitmq:
    host: 10.138.223.126
    port: 5672
    username: guest
    password: guest
```
4、编写测试类,将HashMap写入Queue
```java
@Test
	public void contextLoads() {
		// Message需要自己构建一个；定义消息体内容和消息头
		// rabbitTemplate.send(exchange, routingKey, message);
		// Object 默认当成消息体，只需要传入要发送的对象，自动化序列发送给rabbitmq；
		Map<String,Object> map = new HashMap<>();
		map.put("msg", "这是第一个信息");
		map.put("data", Arrays.asList("HelloWorld", 123, true));
		//对象被默认序列以后发送出去
		rabbitTemplate.convertAndSend("exchange.direct","atct.news",map);
	}
```
5、查看网页的信息，默认使用java序列的方式
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg23.png?raw=true)

6、取出队列的值

取出队列中数据就没了
```java
@Test
	public void receiveAndConvert(){
		Object o = rabbitTemplate.receiveAndConvert("cuzz.news");
		System.out.println(o.getClass());
		System.out.println(o);
	}
```
结果
```java
class java.util.HashMap
{msg=这是第一个信息, data=[HelloWorld, 123, true]}
```

7、使用Json方式传递，并传入对象Book

1）、MyAMQPConfig，自定义一个MessageConverter返回Jackson2JsonMessageConverter
```java
@Configuration
public class MyAMQPConfig {

    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```
转化为json
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg24.png?raw=true)

2）、编写Book实体类
```java
@Data
public class Book {
    private String  bookName;
    private String author;

    public Book(){
    }

    public Book(String bookName, String author) {
        this.bookName = bookName;
        this.author = author;
    }
}
```
3）、测试类
```java
@Test
public void test() {
    // 对象被默认序列以后发送出去
    rabbitTemplate.convertAndSend("exchange.direct","atct.news", new Book("Effect java", "Joshua Bloch"));
}
```
4）、取出数据
```java
@Test
public void reciverAndConvert(){
    Object o = rabbitTemplate.receiveAndConvert("atct.news");
    System.out.println(o.getClass());
    System.out.println(o);
}
```
5）、结果演示
```java
class com.amqp.bean.Book
Book(bookName=Effect java, author=Joshua Bloch)
```

2、开启基于注解的方式

1、新建一个BookService
```java
@Service
public class BookService {
    @RabbitListener(queues = "cuzz.news")
    public void receive(Book book){
        System.out.println(book);
    }

    @RabbitListener(queues = "cuzz")
    public void receive02(Message message){
        System.out.println(message.getBody());
        System.out.println(message.getMessageProperties());
    }
}
```
2、主程序开启RabbitMQ的注解
```java
@EnableRabbit // 开启基于注解的rabbitmq
@SpringBootApplication
public class Springboot10AmqpApplication {

	public static void main(String[] args) {
		SpringApplication.run(Springboot10AmqpApplication.class, args);
	}
}
```
3、AmqpAdmin
创建和删除 Exchange 、Queue、Bind
1）、创建Exchange
```java
@Test
public void createExchange(){
    amqpAdmin.declareExchange(new DirectExchange("amqpadmin.direct"));
    System.out.println("Create Finish");
}
```
2）、创建Queue
```java
@Test
public void createQueue(){
    amqpAdmin.declareQueue(new Queue("amqpadmin.queue",true));
    System.out.println("Create Queue Finish");
}
```
3）、创建Bind规则
```java
@Test
public void createBind(){
    amqpAdmin.declareBinding(new Binding("amqpadmin.queue",Binding.DestinationType.QUEUE , "amqpadmin.direct", "amqp.haha", null));
}
```
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootmsg25.png?raw=true)

删除类似
```java
@Test
public void deleteExchange(){
    amqpAdmin.deleteExchange("amqpadmin.direct");
    System.out.println("delete Finish");
}
```
