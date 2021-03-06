# Spring Cloud

### Eureka Server细节说明

##### 1. application.yml

eureka server分为single模式和ha模式，对应的配置参数也有区别

###### single模式

```properties
##放弃自我发布
eureka.client.register-with-eureka=false
##设置不获取注册服务的信息
eureka.client.fetch-registry=false 
```

###### ha模式

```properties
##自我注册发布开启，因为默认开启，可以不用管
eureka.client.register-with-eureka=true

##自我保护机制，注册服务器会维护和服务之间的心跳，当超过一定的阈值时设置当前服务是否需要保护起来，如果是测试环境建议为false,生成环境建议为true开启保护，防止网络波动。如果设置关闭保护那就需要设置对应的心跳阈值来控制
##eureka.server.renewal-percent-threshold=0.5
eureka.server.enable-self-preservation=false

##将此eureka服务注册的地址，可以注册多个（会自动过滤自己）
##PS: 注意发布地址后面有/eureka，不是服务启动地址
eureka.client.serviceUrl.defaultZone=http://eurekanode2:2222/eureka/,http://eurekanode1:2223/eureka/ 
```

###### ip地址

```properties
##生产环境一般用IP地址来定义服务
eureka.instance.hostname=127.0.0.1
eureka.instance.prefer-ip-address= ture
```

