# Dubbo系列微服务

### 1.注册中心zookeeper

首先，下一个zookeeper，解压之后，在文件夹下简历data文件夹用来存放数据，复制conf文件夹下的zoo_sample.cfg文件，修改内容dataDir=../data，然后保存该文件名字为zoo.cfg

然后在bin下面启动zkServer.cmd，然后此处只是把它当做一个注册中心，如果想进一步了解这个，请参看zookeeper文档

2.









### dubbo配置相关

#### 普通的ssm项目

##### 服务提供者

pom.xml

```xml
  <!--引入dubbo-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.6.2</version>
        </dependency>

        <!--注册中心使用的是zookeeper，引入操作zookeeper的客户端-->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>2.12.0</version>
        </dependency>
```

provider.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
              http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
                  http://dubbo.apache.org/schema/dubbo
                    http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    
    <!--1.指定当前服务/应用的名字（同样的服务名字相同，不要和别的服务同名）-->
    <dubbo:application name="user-service-provider"/>
    <!--2.指定注册中心的位置，有下面两种形式-->
    <!--<dubbo:registry address="zookeeper://127.0.0.1:2181"/>-->

    <dubbo:registry protocol="zookeeper" address="127.0.0.1:2181"/>
    
    <!--3.指定通信规则（通信协议，通信端口）-->
    <dubbo:protocol name="dubbo" port="20080"/>
    
    <!--4.暴露服务(ref指向真正的实现对象)-->
    <dubbo:service interface="com.hua.gmall.service.UserService" ref="userServiceImpl"/>

    <bean id="userServiceImpl" class="com.hua.gmall.service.impl.UserServiceImpl">

    </bean>

    <!--监控中心协议，如果为protocol="registry"，表示从注册中心发现监控中心地址，否则直连监控中心。-->
    <dubbo:monitor protocol="registry"/>


</beans>
```



##### 服务消费者

pom.xml

同上面服务提供者



consumer.jsp

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo" xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
             http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
                http://dubbo.apache.org/schema/dubbo

                  http://dubbo.apache.org/schema/dubbo/dubbo.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
    <dubbo:application name="order-service-consumer"  />

    <!-- 使用multicast广播注册中心暴露发现服务地址 -->
    <dubbo:registry address="zookeeper://127.0.0.1:2181" />

    <!-- 声明需要调用的远程服务的接口，生成远程服务代理 -->
    <dubbo:reference id="userService" interface="com.hua.gmall.service.UserService" />


    <context:component-scan base-package="com.hua.gmall.service.impl"/>

    <!--监控中心协议，如果为protocol="registry"，表示从注册中心发现监控中心地址，否则直连监控中心。-->
    <dubbo:monitor protocol="registry"/>

    
</beans>
```



#### springboot项目

##### 服务提供者





##### 服务消费者





#### 配置文件相关



##### 覆盖策略

![](C:\Users\lenovo\Desktop\dubbo\image\覆盖策略.png)



注意，上面的xml配置也就对应着springboot中的application.properties和application.yml配置文件

具体配置可以在官方文档中查找

