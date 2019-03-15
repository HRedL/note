# 1、什么是Springboot

用一些固定的方式来构建生产级别的spring应用，Springboot推崇约定大于配置的方式乙方于你能够尽可能快速的启动并运行程序

其实人们把Spring Boot称为脚手架，其最主要的就是帮我们快速的构建庞大的spring项目，并且尽可能减少一切xml配置，做到开箱即用，迅速上手，让我们更关注业务而不是逻辑



**总结：这个东西就是让我们开发变得简单，不需要进行大量的配置才能写业务逻辑。从一定意义上解决了java比较臃肿的缺点**

# 2、入门Springboot

可以按照官方文档自己去一步步的跟着来（但好像没那么容易）

此处以搭建一个springmvc的web项目为例

## 2.1SpringBoot的HelloWorld编写

##### ①建立一个简单的maven项目

##### ②在pom.xml文件里引入依赖

```xml
  <!--引用Springboot父项目，其中规定了各种依赖的版本-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.6.RELEASE</version>
    </parent>

    <dependencies>
        <!--引入一个web项目的启动器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```

##### ③建立一个HelloWorld类

```java
package com.hua;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
//此处的注解一定要加
@SpringBootApplication
public class HelloWorld {

    public static void main(String[] args) {
        //第一个参数是当前类，第二个参数是main方法参数
        SpringApplication.run(HelloWorld.class,args);
    }
}

```

##### ④可以写springmvc的转发逻辑了

```java
package com.hua.Controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class HelloController {

    //加入responseBody注解之后，方法的返回值会打印在目标页面上
    //此处只是演示helloWorld，所以写的简单一些
    @ResponseBody
    @RequestMapping("toHelloPage")
    public String toHelloPage(){
        return "hello";
    }
}
```

##### ⑤启动项目

打开刚刚写好的HelloWorld类，找到main方法运行即可,运行结果如下

==》控制台

![QQ图片20181028101925](C:\Users\lenovo\Desktop\Springboot学习\QQ图片20181028101925.png)

==》页面

![1540693480783](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1540693480783.png)





## 2.2 SpringBoot的HelloWorld中需要注意的点

①注意Controller要放在编写的HelloWorld所在包或者其子包下（因为他自动扫描的包默认是在这个类所在的包下）

②新版本的spring里面加入了很方便的restful风格的映射

```
  即@GetMapping("toHelloPage")这个注解就相当于下面那个注解，而对应的也有Delete、Post、Put
  
 @RequestMapping(value = "toHelloPage", method = RequestMethod.GET)
```

③其中还可以在类上用@RestController来代替@Controller和@Responsebody（这个注解注解到类上，说明这个类的方法都是把返回值直接打印在页面上）

# 3.注解代替xml

现在流行的就是注解开发，xml是很久很久之前的遗留产物，而我们学习的ssm，是使用的注解和xml混合的使用，而这种写代码的方式也快要过时，所以说，此处学习用java写一个配置类来代替xml

### 3.1体会注解开发

首先，如果想进行下面的配置类学习，先了解一下这几个注解，详细会写在下面的代码的注解部分，此处先了解一下

**@Configuration：声明一个类作为配置类，代替文件**

**@Bean：声明在方法上，将方法的返回值加入Bean容器，代替<bean>标签**

**@Value：属性注入**

**@PropertySource：指定外部属性文件**

我们接下来就是用java配置的方式来写为我们的数据库连接搞一个配置类试试先

①在pom.xml引入c3p0的依赖

```xml
<!-- https://mvnrepository.com/artifact/com.mchange/c3p0 -->
<dependency>
    <groupId>com.mchange</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.5.2</version>
</dependency>
```

②写一个数据库的配置文件db.properties

```properties
mysql.driverClass=com.mysql.jdbc.Driver
mysql.url=jdbc:mysql://localhost:3306/database?springboot
mysql.userName=root
mysql.password=171911
```

③写一个配置类

```java
package com.hua.conf;

import com.mchange.v2.c3p0.ComboPooledDataSource;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;

import javax.sql.DataSource;

/**
 * 其中@Configuration是标注此类是一个配置类（配置类就相当于原来的那个配置文件）
 * @PropertySource是标注在该类中引入的外部配置文件的位置
 */
@PropertySource("classpath:db.properties")
@Configuration
public class DbConf {
    /*
    @Value注解加上表达式，为该属性赋予外部配置文件中的相应的值
    */
    
    @Value("${mysql.userName}")
    private String userName;

    @Value("${mysql.password}")
    private String password;

    @Value("${mysql.url}")
    private String url;

    @Value("${mysql.driverClass}")
    private String driverClass;

    /*
    @Bean的效果是把这个方法的返回值放到容器中去，此处也就是把dataSource放到了容器中去，所以说
    就可以在其他类中自动注入这个dataSource了
    */
    
    @Bean
    public DataSource getDataSource()throws Exception{
        ComboPooledDataSource dataSource=new ComboPooledDataSource();
        dataSource.setJdbcUrl(url);
        dataSource.setUser(userName);
        dataSource.setPassword(password);
        dataSource.setDriverClass(driverClass);

        return dataSource;


    }

}
```