![1585818317858](https://github.com/TrimGHU/reborn/blob/master/images/1585818317858.png)



##### 2. 加入权限

加入security的依赖

```xml
<dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-security</artifactId>
</dependency>
```

加入yml配置即可

```properties
##是否启用eureka认证security:
security:
	basic:
  		enabled: true
	user:
  		name: XXX
  		password: XXX
```



##### 3. MAVEN打包可运行JAR包（题外话）

IntelliJ IDE默认maven打包是无法运行的，打出来的jar运行也是提示无主清单属性的

需要在pom里配置main class以及加入引用jar配置，maven install出来的jar才可以执行

```
<build>
        <finalName>eureka-server</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                    <mainClass>com.XXX.EurekaServerApplication</mainClass>
                    <layout>ZIP</layout>
                </configuration>
                <executions>
                    <execution>
                        <configuration>
                            <classifier>exec</classifier>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

### Eureka Client细节说明

##### 1. application.yml

###### 正常使用

```yml
eureka:
  client:
    serviceUrl:
      ## 当注册eureka服务是集群时，往一个地址注入，集群内会自动同步注册
      defaultZone: http://127.0.0.1:1002/eureka
      ## 当eureka服务需要用户名和密码时使用下面的方式
      ## http://user:password@localhost:8761/eureka
```

###### 自定义元数据

```properties
## 元数据配置，不影响正常eureka使用，可以用于云商发布版本 或者 提供给客户端的常量等等
eureka：
	instance：
		metadataMap:
			username: xxx
			key: xxx
```

##### 2. 启动类Application.class

@EnableEurekaClient 只发现eureka的注册服务。

@EnableDiscoveryClient 能发现很多种类的注册服务。

![1585821358532](https://github.com/TrimGHU/reborn/blob/master/images/1585821358532.png)

### Consul

##### 1. 服务注册与发现

##### 2. 分布式锁

### Ali-Nacos

##### 1. 服务注册与发现

##### 2. 配置中心（类似Spring Cloud Config）

##### 3. 集群部署



### Ribbon & Feign

##### 1. 负载均衡

> Ribbon is a client side load balancer which gives you a lot of control over the behaviour of HTTP and TCP clients. 

Ribbon是一个客户端的负载均衡器，内置多种负载均衡算法

```properties
##轮询 
RoundRobinRule 
##随机
RandomRule  
##会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务,以及并发的连接数量超过阈值的服务,然后对剩余的服务列表按照轮询策略进行访问
AvailabilityFilteringRule 
##根据平均响应时间计算所有服务的权重,响应时间越快,服务权重越大,被选中的机率越高;刚启动时,如果统计信息不足,则使用RoundRobinRule策略,等统计信息足够时,会切换到WeightedResponseTimeRule
WeightedResponseTimeRule 
##先按照RoundRobinRule的策略获取服务,如果获取服务失败,则在指定时间内会进行重试,获取可用的服务
RetryRule
##会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务,然后选择一个并发量最小的服务
BestAvailableRule 
##默认规则,复合判断server所在区域的性能和server的可用性选择服务器
ZoneAvoidanceRule
```

```java
@Bean
public IRule cusomRule(){
	return new RoundRobinRule();
}
```

>Feign is a declarative web service client.  It makes writing web service clients easier.  To use Feign create an interface and annotate it.  It has pluggable annotation support including Feign annotations and JAX-RS annotations. 

Feign是整合了Ribbon的声明式客户端，因为其负载均衡由ribbon完成，所以配置方法和其一致。



##### 2. 引入和调用

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-ribbon</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```

```JAVA
@FeignClient(name = "user")
public interface UserService {

    @GetMapping("/name")
    String getName();
}
```

```JAVA
@Autowired
private RestTemplate restTemplate;

@GetMapping("/name-ribbin")
	public String getUserNameRibbin(){
	return restTemplate.getForObject("http://user/name",String.class);
}
```

```properties
##Feign默认hystrix是关闭的
feign:
  hystrix:
    enabled: true
```



##### 3. 饥饿加载模式

开启饥饿加载模式是指在服务启动时就创建ribbonclient或feignclient，而不是服务调用时创建。

```properties
ribbon.eager-load.enabled=true
ribbon.eager-load.clients=hello-service, user-service
```

##### 4. Hystrix的相关

###### 服务降级 fallback / 依赖隔离 / 熔断(见下方)



### Hystrix

>Hystrix is a library that helps you control the interactions between these distributed services by adding latency tolerance and fault tolerance logic. Hystrix does this by isolating points of access between the services, stopping cascading failures across them, and providing fallback options, all of which improve your system’s overall resiliency.

##### 1. 引入

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```

##### 2. 使用和配置

`@SpringCloudApplication` 类上加入组合注解或者`@EnableCircuitBreaker`，开启熔断

`@HystrixCommand(fallbackMethod = "xxxx") `使用在方法上可以配置超时时间，配合Ribbon使用

`@FeignClient(name = "user", fallback = xxxx.class)` 使用在远程调用的Interface上，配合Feign使用

###### 超时配置

```java
@HystrixCommand(fallbackMethod="fallback",
	commandProperties = {
	     @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "1000" )
	}
)
```

+

```java
@HystrixCommand(fallbackMethod="fallback",commandKey="||userGetKey||")
```

```properties
hystrix.command.||userGetKey||.execution.isolation.thread.timeoutInMilliseconds = 13000
```

+

```properties
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=3000
```

+

```properties
hystrix.command.||UserRemoteClient#getUser(Long)||.execution.isolation.thread.timeoutInMilliseconds = 300
```

+

```properties
hystrix.command.||service-id||.execution.isolation.thread.timeoutInMilliseconds=3000
```

##### 3. 隔离策略

###### 线程池（默认）

​	即每组服务都定义隔离开的一个线程池，每个线程池之间互不影响。

​	此方法对服务器的性能有一定的损耗，因为多线程之间的数据互通会产生一定的延迟，并且易生成碎片	

###### 信号量

​    即对每个资源调用限制并发数，缺点是无法对超时自动降级，需要等待客户端自己超时处理

##### 4. 熔断

​    断路器的三个重要参数：快照时间窗、请求总数下限、错误百分比下限。 

###### 快照时间窗

​	 断路器确定是否打开需要统计一些请求和错误数据，而统计的时间范围就是快照时间窗，默认为最近的10秒。 

###### 请求总数下限

​	 在快照时间窗内，必须满足请求总数下限才有资格根据熔断。默认为20，意味着在10秒内，如果该hystrix命令的调用此时不足20次，即时所有的请求都超时或其他原因失败，断路器都不会打开。 

###### 错误百分比下限

​	 当请求总数在快照时间窗内超过了下限，比如发生了30次调用，如果在这30次调用中，有16次发生了超时异常，也就是超过50%的错误百分比，在默认设定50%下限情况下，这时候就会将断路器打开。 

##### 5. 请求合并

​	支持在时间窗内依赖同以服务的多个请求合并，独立线程池批量请求服务端（服务端支持批量请求接口），处理完毕之后一起返回。

​	其实请求合并的作用还是需要仔细当前场景下是否适合，因为毕竟是在一个时间窗口内的，如果一个高延迟的服务合并确实会起到一定的作用，但是作为一个低延迟高低并发的接口呢？这就不一定了。关键在于2点因素，**服务的处理时间** 和 **窗口时间内的并发量**。

![hystrix-collapse-after](https://github.com/TrimGHU/reborn/blob/master/images/hystrix-collapse-after.png)

##### 6. Dashboard

######  Actuator 

	 ###### Turbine 



### Ali-Sentinel

##### 1. 应用

##### 2. 隔离策略

###### 信号量 + 并发线程数限制

##### 3. 比较Hystrix

![sentinel-hystrix](https://github.com/TrimGHU/reborn/blob/master/images/sentinel-hystrix.png)


### Zull & Gateway

> Zuul is a JVM based router and server side load balancer by Netflix. 

##### 1. 引入

```XML
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-zuul</artifactId>
</dependency>
```

##### 2. 配置文件

```properties
##最大连接数 默认200
zuul.host.maxTotalConnections=200
##每个路由最大链接数20
zuul.host.maxPerRouteConnections=20
##默认hystrix隔离是信号量隔离SEMAPHORE
zuul.ribbonIsolationStrategy=THREAD
```

###### 路由配置

```properties
#####《传统模式》
zuul.routes.userweb.path=/user/**
zuul.routes.userweb.url=http://12.12.1.1:8080/user

#####《服务实例化模式》
##服务实例自动化配置
zuul.routes.userweb.path=/user/**
zuul.routes.userweb.serviceId=user
##或者
zuul.routes.userweb=/user/**

##其中 * 、**、? 三种通配符也需要了解 

##路由配置忽略
zuul.ignored-patterns=/**/hello/**

##内置hystrix 和 ribbon超时时间以及重试问题
hystrix.command.default.execution.isolation.thread.timeoutInMillseconds=5000
ribbon.ConnectTimeOut=2000
ribbon.ReadTimeOut=2000

##关闭重试
zuul.retryable=false
##关闭某个实例的重试
zuul.routes.user.retryable=false
```

###### 饥饿加载问题

如果路由规则不是指定，而是采用默认路由的话，那光enabled是无法起作用的，因为系统需要知道你指定的加载路由的服务有哪些，所以需要忽略所有的默认路由，让所有服务的路由规则都在系统的维护中才能完成提前加载。

```properties
zuul.ribbon.eager-load.enabled=true
zuul.ignored-services=*
```

###### cookie和重定向

默认zuul路由配置下是会忽略Cookie 和 Authorization相关信息，不会继续往下传递的，所以会导致有些会话信息丢失，如果需保持就需要在配置中指定不忽略向下传递。

重定向问题同理也是忽略了header中的host导致

```properties
##全局设置
zuul.sensitive-headers=
##指定路由设置
zuul.routes.user.sensitive-headers=
zuul.routes.user.custom-sensitive-headers=true

##加入host的传递
zuul.add-host-header=true
```

##### 3. 过滤器&认证

###### 过滤器4个方法

- `filterType`：过滤器的类型，它决定过滤器在请求的哪个生命周期中执行。
  - `pre` 代表会在请求被路由之前执行。
  - `route` 代表在请求被路由时执行
  - `post` 代表在routing和error过滤器之后执行
  - `error` 代表处理请求时发生错误时执行

- `filterOrder`：过滤器的执行顺序。数字越低越先执行。

- `shouldFilter`：判断该过滤器是否需要被执行。true标识执行，false标识不执行。

-  `run`：过滤器的具体逻辑。

  ![zull](https://github.com/TrimGHU/reborn/blob/master/images/zull.png)

###### 常见实例1：鉴权认证

```JAVA
public class AccessFilter extends ZuulFilter  {

    private static Logger log = LoggerFactory.getLogger(AccessFilter.class);

    @Override
    public String filterType() {
        return "pre";
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();

        Object accessToken = request.getParameter("accessToken");
        if(accessToken == null) {
            log.warn("access token is empty");
            ctx.setSendZuulResponse(false);
            ctx.setResponseStatusCode(401);
            return null;
        }
        log.info("access token ok");
        return null;
    }
}
```

###### 常见实例2：异常处理

```java
//SendErrorFilter.class
@Override
public boolean shouldFilter() {
	RequestContext ctx = RequestContext.getCurrentContext();
	// only forward to errorPath if it hasn't been forwarded to already
	return ctx.getThrowable() != null
				&& !ctx.getBoolean(SEND_ERROR_FILTER_RAN, false);
}
```

常见的zuul过滤器类SendErrorFilter只有在ctx.getThrowable() != null才会执行，并且在spring cloud的D版中将SendErrorFilter从post改为error类别过滤器解决很多问题，例如以前type=post发生的错误，需要自己自定义方法解决，现在SendErrFilter也能很好的handle了

![spring-cloud-zuul-core-filter](https://github.com/TrimGHU/reborn/blob/master/images/spring-cloud-zuul-core-filter.png)




### Stream

> Spring Cloud Stream is a framework for building message-driven microservice applications. 
>
>SpringCloud Stream是一个整合当前主流MQ一个集成框架，类似于SpringBoot和SpringMVC之间的关系。

**PS： 下文全部都用RabbitMQ作为传输**

RabbitMQ安装方式自行百度，只是要注意erlang和rabbit之间的版本关联[<<走你>>](https://www.rabbitmq.com/which-erlang.html)

```properties
##rabbit 默认端口说明
4369 erlang发现口 
5672 client端通信口 
15672 管理界面ui端口 
25672 server间内部通信口 
```

##### 1. 引入

```XML
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
```

##### 2. 配置

在配置好配置文件之后，启动时会自动在相应的mq中产生相应的queue通道。

很多种配置方式

1. 多个rabbitmq配置
2. 多queue通道的
3. 多种类型mq的
4. 多binder的

```properties
spring:
  cloud:
    stream:
      bindings:
      	add_user_output:
      	  destination: add_user_queue
      	  group: adduser
        email_input:
          destination: add_user_queue
          group: email
        addition_info_input:
          destination: add_user_queue
          group: additioninfo
  ###还有很多种配置方
  ###1. 支持多rabbitmq端
  ###2. 多种类型mq
  ###3. 多binder的配置
  rabbitmq:
    host: xx.xx.xx.xx
    port: 5672
    username: xxx
    password: xxx
```

##### 3. 自定义输出和接受渠

从官方文档中可以了解到引入binder，是由它和 消息中间件 MQ去交互，然而应用程序只需要和binder透明的交互就好，并且这交互的渠道叫Channel分为input和output，相当于发布和订阅。

![SCSt-with-binder](https://github.com/TrimGHU/reborn/blob/master/images/SCSt-with-binder.png)

官方默认有输出和接受渠，对应是Source.class, Sink.class

```JAVA
package org.springframework.cloud.stream.messaging;

import org.springframework.cloud.stream.annotation.Input;
import org.springframework.messaging.SubscribableChannel;

/**
 * Bindable interface with one input channel.
 *
 * @see org.springframework.cloud.stream.annotation.EnableBinding
 * @author Dave Syer
 * @author Marius Bogoevici
 */