[dubbo官方文档](http://dubbo.apache.org/zh-cn/docs/user/configuration/properties.html)

![](C:\Users\lenovo\Desktop\dubbo\image\dubbo配置.png)

##### 常用配置

1.启动时检查

直接参考官方文档的实例

官方文档三个配置的含义：：

配置服务消费者在启动的时候某个服务在注册中心没有不报错

配置服务消费者在启动的时候所有服务在注册中心没有不报错

配置服务提供者在启动的时候没有注册中心不报错



2.超时属性

就是超过多少秒这个服务没反应就中断

规则：

以 timeout 为例，显示了配置的查找顺序，其它 retries, loadbalance, actives 等类似：

- 方法级优先，接口级次之，全局配置再次之。
- 如果级别一样，则消费方优先，提供方次之。

其中，服务提供方配置，通过 URL 经由注册中心传递给消费方。

```xml
    
<!--方法级，消费者-->
<dubbo:reference interface="com.hua.gmall.service.UserService">
        <dubbo:method name="getUserAddressList" timeout="1000"/>
</dubbo:reference>

<!--方法级，消费者-->
<dubbo:service interface="com.hua.gmall.service.UserService">
  	<dubbo:method name="getUserAddressList" timeout="1000"/>
</dubbo:service>

<!--接口级，消费者 -->
    <dubbo:reference id="userService" interface="com.hua.gmall.service.UserService" timeout="1000" />

<!--接口级，消费者 -->
 <dubbo:service interface="com.hua.gmall.service.UserService" ref="userServiceImpl" timeout="1000"/>

<!--全局配置，消费者-->
<dubbo:consumer timeout="1000"/>

<!--全局配置，消费者-->
 <dubbo:provider timeout="1000"/>
```



3.重试次数

retries就是配置上面那个超时之后规定多试几次，这个配置是一个整数，这个重试次数不包含第一次调用

这个东西如果有多个服务提供者，那么服务消费者重试的时候，如果这个失败了，就会试另一个

幂等（方法每次执行完之后结果相同）：设置重试次数（查询、删除、修改，修改和删除方法最好做成幂等的）

非幂等：不能设置重试次数



4.多版本

```xml
version="1.0.0
```

可以在服务提供者设置不同的版本的对应的接口的实现，然后写上对应的版本

设置完之后，就可以在服务消费者里设置调用哪个版本的实现，如果写*的话那么就是随机调用一个。。。



5.本地存根

也就是在调用远程的那个实现之前，在本地判断一下条件，满足了再去调

服务消费者本地的实现类

```java
package com.hua.gmall.service.impl;

import com.hua.gmall.bean.UserAddress;
import com.hua.gmall.service.UserService;
import org.apache.commons.lang3.StringUtils;

import java.util.List;

public class UserServiceImpl implements UserService {

    private final UserService userService;

    public UserServiceImpl(UserService userService){
        this.userService=userService;
    }



    public List<UserAddress> getUserAddressList(String userId) {

        if(StringUtils.isNotBlank(userId)){
            return userService.getUserAddressList(userId);
        }
        return null;
    }
}
```



服务消费者的配置文件

```xml
<dubbo:service interface="com.foo.BarService" stub="true" />
<!--还可以使用下面的配置方式-->
<dubbo:service interface="com.foo.BarService" stub="com.foo.BarServiceStub" />
```

springboot版本的配置（在注解@reference上指定stub属性即可）

```java
@Reference(stub = "com.hua.gmall.service.impl.UserServiceStub")
UserService userService;
```

### springboot项目的其他几种方式

1）上面那种配置方式，在application.properties配置属性，使用@Service暴露服务，使用@Reference引用服务

2）保留dubbo.xml配置文件

导入dubbo-start，使用@ImportResource导入配置文件即可

3）使用javaconfig

将每一个组件手动创建到容器中，dubbo的对应的配置都有一个对应的xxxConfig类





### Zookeeper宕机与dubbo直连

现象：zookeeper注册中心宕机，还可以消费dubbo暴露的服务

原因

健壮性

监控中心宕掉不影响使用，还是丢失部分采样数据

数据库宕掉后，注册中心仍能通过缓存提供服务列表查询，但不能注册新服务

注册中心对等集群，任意一台宕掉后，将自动切换到另一台

注册中心全部宕掉后，服务提供者和服务消费者仍能通过本地缓存通讯

服务提供者无状态，任意一台宕掉后，不影响使用

服务提供者全部宕掉后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复





### 负载均衡

![1544102490162](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1544102490162.png)

默认为随机，上面展示了如何设置其他负载均衡的策略，还有如果要调整权重的话， 那么可以在zookeeper注册中心的那个图形界面上调节



### 服务降级

什么事服务降级？

当服务器压力剧增的情况下，根据实际业务情况及流量，对一些服务和页面有策略的不处理或换种简单的方式处理，从而释放服务器资源以保证核心交易正常运作或高校运作

可以通过服务降级功能临时屏蔽某个出错的非关键服务，并定义降级后的返回策略

向注册中心写入动态配置覆盖规则





### 集群容错

dubbo提供了多种容错方案，默认为failover重试

failfast  Cluster：快速失败，只发起一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增记录。

Failsafe  Cluster：失败安全，出现异常时，直接忽略。通常用于写入审计日志等操作

Failback Cluster：失败自动回复，后台记录失败请求，定时重发，通常用于消息通知操作

Forking Cluster：并行调用多个服务器，只要一个成功都返回。通常用于实时性要求较高的读操作，但需要浪费更多服务器。可通过fork="2"来设置最大并行数

Broadcast Cluster：广播调用所有提供者，逐个调用，任意一台报错则报错【2】。通常用于通知所有与提供者更新缓存或日志等本地资源信息

集群模式配置

按照以下实例服务提供方和消费方配置集群模式

```xml
<dubbo:service cluster="failsafe"/>
或
<dubbol:reference cluster="failsafe"/>
```

一般的服务容错，是来整合hystrix来实现的

整合hystrix

Hystrix旨在通过控制那些访问远程系统、服务和第三方库的节点，从而对延迟和故障提供更强大的容错能力。Hystrix具备拥有回退机制和断路器功能的线程和信号隔离，请求缓和请求打包，以及监控和配置等功能