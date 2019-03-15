# 工程的建立

### 1.首先建立一个空的项目，然后建立一个父module

pom文件如下所示

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.hua</groupId>
    <artifactId>microservicecloud</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
       <!--这两个配置是保证每次编译都是java8-->
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <!---->
        <junit-version>4.12</junit-version>
        <log4j-version>1.2.17</log4j-version>
        <lombok.version>1.16.18</lombok.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Dalston.SR1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>1.5.9.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>5.0.4</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>1.0.31</version>
            </dependency>
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>1.3.0</version>
            </dependency>
            <dependency>
                <groupId>ch.qos.logback</groupId>
                <artifactId>logback-core</artifactId>
                <version>1.2.3</version>
            </dependency>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>${junit-version}</version>
                <scope>test</scope>
            </dependency>
            <dependency>
                <groupId>log4j</groupId>
                <artifactId>log4j</artifactId>
                <version>${log4j-version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

    
</project>
```

需要注意的是：packing的方式一定要注明为pom，这个项目主要是定义pom文件，然后为后续的子module公用的jar包等统一提取出来，就跟个父类似的

### 2.然后建立api公共模块

1）pom.xml中如下所示

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!--子类里面显示声明才能有明确的继承变现，无意外就是雷飞的默认版本否则自己定义-->
    <parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>com.hua</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../microservicecloud/pom.xml</relativePath>
    </parent>

    <!--当前module自己的名字-->
    <artifactId>microservicecloud-api</artifactId>

    <!--当前Module需要用到的jar包，按自己需求添加，如果父类已经包含了，可以不用写版本号-->
    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>

</project>
```



2）创建一个entity包，然后你就会想写如下的entity，

​	-声明属性

​	-getter和setter

​	-toString

​	-全参数构造方法

​	-无参构造方法

​	-实现序列化接口

因为有lombok，所以说上面的事情都不要自己干了，要安装这个插件在你的idea上，自己找一下自己idea对应的版本https://github.com/mplushnikov/lombok-intellij-plugin/releases，我是在这个人的github上找到的

```java
package com.hua.springcloud.entites;

/**
 * @Author hua
 * @Version 2018/10/31
 */

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.experimental.Accessors;

import java.io.Serializable;

/**
 * Dept(Entity) orm mysql->Dept(table)类表关系映射
 */
@AllArgsConstructor//全参
@NoArgsConstructor//无参
@Data//生成set和get
@Accessors(chain = true)//链式写法，就是set之后接着set
public class Dept implements Serializable {

    private Long deptno;//主键
    private String dname;
    private String db_source;//来自哪个数据库，因为微服务架构可以一个服务对应一个数据库，同一个信息被存储到不同数据库

    public Dept(String dname){
        super();
        this.dname=dname;
    }

}
```



弄完clean，install一下

### 3.部门微服务提供者

1）pom.xml

```xml
<<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>com.hua</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../microservicecloud/pom.xml</relativePath>
    </parent>


    <artifactId>microservicecloud-provider-dept-8001</artifactId>

    <dependencies>
        <!--引入自己定义的api通用包，可以使用Dept部门Entity-->
        <dependency>
            <groupId>com.hua</groupId>
            <artifactId>microservicecloud-api</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!--Junit测试-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
        <!--mysql驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!--阿里的连接池-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
        </dependency>
        <!--日志-->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
        </dependency>
        <!--mybatis的启动器-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <!--boot的jetty容器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
        <!--web的启动器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--test启动器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <!--热部署，修改后立即生效-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>

</project>
```

2）application.yml，第二步，写配置文件，配置端口号，mybatis和spring的配置

```yml
server:
  port: 8001
mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml  #mybatis配置文件所在路径
  type-aliases-package: com.hua.springcloud.entities     #所有Entity别名类所在包
  mapper-locations:
  - classpath:mybatis/mapper/**/*.xml        #mapper映射文件
spring:
  application:
    name: microservicecloud-dept
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource     #当前数据源操作类型
    driver-class-name: org.gjt.mm.mysql.Driver        #mysql驱动包
    url: jdbc:mysql://localhost:3306/cloudDB01        #数据库名称
    username: root
    password: 171911
    dbcp2:
      min-idle: 5            #连接池的最小连接数
      initial-size: 5       #初始化连接数
      max-total: 5        #最大连接数
      max-wait-millis: 200   #等待连接获取的最大超时时间
```

3）根据yml文件中的mybatis配置文件的位置来写mybatis.cfg.xml文件

注意：这个文件按道理讲是可以没有的，也没什么必要有，但为了实现全面的架构，所以保留

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!--开启全局的二级缓存 -->
        <setting name="cacheEnabled" value="true"/>
    </settings>
</configuration>
```

4）建立数据库

```java
CREATE TABLE `dept` (
  `deptno` bigint(20) NOT NULL AUTO_INCREMENT,
  `dname` varchar(60) DEFAULT NULL,
  `db_source` varchar(60) DEFAULT NULL,
  PRIMARY KEY (`deptno`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8;
```

5)建立对应的dao接口

```java
package com.hua.springcloud.dao;

import com.hua.springcloud.entites.Dept;
import org.apache.ibatis.annotations.Mapper;

import java.util.List;

/**
 * @Author lenovo
 * @Version 2018/10/31
 */
//注意springboot整合需要加这个注解，不要忘记
@Mapper
public interface DeptDao {

    //添加部门信息
    public boolean addDept(Dept dept);

    //根据部门id查找部门信息
    public Dept findById(Long id);

    //查询出所有部门信息
    public List<Dept> findAll();


}

```

6）根据yml文件中指定的mapper配置文件的地址来写mapper

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--命名空间应该是对应接口的包名+接口名 -->
<mapper namespace="com.hua.springcloud.dao.DeptDao">
    <select id="findById" resultType="com.hua.springcloud.entites.Dept" parameterType="Long">
        select deptno,dname,db_source FROM dept WHERE deptno=#{deptno};
    </select>

    <select id="findAll" resultType="com.hua.springcloud.entites.Dept">
        SELECT deptno,dname,db_source FROM dept;
    </select>

    <insert id="addDept" parameterType="com.hua.springcloud.entites.Dept">
        INSERT INTo dept(dname,db_source) VALUES (#{dname},DATABASE());
    </insert>
</mapper>
```

7）构建service层

有个service接口，对应有这个接口的实现类

```java
package com.hua.springcloud.service;

import com.hua.springcloud.entites.Dept;

import java.util.List;

/**
 * @Author lenovo
 * @Version 2018/10/31
 */
public interface DeptService {

    public boolean add(Dept dept);


    public Dept get(Long id);

    public List<Dept> list();

}
```

```java
package com.hua.springcloud.service;

import com.hua.springcloud.dao.DeptDao;
import com.hua.springcloud.entites.Dept;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * @Author lenovo
 * @Version 2018/11/1
 */
@Service
public class DeptServiceImpl implements DeptService {

    @Autowired
    private DeptDao dao;

    /*
     * 可以发现下面方法和dao层都差不多，而名字却差很多
     * 这是因为这样命名更符合controller里面的rest风格的业务逻辑
     */

    @Override
    public boolean add(Dept dept) {
        return dao.addDept(dept);
    }

    @Override
    public Dept get(Long id) {
        return dao.findById(id);
    }

    @Override
    public List<Dept> list() {
        return dao.findAll();
    }
}

```

8）构建controller层

```java
package com.hua.springcloud.controller;

/**
 * @Author lenovo
 * @Version 2018/11/1
 */

import com.hua.springcloud.entites.Dept;
import com.hua.springcloud.service.DeptService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * 因为前后端分离，我们只提供rest风格的接口，所以说返回到界面上的就是一个json
 * 字符串，所以说，我们可以在这直接加上@RestController注解或者在每个方法里面
 * 都加上@Responsebody
 */
@RestController
public class DeptController {

    @Autowired
    private DeptService service;

    /**
     * 注意，所谓的rest风格，深度的不懂，简单的就是add对应post
     * get就是 get
     * update就是put
     * delete就是delete
     * 然后对哪一种资源进行操作就是    /资源/add等
     * 还有
     * 下面这个@RequestMapping注解也可以直接写成@PostMapping
     */
    @RequestMapping(value="/dept/add",method = RequestMethod.POST)
    public boolean add(@RequestBody Dept dept){
        return service.add(dept);
    }

    @RequestMapping(value="/dept/get/{id}",method = RequestMethod.GET)
    public Dept get(@PathVariable("id")Long id){
        return service.get(id);
    }

    @RequestMapping(value = "/dept/list",method = RequestMethod.GET)
    public List<Dept> list(){
        return service.list();
    }

}
```

9）编写springboot的主启动类

注意这个启动类要放在较高的包下面

```java
package com.hua.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @Author lenovo
 * @Version 2018/11/1
 */
@SpringBootApplication
public class DeptProvider8001_App {
    
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8001_App.class,args);
    }
}
```

10）启动看成功否

![1541003447849](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1541003447849.png)

### 4.部门微服务的消费者

这个配置文件和上面套路差不多，所以说下面只写配置了，就不详细注解介绍了

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>com.hua</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../microservicecloud/pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-consumer-dept-80</artifactId>
    <description>部门微服务消费者</description>

    <dependencies>
        <!--api项目-->
        <dependency>
            <groupId>com.hua</groupId>
            <artifactId>microservicecloud-api</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!--web的启动器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--修改后立即生效，热部署-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
        </dependency>
    </dependencies>

</project>
```

application.yml

```yml
server:
  port: 8020
```

因为这边写的是消费方，需要从提供者那边获取服务，所以说，需要RestTemplate，这个东西吧，需要写一个配置类，加上注解，把他放到容器里面

```java
package com.hua.springcloud.cfgbeans;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

/**
 * @Author lenovo
 * @Version 2018/11/1
 */

/**
 * 只要加了这个注解，就等同于原来的配置文件
 */
@Configuration
public class ConfigBean {

    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

然后写一个controller

```java
package com.hua.springcloud.controller;

import com.hua.springcloud.entites.Dept;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import java.util.List;

/**
 * @Author lenovo
 * @Version 2018/11/1
 */
@RestController

public class DeptController_Consumer {

    private static final String REST_URL_PREFIX="http://localhost:8001";

    /**
     * RestTemplate是个什么呢？
     * 它就是如果你要对于那个请求的封装，给你弄一个模板任你使用
     * 如何使用呢？
     * (url,requestMap,ResponseBean.class)
     * 这三个参数分别代表REST请求地址、请求参数、HTTP响应转换被转换成的对象类型
     */
    @Autowired
    private RestTemplate template;

    @RequestMapping(value = "/consumer/dept/add")
    public boolean add(Dept dept){
        return template.postForObject(REST_URL_PREFIX+"/dept/add",dept,Boolean.class);
    }

    @RequestMapping(value="/consumer/dept/get/{id}")
    public Dept get(@PathVariable("id")Long id){
        return template.getForObject(REST_URL_PREFIX+"/dept/get/"+id,Dept.class);
    }

    @RequestMapping(value = "/consumer/dept/list")
    public List<Dept> list(){
        return template.getForObject(REST_URL_PREFIX+"/dept/list",List.class);
    }

}
```

写启动类

```java
package com.hua.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @Author lenovo
 * @Version 2018/11/1
 */
@SpringBootApplication
public class DeptConsumer80_App {

    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer80_App.class,args);
    }
}
```

# 学习Eureka

### 服务注册和发现

### Eureka是什么

#### 介绍

是Netfix的一个子模块，也是核心模块之一，Eureka是一个机遇REST的服务，用于定位服务，以实现云端中间层服务发现和故障转移，服务注册与发现对于微服务架构来说是非常重要的，有了服务发现与注册，主要需要服务的标识符，就可以访问到服务，而不需要修改服务调用的配置文件了。功能类似于dubbo的注册中心，比如Zookeeper

#### 原理

Eureka采用了C-S的设计架构，Eureka Server作为服务注册功能的服务器，它是注册中心

而系统的其他微服务，使用Eureka的客户端连接到Eureka Server并维持心跳连接。这样系统的维护人员就可以通过Eureka Server来监控系统中各个微服务是否正常运行。SpringCloud的一些其他模块（比如Zuul）就可以通过Eureka Server来发现系统中的其他微服务，并执行相关的逻辑

#### 包含

Eureka包含两个组件：Eureka Server和Eureka Client

Eureka Server提供服务注册服务

各个节点启动后，会在EurekaServer中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面上直观的看到

EurekaClient是一个Java客户端，用于简化EurekaServer的交互，客户端同时也具备一个内置的、使用轮询（round-robin）负载算法的负载均衡器，在应用启动后，将会像Eureka Server发送心跳（默认周期为30秒）。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒）

### 构建Eureka服务注册中心

引入cloud的新增一个新技术组件

基本有两步

​	1.新增一个相关的maven坐标

​	然后引入相关的依赖

​	2。在朱启动类上面，标注的启动该新组件技术的相关注解标签

​	@EnableXxxx

1.引入eureka

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>com.hua</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../microservicecloud/pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-eureka-7001</artifactId>
    <dependencies>
        <!--eureka-server服务端-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-netflix-eureka-server</artifactId>
        </dependency>
        <!--修改后立即生效，热部署-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>

</project>
```

2.编写yml配置文件

```yml
server:
  port: 7001
eureka:
  instance:
    hostname: localhost       #eureka服务端的实例名称
  client:
    register-with-eureka: false   #false表示不向注册中心注册自己
    fetch-registry: false         #false表示自己就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/          #设置与Eureka Server交互的地址查询服务和注册都需要依赖这个地址
```

3.编写启动类

```java
package com.hua.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

/**
 * @Author lenovo
 * @Version 2018/11/1
 */
@SpringBootApplication
@EnableEurekaServer
public class EurekaServer7001_App {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServer7001_App.class,args);
    }
}
```

4，启动之后，登录localhost:7001可以看到如下界面

![1541075818365](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1541075818365.png)

### 将上面创建的微服务提供者项目放入注册中心

修改的部分

1.在pom.xml里面增加下面的依赖

```xml
<!--将服务provider侧注册进eureka-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

2.在yml文件里面增加下面的配置

```yml
eureka:
  client: #客户端注册进eureka服务列表内
    service-url:
      defaultZone: http://localhost:7001/eureka
```

3.在启动类上加上@EnableEurekaClient注解

```java
package com.hua.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

/**
 * @Author lenovo
 * @Version 2018/11/1
 */
@SpringBootApplication
@EnableEurekaClient//服务启动后会注册进入eureka
public class DeptProvider8001_App {

    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8001_App.class,args);
    }
}
```

4.弄完之后启动项目，在注册中心会看到下面的东西

![1541079855553](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1541079855553.png)



可以看出这个application的名字对应的是yml里面配置的那个spring.application.name的名。

看见红的先别慌，那个是eureka的自我保护机制这个东西后面说

### 改进

actuator与注册微服务信息完善

1.主机名称：服务名称修改

![1541081214437](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1541081214437.png)

在yml加入配置

```yml
  instance:
    instance-id: microservicecloud-dept8001    #自定义服务器名称信息
```

2.访问路径可以显示ip地址

```yml
prefer-ip-address: true  #访问路径可以显示IP地址
```

3.点击这个服务可以显示相应的信息

1）在服务提供者的那个pom.xml文件里面添加依赖

```xml
<!--actuator监控信息完善-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

2）在总的父工程里面的pom.xml添加如下信息

```xml
<!--build是构建的意思-->
<build>
    <finalName>microservicecloud</finalName>
    <!--允许访问这个路径下的文件，访问的过滤开启-->
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
    <!--以$开始和$结尾的在上面那个配置文件的信息就能够读取-->
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-resources-plugin</artifactId>
            <configuration>
                <delimiters>
                    <delimit>$</delimit>
                </delimiters>
            </configuration>
        </plugin>
    </plugins>
</build>
```

3）.在服务调用者8001工程里面添加如下信息

```yml
info:
  app.name: hua-microservicecloud
  company.name: www.hua.com
  build.artifactId: $project.artifactId$
  build.version: $project.version$
```

### Eureka的自我保护机制

![1541088678576](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1541088678576.png)



###### 一句话：某时刻某一个微服务不可用了，eureka不会立刻清理，依旧会对该微服务的信息进行保存

默认情况下，如果EurekaServer在一定时间内没有接收到某个微服务实例的心跳，EurekaServer将会注销该实例（默认90秒）但是当网略分区故障发生时，微服务在与EurekaServer之间无法正常通信，以上行为可能变得非常危险了——因为微服务本身其实是健康的，此时本不应该注销这个微服务。Eureka通过“自我保护模式”来解决这个问题——当EurekaServer节点在短时间内丢失过多客户端时，不再删除服务注册表中的数据（也就是不会注销任何微服务）。当网略故障恢复后，该Eureka Server节点会自动退出自我保护模式

**在自我保护模式中，Eureka Server会保护服务注册表中的信息，不再注销任何服务实例。当它收到的心跳数重新恢复到阈值以上时，该Eureka Server节点就会自动退出自我保护模式。它的设计哲学就是宁可保留错误的服务注册信息，也不盲目注销任何可能健康的服务实例。一句话理解：好死不如赖活着**

这个自我保护机制可以通过在配置文件里配置开启还是关闭，时间也可以进行配置，但是一般没打毛病，最好别瞎鸡儿配



### Eureka的服务发现

这里不重要，就跟自己提供一个访问地址，里面都是我这个服务的信息一样。。。

就跟自己测自己一样，这里为了训练DiscoveryClinet的用法

在服务提供者的controller类里面添加如下代码

```java
@Autowired
private DiscoveryClient client;
 @RequestMapping(value = "/dept/discovery",method = RequestMethod.GET)
    public Object discovery(){
        List<String> list=client.getServices();
        System.out.println("************"+list);
        List<ServiceInstance> serviceInstanceList=client.getInstances("MiCROSERVICECLOUD-DEPT");

        for(ServiceInstance element:serviceInstanceList){
            System.out.println(element.getServiceId()+"\t"+element.getHost()+"\t"+element.getPort()+"\t"
            +element.getUri());
        }

        return this.client;

    }
```



2.在启动类里加入

```java
@EnableDiscoveryClient//服务发现
```

3.在服务消费者的controller里添加如下代码

```java
@RequestMapping(value = "/consumer/dept/sicovery")
public Object dicovery(){
    return template.getForObject(REST_URL_PREFIX+"dept/discovery",Object.class);
}
```

这样自己也能找到自己信息，然后服务消费者也能找到自己的信息



### Eureka的集群配置

这个很重要

就是弄一窝注册中心，一个废掉了，另一个还能跑

此处弄三个，分别是7001，7002，7003

1.首先，在自己机器上搞一个域名映射(自己欺骗一下自己)

```java
C:\Windows\System32\drivers\etc
这个地址下面的hosts文件加入下面的内容

127.0.0.1 eureka7001.com
127.0.0.1 eureka7002.com
127.0.0.1 eureka7003.com
```

2.新建两个项目7002，7003

把7001项目的pom.xml文件和主启动类都粘贴过去

这样的话，我们改了域名映射，所以以前的hostname就得改成对应的咱映射的那个名字

而且defaultZone也得改成别的两个地方的地址

``` yml
server:
  port: 7001
eureka:
  instance:
    hostname: eureka7001.com       #eureka服务端的实例名称
  client:
    register-with-eureka: false   #false表示不向注册中心注册自己
    fetch-registry: false         #false表示自己就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      #defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/          #设置与Eureka Server交互的地址查询服务和注册都需要依赖这个地址
      defaultZone: http://eureka7002.com:7002/eureka,http://eureka7003:7003/eureka
```

注意端口号不要忘记改

改完这些，，再改原来的服务提供者8001，把他注册进这三个注册中心

``` yml
eureka:
  client: #客户端注册进eureka服务列表内
    service-url:
       defaultZone: http://eureka7002.com:7002/eureka,http://eureka7002:7002,http://eureka7003:7003/eureka
```



![1541211259018](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1541211259018.png)



可以看出DS Replicas上面挂了两个注册中心。



# 学习Ribbon

### 是什么

**客户端** **负载均衡**的工具

简单的说，Ribbon是Netfix发布的开源项目，主要功能是提供**客户端的软件负载均衡*算法，将Netfix的中间层服务连接在一起，Ribbon客户端组件提供一系列完善的配置项如连接超市，重试等。简单的说，就是在配置文件中列出load Balancer（建成LB）后面所有的及其，Ribbon会自动的帮助你基于某种规则（如简单轮询，随即连接等）去连接这些及其，我们也很容器使用ribbon实现自定义的负载均衡算法

### 负载均衡是什么

在微服务或分布式集群中经常用的一种应用

负载均衡简单的说就是将用户的请求平摊的分配到多个服务上，从而达到系统的HA

常见的负载均衡有软件Nginx，LVS，硬件F5等

相应的在中间件，例如：dubbo和SpringCloud中均给我们提供了负载均衡，SPringcloud的负载均衡算法可以自定义

### 使用ribbon

跟上面一样，加依赖，加注解，完事

#### 事前准备

让服务消费者在注册中心中去拉去服务

在消费者项目的pom.xml文件中添加如下依赖

``` xml
<!--将这个弄进注册中心-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<!--Rbibbon相关-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-ribbon</artifactId>
</dependency>
```

2.修改yml文件

``` yml
eureka:
  client: #客户端注册进eureka服务列表内
    service-url:
       defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
    register-with-eureka: false
```

注意：这里把注册进入eureka置为false，而拉去服务没有设置，那么就默认为true

3.在启动类上加上eureka客户端的注解

```java
@EnableEurekaClient
```

4.在配置类的返回值为RestTemplate的方法上，加上@LoadBalanced注解

``` java
@Bean
@LoadBalanced
public RestTemplate getRestTemplate(){
    return new RestTemplate();
}
```



然后把服务提供者复制两份，并且分配给两个的数据库

然后说启动之后，发现它默认的负载均衡策略是轮询

### Ribbon自定义

修改服务消费者项目

1.在启动类上加上如下注解

这个注解的作用是在启动该微服务的时候就能去加载我们的自定义Ribbon配置类，从而使配置生效，形如：

2.然后创建一个配置类，里面的方法的返回值是

官方文档明确给出了警告：

这个自定义配置类不能放在@ComponentScan所扫描的当前包下以及子包下，否则我们自定义的这个配置类就会被所有的Ribbon客户端所共享，也就是说我们达不到特殊化定制的目的了（而我们的我们启动类就是有@SpringCloudApplication）

``` java
package com.hua.myRule;

import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.RandomRule;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @Author lenovo
 * @Version 2018/11/3
 */
@Configuration
public class MySelfRule {

    @Bean
    public IRule myRule(){
        return new RandomRule();
    }

}
```

这个返回值可以是自己定义的来实现IRule接口的类，也可以是官方给的那些实现的算法，还可以是AbstractLoadBalanceRule抽象类的实现类



# 学习Feign

就是人们都比较熟悉面向结构编程，所以说，如果说服务的调用不是去直接的调用那个服务，而是在service层获取，然后调用service接口不就更好了吗

根据8020项目，新建一个feign项目



1.修改api项目的pom.xml文件，加入feign相关依赖

为什么要改api项目呢？

因为feign是要生成一个可以调用的service接口，然后在别的项目的controller里面调用那个service，所以说，我这边可能还有别的地方要用这个，所以说把这个放到api工程里面

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stater-feign</artifactId>
</dependency>
```

2.在api项目新建一个service接口，引入如下代码

注意：下面的代码实际上

```java
package com.hua.springcloud.service;

import com.hua.springcloud.entites.Dept;
import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;

import java.util.List;

/**
 * @Author lenovo
 * @Version 2018/11/3
 */
@FeignClient(value = "MICROSERVICECLOUD-DEPT")
public interface DeptClientService {

    @RequestMapping(value = "/dept/add")
    public boolean add(Dept dept);

    @RequestMapping(value="/dept/get/{id}")
    public Dept get(@PathVariable("id")Long id);

    @RequestMapping(value = "/dept/list")
    public List<Dept> list();


}

```

3.修改feign项目里面的controller

也就是把以前的是用RestTemplate模板来实现的服务的调用，而用调用service的方法来实现

```java
package com.hua.springcloud.controller;

import com.hua.springcloud.entites.Dept;
import com.hua.springcloud.service.DeptClientService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import java.util.List;

/**
 * @Author lenovo
 * @Version 2018/11/1
 */
@RestController
public class DeptController_Consumer {

    @Autowired
    private DeptClientService service;


    @RequestMapping(value = "/consumer/dept/add")
    public boolean add(Dept dept){
        return this.service.add(dept);
    }

    @RequestMapping(value="/consumer/dept/get/{id}")
    public Dept get(@PathVariable("id")Long id){
        return this.service.get(id);
    }

    @RequestMapping(value = "/consumer/dept/list")
    public List<Dept> list(){
        return this.service.list();

    }


}
```

4.在主启动类上加上注解（这是套路，引入springcloud的新技术，1，引入依赖，2，启动类上加注解）

```java
@EnableFeignClients(basePackages = "com.hua.springcloud")
@ComponentScan(value = "com.hua.springcloud")
```



feign里面的过滤器，，实现ZuulFilter

要写。。过滤器类型，过滤器级别，过滤器是否生效，过滤器的运行逻辑

运行逻辑中获取request对象的方法：RequestContext ctx=RequestContext.getCurrentContext();



# 学习Hystrix

### 介绍

断路器，程序出异常了，微服务长时间调用不恰当然后该怎么办

服务雪崩

多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其它的微服务，这就是所谓的“扇出”。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的“雪崩效应”

Hystrix是一个用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性

“断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），向调用方返回一个符合预期的，可处理的备选响应（FallBack），而不是长时间的等待或者抛出调用方法无法处理的异常，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩



### 使用

1.copy一个上面的8001项目，然后在pom.xml文件中添加如下依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```

2.在启动类上加注解

```java
@EnableCircuitBreaker
```

3.修改controller代码

```java
package com.hua.springcloud.controller;

/**
 * @Author lenovo
 * @Version 2018/11/1
 */

import com.hua.springcloud.entites.Dept;
import com.hua.springcloud.service.DeptService;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * 因为前后端分离，我们只提供rest风格的接口，所以说返回到界面上的就是一个json
 * 字符串，所以说，我们可以在这直接加上@RestController注解或者在每个方法里面
 * 都加上@Responsebody
 */
@RestController
public class DeptController {

    @Autowired
    private DeptService service;



    @RequestMapping(value="/dept/get/{id}",method = RequestMethod.GET)
    @HystrixCommand(fallbackMethod = "processHystrix_Get")
    public Dept get(@PathVariable("id")Long id){
        Dept dept=this.service.get(id);
        if(null==dept){
            throw new RuntimeException("该ID："+id+"没有对应的信息");
        }

        return dept;
    }


    public Dept processHystrix_Get(@PathVariable Long id){
        return new Dept().setDeptno(id).setDname("该ID"+id+"没有对应的信息-@HystrixCommand")
                .setDb_source("no this database in MySQL");
    }




}
```

看一看出上面用@HystrixCommand注解指定了一个跟上面返回值相同，并且参数也相同的方法，里面定义了异常的处理机制，这差不多就是那个啥子，服务熔断

可以看出这样的话岂不是每写一个方法就要写一个对应的异常的处理的。。东西，，所以说，这个东西吧，，还得改进

### 服务降级

服务降级是在服务调用者，，，上实现的。。

所谓降级，一般是从整体负荷考虑。就是当某个服务熔断之后，服务器将不再被调用

此时客户端可以自己准备一个本地的fallback毁掉，返回一个缺省值

这样做，虽然服务水平下降，但好歹可用，比直接挂掉要强

所以说在api项目里写一个，这样的话就把所有的熔断机制都抽取出来了

```java
package com.hua.springcloud.service;

import com.hua.springcloud.entites.Dept;
import feign.hystrix.FallbackFactory;
import org.springframework.stereotype.Component;

import java.util.List;

/**
 * @Author lenovo
 * @Version 2018/11/5
 */
@Component//前面别忘记加@Component注解
public class DeptClientServiceFallbackFactory implements FallbackFactory<DeptClientService> {
    @Override
    public DeptClientService create(Throwable throwable) {
        return new DeptClientService() {
            @Override
            public boolean add(Dept dept) {
                return false;
            }

            @Override
            public Dept get(Long id) {
                return null;
            }

            @Override
            public List<Dept> list() {
                return null;
            }
        };
    }
}
```

然后在DeptClientService的注解里指定这个类

```java
//@FeignClient(value = "MICROSERVICECLOUD-DEPT")
@FeignClient(value = "MICROSERVICECLOUD-DEPT",fallbackFactory = DeptClientServiceFallbackFactory.class)
```

 然后修改feign项目里面的yml文件，加入如下内容

```java
feign:
  hystrix:
    enabled: true
```

这他妈的根本看不懂好吗，，，，有个鸟用啊，这个配置，，，，



### 服务监控HystrixDashboard

对微服务的调用状况、压力状况做一个图形化展现

#### 使用

弄一个项目，跟上面一样，农一个springcloud的项目，然后加pom.xml依赖，加注解，改yml，使用

pom.xml

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>


<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
</dependency>
```

application.yml

```yml

server:
  port: 9001
```

在朱启动类上加入@EnableHystrixDashBoard

```java
package com.hua.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;

/**
 * @Author lenovo
 * @Version 2018/11/6
 */
@SpringBootApplication
@EnableHystrixDashboard
public class DeptConsumerDashBoard {

    public static void main(String[] args) {
        SpringApplication.run(DeptConsumerDashBoard.class,args);
    }
}
```

然后在微服务的提供者的pom.xml文件中，添加下面的依赖

```xml
<!--actuator监控信息完善-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

看到这，，，你会注意，，上面已经加过这个依赖了，但是没什么用，到这你就知道是什么用了

然后访问locahost:9001/hystix，会看到下面

![1541487149808](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1541487149808.png)

那么这个玩意，怎么用呢，然后把，，，就启动我们的三个注册中心和上面写好的hystrix熔断的那个项目

然后在浏览器上访问这个的话是localhost:8001/hystrix.stream，你会发现它在不停的加载一些数据，而这些数据看起来又比较麻烦，所以说，需要一个图形界面。。那么就要用到刚刚那个hysrtix的主页面了

![1541489023329](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1541489023329.png)



#### 主页面，

上面输入http://ip:端口/hystrix.stream

Delay：该参数用来控制服务器上轮询监控信息的延迟时间。默认为2000毫秒，可以通过配置该属性来降低客户端的网略和CPU消耗

Title：该参数对应了头部标题Hystrix Stream之后的内容，默认会使用具体监控实例的URL，可以通过配置该信息来展示更合适的标题

三个东西输入完之后，进入下面的界面

![1541489367429](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1541489367429.png)

#### 界面介绍

7色，1圈，1线

1圈：就是一个实心圆。共有两种含义。它通过颜色的变化代表了实例的健康程度，它的健康度从绿色《黄色《橙色《红色递减。

该实心圆除了颜色的变化之外，它的大小也会根据实例的请求流量发生变化。流量越大该实心圆越大。所以通过该实心圆的展示，就可以在大量的实例中快速的发现故障实例和高压力实例

1线：曲线。用来记录2分钟内流量的相对变化，可以通过它来观察到流量的上升和下降趋势。

![1541489821865](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1541489821865.png)

# 学习Zuul

### 是什么

Zuul包含了对请求的路由和过滤两个最主要的功能：

其中路由功能负责将外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础而过滤器功能则负责对请求的处理过程进行干预，是实现请求校验、服务聚合等功能的基础Zuul和Eureka进行聚合，将Zuul自身注册为Eureka服务治理下的应用，同时从Eureka中获得其他微服务的信息，也即以后的访问微服务都是通过Zuul跳转后获得。

注意:Zuul服务器最终还是会注册进Eureka

提供   ：路由+代理+过滤三大功能（上面说俩，这又说三，，，有毒）

### 使用

新建一个项目，里面引入正常的服务的依赖，多加上zuul的依赖

pom.xml

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zuul</artifactId>
</dependency>
```

yml配置文件

```yml
server:
  port: 9527
spring:
  application:
    name: microservicecloud-zuul-gateway
eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka

  instance:
    instance-id: localhost
    prefer-ip-address: true
info:
  app.name: hua-microcloud
  company-name: www.hua.com
  build-artifactId: $project.artifactId$
  build.version: $project.version$
```

主启动类

```java
package com.hua.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

/**
 * @Author lenovo
 * @Version 2018/11/6
 */
@SpringBootApplication
@EnableZuulProxy
public class Zuul_9527_StartSpringCloud {

    public static void main(String[] args) {
        SpringApplication.run(Zuul_9527_StartSpringCloud.class,args);
    }
}
```



启动测试：三个注册中心集群，8001服务提供者，刚创建的这个路由

不用路由：http://localhost:8001/dept/get/1

这个localhost指的是8001服务地址

用路由：http://localhost:9527/microservicecloud-dept/dept/get/1

这个localhost指的是服务路由的地址（因为，，都是在我电脑上，所以现在都是localhost），microservicecloud-dept是服务的名字，后面那个就是，，，请求的东西。。。



### 设置虚拟微服务名

上面的使用方式挺好，但是有个问题就是把我们的真实的微服务的名字暴露出去了，，，但不知道有什么不好的，，

在yml文件中可以加入如下的配置来把这个真实的微服务的名字跟相关的虚拟的名字做一个映射

```yml
zuul:
  routes:
    mydept.serviceId: microservicecloud-dept
    mydept.path: /mydept/**
```

还可以写下面两种形式的

```yml
zuul:
  routes:
    mydept:
    	path: /mydept/**
    	serviceId: microservicecloud-dept
```

```yml
zuul:
  routes:
   	microservicecloud-dept: /mydept/**
```

第一种是zuul的完整配置方法，第二个是简化版，服务id加映射路径

他会默认的给你一种，，配置

```yml
microservicecloud-dept: /microservicecloud-dept/**
```

这种配置会默认生效，，，所以说，，，，要把之前的禁用掉，，看下面

改完之后启动测试，地址：http://localhost:9527/mydept/dept/get/1

这样改完之后发现原来的还能用，所以说要把原来真实的微服务的名字禁用掉

```yml
zuul:
  ignored-services: microservicecloud-dept
```

那么，如果想一次性禁用多个，，，微服务呢

```yml
zuul:
  ignored-services: "*"
```

这样的话就可以把所有用真名访问给禁用了

### 设置统一公共前缀

```yml
zuul:
  prefix: "hua"
```

# 学习SpringCloudConfig

### 是什么

SpringCloud Config为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为各个不同微服务应用的所有环境提供了一个中心化的外部配置

### 怎么用

SpringCloud Config分为服务端和客户端两部分

服务端也称为分布式配置中心，它是一个独立的微服务应用，用来连接配置服务器并为客户端提供获取配置信息，加密/解密信息等访问接口

客户端则是通过指定的配置中心来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息配置服务器默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过git客户端工具来方便的管理和访问配置内容



### 服务端使用

#### 学习使用前的准备

首先，在github上面建一个项目，然后在本地上clone下来，然后添加一个application.yml文件如下所示，然后提交到github上（模拟，运维工程师给我们弄的配置文件，）

```yml
spring:
  profiles:
    active:
    - dev
---

  spring:
    profiles: dev #开发环境
    application:
      name: microservice-config-hua-dev

---
spring:
  profiles: test  #测试环境
  application:
    name: microservicecloud-config-hua-test
```



#### 正儿八经的使用

还是原来的那几步-创建项目-加依赖-改yml-加注解

创建项目，引入正常的springcloud项目应该引入的那些依赖

pom.xml文件中加上下面的依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

yml

```yml
server:
  port: 3344
spring:
  application:
    name: microservicecloud-config
  cloud:
    config:
      server:
        git:
          uri: https://github.com/HRedL/microserviecloud-config.git
```



主启动类加上@EnableConfigServer

```java
package com.hua.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

/**
 * @Author lenovo
 * @Version 2018/11/7
 */
@SpringBootApplication
@EnableConfigServer
public class Config_3344_StartSpringCloudApp {


    public static void main(String[] args) {
        SpringApplication.run(Config_3344_StartSpringCloudApp.class,args);
    }
}
```



启动项目

![1541557557118](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1541557557118.png)

访问地址localhost:3344/application-dev.yml

意思就是访问application.yml里面的dev的那一块，，，这是springboot里面的内容，，可以回去看看







### 客户端使用

#### 使用前准备

先在文件下建立microservercloud-config-client.yml，然后提交该文件到github上

```yml
spring:
  profiles:
    active: 
    - dev
---
server:
  port: 8201
  
spring:
  profiles: dev #开发环境
  application:
    name: microservice-config-hua-dev
  
---
server:
  port: 8202
spring:
  profiles: test  #测试环境
  application:
    name: microservicecloud-config-hua-test
```

#### 正儿八经的使用

先建立一个项目，3355

引入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

引入，，bootstrap.yml，，，这个东西是什么，，，介绍一下

是系统级的配置文件，优先级更高





# 整个教程跑下来未解决的问题：

1.上面的热部署就一直没管用过（在idea中）

2.上面点击这个服务可以显示相应的信息中的$玩意不管用88



# 补充

可以在类上标注@DefaultProperties（defaultFallback=“defaultFallback”），设置这个controller下面的全部的服务降级处理。。只需要在方法上。。。标注@HystrixCommand



最新版的feign的starter的名字叫做openfeign