public interface Sink {

	String INPUT = "input";

	@Input(Sink.INPUT)
	SubscribableChannel input();
}
```

```java
package org.springframework.cloud.stream.messaging;

import org.springframework.cloud.stream.annotation.Output;
import org.springframework.messaging.MessageChannel;

/**
 * Bindable interface with one output channel.
 *
 * @see org.springframework.cloud.stream.annotation.EnableBinding
 * @author Dave Syer
 * @author Marius Bogoevici
 */
public interface Source {

	String OUTPUT = "output";

	@Output(Source.OUTPUT)
	MessageChannel output();

}
```

但如果出现同一渠道被多方接收，或者多个渠道被1方同时接收时就需要自定义了，且自定义也更好管理相应的通道。

```JAVA
public interface ISourceReceiver {
    String EMAIL_INPUT = "email_input";
    String ADDITION_INFO_INPUT = "addition_info_input";

    /**
     * 添加用户触发的发送email
     * @return
     */
    @Input(EMAIL_INPUT)
    SubscribableChannel emailInput();

    /**
     * 添加用户触发的添加附加信息
     * @return
     */
    @Input(ADDITION_INFO_INPUT)
    SubscribableChannel additionInfoInput();
}
```

```JAVA
public interface ISinkSender {

    String ADD_USER_OUTPUT = "add_user_output";