④写一个类，自动注入dataSource进行测试，然后debug找到自动注入的那个，看看属性是否存在

![1540707973729](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1540707973729.png)

### 3.2更加完美的注解开发

#### 1.如果在多处都能用到这个对象

独立的写成一个类

①写一个application.properties（注意，文件名必须为这个，当然你也可以写yml配置文件，关于yml详情看后面的补充），内容和前面的db.properties一致（注意，这个配置文件就是springboot的去全局配置文件，有什么配置就写在这里面就行了，在别的地方引用）

②把对应的根跟数据库连接的信息提取出来

```java
package com.hua.conf;

import org.springframework.boot.context.properties.ConfigurationProperties;
/**
 * @ConfigurationProperties 注解直接标注在类上，其中的prefix指定一个前缀，他会自动匹配
 * 配置文件中前缀为mysql的配置并且把对应的值赋给属性
 */
@ConfigurationProperties(prefix = "mysql")
public class Db {

    private String user;

    private String password;

    private String url;

    private String driverClass;

	//getter和setter
    
}
```

③此处有多种方法来用上面创建好的类（只要用@EnableConfigurationProperties标注并且指定上面那个类就能使用）

```java

package com.hua.conf;

import com.mchange.v2.c3p0.ComboPooledDataSource;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;



/**
 * @EnableConfigurationProperties 这个注解就是如果在这个上面就能用刚刚写好的那个
 */
@EnableConfigurationProperties(Db.class)
@Configuration
public class DbConf {
    
    //第一种，在方法参数里写上
    //第二种，用@Autowire标注
    //第三种，构造函数注入

    @Bean
    public DataSource getDataSource(Db db) throws Exception{
        ComboPooledDataSource dataSource=new ComboPooledDataSource();
        dataSource.setJdbcUrl(db.getUrl());
        dataSource.setUser(db.getUser());
        dataSource.setPassword(db.getPassword());
        dataSource.setDriverClass(db.getDriverClass());

        return dataSource;
    }

}
```

#### 2.若果这个对象只使用一次

如果只使用一次，那么就可以直接使用SPringboot帮你自动的匹配

那么只需要在方法上加上一个注解即可

```java
@ConfigurationProperties(prefix = "mysql")
@Bean
public DataSource getDataSource() throws Exception{
    ComboPooledDataSource dataSource=new ComboPooledDataSource();
    return dataSource;
}
```

这样就可以全部获得

![1540710657597](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1540710657597.png)

3.@value的方式获取，，和@ConfigurationProperties获取的区别

![1540710734853](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1540710734853.png)

这是官方文档上的一些对比（翻译之后），官方文档推荐使用@ConfigurationProperties注解



# 注意：一般情况都不要我们配置任何东西，但是万一要配置，，就按照如上所说的方法进行配置

# 4.@SpringBootApplication

### 1.查看这个注解的源码

![1540712557878](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1540712557878.png)

第一个点进去之后发现其实这个注解也就是标注这是一个配置类，和@Configuration注解的作用相同，所以说这个注解的意思就是把当前类编程一个配置类（注意：虽然说作用相同，但平常别用这玩意）

第二个注解的大体意思是，**根据你当前所引入的依赖，去猜测**你想要干些什么事，也就给你配置好什么spring环境

第三个注解的作用就是跟< context:component-scan >配置差不多，就是设置自动扫描包的。如果没设置自动扫描的包，默认是这个注解标注的类的所在的包下面的那个包



### 2.解释

上面所说的根据依赖去猜测你想干什么事，是怎么猜的呢？

它会根据你在pom.xml引入的starter来确定你要干什么事，也就是说你想要干什么就在pom.xml写上什么starter。那么是不是我们引入了这个东西，他把所有配置都给你直接生效了呢？

并不是，它底层会采用@ConditionalOnXXX和@ConditionalOnMissingXXX方法来决定生效的条件

# 5.静态文件

在说静态文件之前，要先说明一个事情，就是SPringboot整个架构里面是打包成一个jar包，所以就没有webapp这个文件夹，所以说资源文件应该写在哪呢，查看源码，发现