    /**
     * 添加用户的mq输出
     * @return
     */
    @Output(ADD_USER_OUTPUT)
    MessageChannel addUserOutput();
}
```

##### 4. 示例描述

###### 场景描述

​	创建用户之后通知小心添加附加信息 和 发送消息通知

###### 对应工程

​	front-web  定义触发事件MQ消息add_user_output

​	user 用户服务

​	message  定义接受并处理MQ消息email_input和addition_info_input

###### 触发流程

​	请求front-web新增用户，调用user接口创建用户成功后触发产生MQ消息，由message消息中心email_input和addition_info_input接受到，并作出相应的处理。



##### 5. 死信DLQ队列
###### 相关配置和说明

```properties
spring:
  cloud:
    rabbit:
        bindings:
          email_input:
            consumer:
              ##消息消费失败会重新加入队列，直到消费成功
              requeue-rejected: true
              ##DLQ队列
              auto-bind-dlq: true
              ##DLQ死信队列里的消息,重新执行时错误原因会放入header
              republish-to-dlq: true
              ##DLQ队列中消息的存活时间
              dlq-ttl: 10000
    stream:
      bindings:
        email_input:
          consumer:
            ##自动重试次数,如果设置最大重试次数>1，则会将失败的消息传入死信队列
            max-attempts: 3
```

##### 6. 梯度执行消息

###### 用DLX实现

DLQ在没有配置DLX的情况下，会默认产生一个name=DLX的死信通道

监听死信队列，然后通过死信道通发送给原始队列去梯度重复执行即可

**（开启RabbitMQ的延时插件）**

```properties
server:
  port: 6001

spring:
  application:
    name: message
  cloud:
    stream:
      bindings:
        email_input:
          consumer:
            ##自动重试次数
            max-attempts: 1
          destination: add_user_queue
          group: email
        addition_info_input:
          destination: add_user_queue
          group: additioninfo
      rabbit:
        bindings:
          email_input:
            consumer:
              ##消息消费失败会重新加入队列，直到消费成功
              ##requeue-rejected: true
              ##DLQ队列
              auto-bind-dlq: true
              ##DLQ死信队列里的消息,重新执行时错误原因会放入header
              ##republish-to-dlq: true
              ##DLQ队列中消息的存活时间
              dlq-ttl: 5000
  rabbitmq:
    host: xx.xx.xx.xx
    port: 5672
    username: xx
    password: xx

```

```java
package com.hugui.configuration;

import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.ImmediateAcknowledgeAmqpException;
import org.springframework.amqp.core.*;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Map;


@Slf4j
@Configuration
public class DelayMsgConfiguration {
    private static final String ORIGINAL_QUEUE = "add_user_queue.email";

    private static final String DLQ = ORIGINAL_QUEUE + ".dlq";

    private static final String X_RETRIES_HEADER = "x-retries";

    private static final String DELAY_EXCHANGE = "DLX";


    @Autowired
    private RabbitTemplate rabbitTemplate;

    @RabbitListener(queues = DLQ)
    public void rePublish(Message failedMessage) {
        log.info("receiver from email dlq");

        Map<String, Object> headers = failedMessage.getMessageProperties().getHeaders();
        Integer retriesHeader = (Integer) headers.get(X_RETRIES_HEADER);
        if (retriesHeader == null) {
            retriesHeader = Integer.valueOf(0);
        }
        if (retriesHeader < 3) {
            headers.put(X_RETRIES_HEADER, retriesHeader + 1);
            headers.put("x-delay", 5000 * retriesHeader);
            this.rabbitTemplate.send(DELAY_EXCHANGE, ORIGINAL_QUEUE, failedMessage);
        }
        else {
            //需要人工处理
            //发邮件提醒工作人员 或者 存入数据库
            log.info("need administrator to fix it!");
            throw new ImmediateAcknowledgeAmqpException("Failed after 4 attempts");
        }
    }

    @Bean
    public DirectExchange delayExchange() {
        DirectExchange exchange = new DirectExchange(DELAY_EXCHANGE);
        exchange.setDelayed(true);
        return exchange;
    }

    @Bean
    public Binding bindOriginalToDelay() {
        return BindingBuilder.bind(new Queue(ORIGINAL_QUEUE)).to(delayExchange()).with(ORIGINAL_QUEUE);
    }
}
```