```xml
1）在maven里引入webjars（百度搜索）
其实就是把jqeury等文件打成了jar包的方式
2）/**  访问当前项目的任何资源
classpath：/META-INF/resources/
Classpath:/resources/
Classpath:/static/
Classpath:/public/
"/"当前项目的根路径

3)欢迎界面：静态资源文件夹下的index.html页面，被/**映射
会扫描所有静态资源文件夹下面的所有index.html里面

4)图标:所有的**/favicon.ico都是在静态资源文件下找
```





这个时候，有没有疑问？我怎么写jsp页面呢，好吧，，不写jsp，以后都前后端分离了，，后端只写什么接口，然后前端拿这些接口去渲染页面



# 6.拦截器

和springmvc中不一样，这个拦截器只需要写一个类来实现WebMvcConfigurer接口，其中有很多方法其实，比如视图解析器什么的

```java
public class MvcConfig implements WebMvcConfigurer{

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new HandlerInterceptor() {
            private final Logger logger= LoggerFactory.getLogger(this.getClass());
            @Override
            public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
                logger.info("preHandle is running");
                return false;
            }

            @Override
            public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
                logger.info("postHandle is running");
            }

            @Override
            public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
                logger.info("afterCompletion is running");
            }
        }).addPathPatterns("/**").excludePathPatterns("");
    }
}
```

# 7 jbdc相关配置和事务

在pom.xml文件中引入jdbc的starter，和mysql的驱动

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
	 <groupId>mysql</groupId>
     <artifactId>mysql-connector-java</artifactId>
 </dependency>
```

在项目里面想使用事务的话直接使用注解@Transactional就行了，，注意，引入这个注解之后，查看依赖还可以发现它默认它引入了了数据库连接池（HikariCP，目前性能最好的数据库连接池）

![1540727051806](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1540727051806.png)

所以说，引入jdbc的starter之后只需要指定他的四大参数即可，，数据库连接池什么玩意的都给你弄好了

，所以，配置四大参数

```properties
spring.datasource.username=
spring.datasource.password=
spring.datasource.driver-class-name=
spring.datasource.url=
```

，，，你会发现这种方式要写一窝spring.datasource有点烦，所以说一般写springboot的项目，都喜欢用yml配置文件

所以说yml语法就是疯狂运用对齐方式和空格，

```yml
spring:
  datasource:
    username:
    driver-class-name:
    password:
    url: 
```

tip：在用springboot的时候其实可以省略driverClass不写，因为，，，他会根据url来推断你用的哪个。。。贼tm方便

然后如果你不想改它默认的连接池的参数就不用配，jdbc就弄好了。。。就可以用了

# 8.Mybatis的配置

1.首先，先回顾一下在ssm中mybatis有哪些配置

①需要写一个mybatis的配置文件mybatis-config.xml

②需要在spring的配置文件里配置sqlSessionFactoryBean

③还要配置mapper接口的扫描器

2.配置mybatis

①在pom.xml中引入mybatis的starter（注意spring的官网并没有帮mybatis写好starter，mybaits自己写了个，所以要找这个starter要去mybatis官网上去找）

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>
```

②在主配置文件中加入别名扫描的包

```yml
mybatis:
  type-aliases-package: com.hua.entity
```

③在启动类上加上注解，mapper接口扫描器

```java
package com.hua;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@MapperScan("com.hua.mapper")
@SpringBootApplication
public class HelloWorld {

    public static void main(String[] args) {
        //第一个参数是当前类，第二个参数是main方法参数
        SpringApplication.run(HelloWorld.class,args);
    }
}
```

3.配置通用mapper

①在pom.xml文件中加入下面的starter

```xml
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-spring-boot-starter</artifactId>
    <version>1.2.3</version>
</dependency>
```

②最好是将启动类上面的注解换一个

```java
package com.hua;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import tk.mybatis.spring.annotation.MapperScan;
/*
	这个注解和上面的@MapperScan不是一个包里的
*/
@MapperScan
@SpringBootApplication
public class HelloWorld {

    public static void main(String[] args) {
        //第一个参数是当前类，第二个参数是main方法参数
        SpringApplication.run(HelloWorld.class,args);
    }
}
```

# 9 thymeleaf

Springboot并不推荐使用jsp，但是支持一些模板引擎

### 模板引擎

FreeMaker（比较火）

Thymeleaf（这个比较新）

Mustache

### Thymeleaf的优势

动静结合等

springboot也写了thymeleaf的启动器

### 用法

①首先，引入thymeleaf的starter

``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

②然后thymeleaf的文件默认放到templates下

他的东西正好是html