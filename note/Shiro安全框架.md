Shiro安全框架

md，气死，，，写了半天的笔记，，没保存，草



# 1.Shiro简介

Apache Shiro是一个强大而灵活的开源安全框架，它干净利落地处理身份认证，授权，企业会话管理和加密

验证用户来核实他们的身份

对用户执行访问控制，如：

​	判断用户是否被分配了一个确定该的安全角色

​	判断用户是否被允许做某事

在任何环境下使用Session API，及时没有Web或EJB容器

在身份验证，访问控制期间或在会话的生命周期，对事件作出反应。

聚集一个或多个用户安全数据的数据源，并作为一个单一的符合用户“视图“。

启用单点登录（SSO）功能

# 2.为啥学习Shiro

spring security要依赖于spring，并且比较复杂，学习曲线比较高。

shiro比较简单，而且shiro比较独立，既可以在javase中使用，也可以在javaee中使用，并且在分布式集群环境下也可以使用。

# 3.Shiro的功能

#### ①Authentication认证

验证用户是否合法，也就是登录

#### ②Authorization授权

授予谁具有访问某些资源的权限

#### ③Session Management会话管理

用户登录后的用户信息通过Session Management来进行管理，不管是在什么应用中

#### ④Cryptography加密

提供了常见的一些加密算法，使得在应用中可以很方便的实现数据安全。并且使用最便捷

#### ⑤Web support：web应用程序支持

shiro可以很方便的集成到web应用程序中

#### ⑥Caching缓存

shiro提供了对缓存的支持，支持多种缓存架构；如：ehcache还支持缓存数据库（例如redis）

#### ⑦Concurrency并发支持

#### ⑧Testing测试

测试支持的存在来帮助你编写单元测试和集成测试，并确保你的能够和预期的一样安全

#### ⑨RunAs

支持一个用户在允许的前提下使用另外一个身份登录

#### ⑩RememberMe记住我

在会话中记住用户的身份，所以他们只需要在强制时候登录

# 4.Shiro架构

#### ①Subject

Subject实质上是一个当前执行用户的特定的安全“视图”，鉴于“User”一词通常意味着一个人，而一个subject可以是一个人，但它还可以代表第三方服务，daemon account，cron，job

所有subject实例都被绑定到一个SecurityManager上，当你与一个Subject交互时，那些交互作用转化为与SecurityManager交互的特定的subject交互作用

#### ②SecurityManager

SecurityManager是Shiro的心脏，并作为一种“保护伞”对象来协调内部的安全组件共同构成一个对象图。然而，一旦SecurityManager和它的内置对象图已经配置给了一个应用程序，那么它单独留下来，且应用程序开发人员几乎使用它们所有的事件来处理Subject API

当你正与一个subject进行交互时，实质上是幕后的SecurityManager处理所有繁重的subject安全操作，这反映在上面的基本流程图

#### ③Authenticator认证器

负责验证用户的身份

#### ④Authorizer授权器

负责为合法的用户指定其权限，控制用户可以访问哪些资源

#### ⑤Realms

域，用户通过shiro完成相关的安全工作，shiro是不会去维护数据信息的，在shiro的工作过程中，数据的查询和获取工作室通过Relam从不同的数据源来获取的。Realm可以获取数据库信息，文本信息等。在shiro中可以有一个realm也可以有多个





# 5.用户认证Authentication

## 5.1介绍

验证用户是否合法

需要提交身份和凭证给shiro

### 5.2Principals用户身份信息

是subject的标识属性，能够唯一标识Subject

如：电话号码、电子邮箱、身份证号码等

### 5.3Credentials凭证

是只被subject知道的秘密值，可以是密码，也可以是数字证书等



principal和credentials最常见的组合：用户名/密码，在shiro中通常使用username/passwordToken来指定身份和凭证信息

## 5.4代码

### ①.引入maven依赖

```xml
<dependencies>
    <dependency>
        <groupId>commons-beanutils</groupId>
        <artifactId>commons-beanutils</artifactId>
        <version>1.9.2</version>
    </dependency>
    <dependency>
        <groupId>commons-logging</groupId>
        <artifactId>commons-logging</artifactId>
        <version>1.2</version>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-all</artifactId>
        <version>1.2.3</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.7.7</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.7.5</version>
    </dependency>
</dependencies>
```

### ②.编写shiro的数据文件-配置

```ini
[users]
zhangsan=1111
lisi=1111
```





### ③.配置log4j日志

```properties
log4j.rootLogger=debug,stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] -%m%n
```

### ④.编码测试

```java
package com.hua.shiro;

import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.config.IniSecurityManagerFactory;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.subject.Subject;
import org.apache.shiro.util.Factory;

/**
 * 完成用户认证功能
 */
public class AuthenticationDemo {

    public static void main(String[] args) {
        //1.创建SecurityManager工厂
        Factory<SecurityManager> fatory=new IniSecurityManagerFactory("classpath:shiro.ini");
        //2.通过SecurityManager工厂获取SecurityManager的实例
        SecurityManager securityManager=fatory.getInstance();
        //3.将SecurityManager对象设置到运行环境中
        SecurityUtils.setSecurityManager(securityManager);
        //4.通过SecurityUtils获取主体subject
        Subject subject=SecurityUtils.getSubject();


        //5.假如用户名、密码为zhangsan和111，这个表示用户登录时输入的信息
        //而shiro.ini中的用户名和密码相当于数据库中数据
        UsernamePasswordToken token=new UsernamePasswordToken("zhangsan","1111");

        //进行用户身份验证
        subject.login(token);
        //通过subject来判断用户是否通过验证
        if(subject.isAuthenticated()){
            System.out.println("用户登录成功");
        }else{
            System.out.println("用户名或密码不正确");
        }
    }

}
```



## 5.5常见的异常信息及处理

idea中查看层次结构快捷键ctrl+H

因为上面如果出现密码账户不正确没有处理相应的异常，所以这里补充一下

![1542776570138](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1542776570138.png)

在认证过程中有一个父异常为：AuthenticationException

该异常有几个子类

a)DisabledAccountException账户失效异常

b)ExcessiveAttemptException：尝试次数过多

c)UnknownAccountException:用户不正确

d)ExpiredCredentialsException:凭证过期异常

e)IncorrectCredentialsException:凭证不正确

虽然shiro为每一种异常都提供了准确的异常类，但是在编写代码过程中，应提示给用户的异常信息为模糊的，常见的处理方式为

```java
try {

    //6.进行用户身份验证
    subject.login(token);
    //通过subject来判断用户是否通过验证
    if (subject.isAuthenticated()) {
        System.out.println("用户登录成功");
    }
}catch (UnknownAccountException e){
    System.out.println("用户或密码不正确");
}catch (IncorrectCredentialsException e){
    System.out.println("用户或密码不正确");
}
```

## 5.6执行过程

1.通过shiro相关api，创建SecurityManager及获取Subject实例

2.封装token信息

3.通过subject.login(token)进行用户认证

​	Subject接收token，通过其实现类DelegatingSubject将token委托给SecurityManager来完成认证。SecurityManager是接口，通过其实现类DefaultSecurityManager来完成相关的功能。由DefaultSecurityManager中login来完成认证过程。在login中调用了authenticate()来完成认证。该方法是由AuthenticatingSecurityManager来完成的。在该类的authenticate()中，通过调用authenticator（认证器）来完成认证工作。Authenticator是由其默认实现类ModularRealmAuthenticator来完成认证。通过ModularRealmAuthenticator中的doAuthenticate来获取Realms信息。如果是单realm，直接将token和realm中的数据进行比较，判断是否认证成功。如果是多realm那么需要通过Authentication Strategy来完成对应的认证工作。

## 5.7JDBCRealm

### 5.7.1.介绍

使用shiro框架来完成认证工作，默认情况下使用的是iniRealm，如果需要使用其他Realm，需要进行相关的配置

### 5.7.2.ini配置文件讲解

```ini
[main]
[users]
[roles]
[urls]
```

#### ①main

是配置应用程序的SecurityManager实例及任何它的依赖组件的地方

```ini
[main]
myRealm=com.hua.realm
securityManager.realm=$myRealm

```



#### ②users

允许你定义一组静态的用户账户。这在大部分拥有少数用户账户或用户账户不需要在运行时被动态的环境下是很有用的

```ini
[users]
zhangsan=1111,role1
```

#### ③roles

允许你把定义在[users]section中的角色与权限关联起来。另外，这在大部分拥有少数用户账户或用户账户不需要在运行时被动态地创建的环境下是很有用的

```ini
[users]
zhangsan=111,role1
[roles]
zole1=user:add,user:delete
```

### 5.7.3.使用JdbcRealm来完成身份认证

通过观察JdbcRealm可知，要实现JdbcRealm

a)需要为JdbcRealm设置dataSource

b)在指定的dataSource所对应的数据库中有用户表users，该表中有username，password，password_salt

数据库

![1542800169720](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1542800169720.png)



shiro.ini

```ini
[main]
dataSource= com.mchange.v2.c3p0.ComboPooledDataSource
dataSource.driverClass=com.mysql.jdbc.Driver
dataSource.jdbcUrl=jdbc:mysql://localhost:3306/shiro
dataSource.user=root
dataSource.password=171911
jdbcRealm=org.apache.shiro.realm.jdbc.JdbcRealm
jdbcRealm.dataSource=$dataSource
securityManager.realm=$jdbcRealm
[users]
zhangsan=1111
lisi=1111
```

测试

```java
package com.hua;

import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.config.IniSecurityManagerFactory;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.subject.Subject;
import org.apache.shiro.util.Factory;

public class JdbcRealmDemo {

    public static void main(String[] args) {
        Factory<SecurityManager> factory=new IniSecurityManagerFactory("classpath:shiro.ini");
        SecurityManager securityManager=factory.getInstance();
        SecurityUtils.setSecurityManager(securityManager);
        Subject subject=SecurityUtils.getSubject();
        UsernamePasswordToken token=new UsernamePasswordToken("zhangsan","1111");
        try {
            subject.login(token);
            if(subject.isAuthenticated()){
                System.out.println("登录成功");
            }
        }catch (AuthenticationException e){
            e.printStackTrace();
            System.out.println("验证失败");
        }
    }
}
```



## 5.8AuthenticationStrategy认证策略

在realm大于1的情况下，在shiro中三种策略

| AuthenticationStrategy       | 描述                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| AtLeastOneSuccessfulStrategy | 如果一个（或更多）Realm验证成功，则整体的尝试被认为是成功的。如果没有一个验证成功，则整体尝试失败 |
| FirstSuccessfulStrategy      | 只要有一个成功地验证的Realm返回的信息将被使用。所有进一步的Realm将被忽略。如果没有一个验证成功，则整体尝试失败。 |
| AllSuccessfulStrategy        | 为了整体的尝试成功，所有配置的Realm必须验证成功。如果没有一个验证成功，则整体尝试失败 |

ModularRealmAuthenticator默认的是AtLeastOneSuccessfulStrategy实现，因为这是最常所需的方案。然而，如果你愿意的话，你可以配置一个不同的方案。

设置认证策略

```ini
[main]
dataSource=com.mchange.v2.c3p0.ComboPooledDataSource
dataSource.driverClass=com.mysql.jdbc.Driver
dataSource.jdbcUrl=jdbc:mysql://localhost:3306/shiro
dataSource.user=root
dataSource.password=171911
jdbcRealm=org.apache.shiro.realm.jdbc.JdbcRealm
#$表示引用对象
jdbcRealm.dataSource=$dataSource
#配置验证器
authenticationStrategy=org.apache.shiro.authc.pam.AllSuccessfulStrategy
securityManager.realm=$jdbcRealm
securityManager.authenticator.authenticationStrategy=$authenticationStrategy
```



# 6.自定义Realm来实现身份认证

## 6.1.分析

JdbcRealm已经实现了从数据中获取用户的验证信息，但是JdbcRealm灵活性太差，如果要实现自己的一些特殊应用时，将不能支持，这个时候可以通过自定义Realm来实现身份的认证功能

## 6.2.Realm介绍

Realm是一个接口，在接口中定义了根据token获得认证信息的方法，shiro实现了一系列的realm。这些不同的Realm实现类提供了不同的功能

AuthenticatingRealm：实现了获取身份信息的功能

AuthorizingRealm：实现了获取权限信息的功能。而这个类继承自上面那个，所以它俩功能都有

通常自定义Realm需要继承AuthorizingRealm

## 6.3.实现自定义Realm

此处先实现认证

UserRealm

```java
package com.hua.realm;

import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.SimpleAuthenticationInfo;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;

/**
 * 自定义Realm的实现
 * 该Realm类实现了两个方法
 * 获取认证信息
 * 获取权限信息
 */
public class UserRealm extends AuthorizingRealm {

    @Override
    public String getName(){
        return "userRealm";
    }

    //完成身份认证（从数据库中取数据），并且返回认证信息
    //如果身份认证失败 返回null
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        //获取用户输入的用户名
        String username=(String)token.getPrincipal();//获取身份信息
        System.out.println("username====="+username);
        //根据用户名到数据库查询密码信息-----模拟
        //鉴定从数据库获取的密码为1111
        String pwd="1111";
        //将从数据库中查询的信息封装到authentication中
        SimpleAuthenticationInfo info=new SimpleAuthenticationInfo(username,pwd,this.getName());
        return info;
    }

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

}
```



shiro.ini

```ini
[main]
userRealm=com.hua.realm.UserRealm
securityManager.realm=$userRealm
```

注意：使用shiro来完成权限管理，shiro并不会去维护数据。shiro中使用的数据，需要程序员根据处理业务将数据传递给shiro的相应接口





# 7.散列算法

即加密

## 7.1.介绍

在身份认证过程中往往会涉及加密。如果不加密那么数据信息不安全。shiro内部实现比较多的散列算法，如MD5，SHA等，并且提供了加盐功能。比如"1111"的MD5 b59c67bf196a4758191e42f76670ceba    ，这个MD5码可以从很多破解网站上找到对应的原密码。但是如果为"1111"+姓名，那么能找到原密码的难度会增加。

## 7.2.测试MD5案例

```java
 public static void main(String[] args) {
        //使用MD5加密算法加密
        Md5Hash md5=new Md5Hash("1111");
        System.out.println("1111=="+md5);
        //加盐
        md5=new Md5Hash("1111","sxt");
        System.out.println("加盐1111=="+md5);
        //加盐，迭代次数
        md5=new Md5Hash("1111","sxt",2);
        System.out.println("加两次盐1111=="+md5);
        //使用SImpleHash，构造器里指定算法
        SimpleHash hash=new SimpleHash("md5","1111","sxt",2);
        System.out.println("跟上面一样"+hash);
    }
```

## 7.3.在自定义的Realm里实现散列算法

UserRealm.java

```java
package hua.realm;

import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.SimpleAuthenticationInfo;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.apache.shiro.util.ByteSource;

/**
 * 自定义Realm的实现
 * 该Realm类实现了两个方法
 * 获取认证信息
 * 获取权限信息
 */
public class UserRealm extends AuthorizingRealm {

    @Override
    public String getName(){
        return "userRealm";
    }

    //完成身份认证（从数据库中取数据），并且返回认证信息
    //如果身份认证失败 返回null
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        //获取用户输入的用户名
        String username=(String)token.getPrincipal();//获取身份信息
        System.out.println("username====="+username);
        //根据用户名到数据库查询密码信息-----模拟
        //鉴定从数据库获取的密码为1111
        String pwd="e41cd85110c7533e3f93b729b25235c3";
        String salt="sxt";
        //跟之前的版本相比这里变了
        SimpleAuthenticationInfo info=new SimpleAuthenticationInfo(username,pwd, ByteSource.Util.bytes(salt),this.getName());
        return info;
    }

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

}
```

shiro.ini

```ini
[main]
credentialsMatcher=org.apache.shiro.authc.credential.HashedCredentialsMatcher
#设置算法
credentialsMatcher.hashAlgorithmName=md5
#设置迭代次数
credentialsMatcher.hashIterations=2

#给自定义realm里面设置散列算法
userRealm=hua.realm.UserRealm
userRealm.credentialsMatcher=$credentialsMatcher
securityManager.realm=$userRealm
```

# 8.授权Authorization

## 8.1.介绍

授权：又称为访问控制，给身份认证通过的人，授予她可以访问某些资源的权限。一个格式良好的权限声明基本上描述了资源以及当Subject与这些资源进行交互时可能出现的行为

权限语句的一些例子

-打开一个文件

-查看'/user/list'网页

-打印文档

-删除用户"jsmith"

大多数资源将支持典型的CRUD（创建、读取、更新、删除）操作，但任何对特定资源有意义的行为都是可以的。

基本概念是，最小的许可声明式基于资源和行为的。

## 8.2.权限粒度

分为粗粒度，细粒度

粗粒度：对user的crud，也就是通常对表的操作

细粒度：是对记录的操作。如：只允许查询id为1的user的工资。

shiro一般管理的是粗粒度的权限，比如：菜单，按钮，url。

一般细粒度的权限是通过业务来控制的。

## 8.3.角色

权限的集合

## 8.4.权限表示规则

资源：操作：实例。可以用通配符表示：

如“user:add”表示对user有添加的权限，user:*  具有所有操作的权限     user:delete:100.表示对user标识为100的记录有删除的权限（这是细粒度的权限，所以说不常用）

## 8.5.Shiro中的授权编码

```java
package com.hua.testRealmDemo;

import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.config.IniSecurityManagerFactory;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.subject.Subject;
import org.apache.shiro.util.Factory;

import java.util.Arrays;


/**
注意自己测试的时候，，，一定要try一下那些异常，处理一下
 */
public class RealmDemo {


    public static void main(String[] args) {
      Factory<SecurityManager> factory=new IniSecurityManagerFactory("classpath:shiro.ini");
      SecurityManager securityManager=factory.getInstance();
      SecurityUtils.setSecurityManager(securityManager);
      Subject subject=SecurityUtils.getSubject();
      UsernamePasswordToken token=new UsernamePasswordToken("zhangsan","1111");
      try{
          //认证
          subject.login(token);
      }catch (AuthenticationException e){
          System.out.println("认证不通过");

      }
      //基于角色的授权
      boolean flag=subject.hasRole("role1");
        System.out.println(flag);
        //判断是否有多个角色
        boolean[] booleans = subject.hasRoles(Arrays.asList("role1", "role2"));
        //可以通过checkRole来检测是否具有某个角色，如果不具有该角色则提出UnauthorizedException
        //而上面那个方法是不会抛出异常的
        subject.checkRole("role2");
        //也可以同时检测多个角色(下面是可变参数的)
        subject.checkRoles("role1","role2");


        //基于资源的授权
        flag=subject.isPermitted("user:delete");
        System.out.println(flag);
        //判断是否多个资源的权限
        subject.isPermittedAll("user:add","user:update","user:delete");
        //也可以通过checkPermission，如果不具有该权限，则抛出UnauthorizedException
        subject.checkPermission("user:dd");

    }
}

```

shiro.ini

```ini
[users]
zhangsan=1111,role1
lisi=1111,role2

[roles]
role1=user:add,user:update,user:delete
role2=user:*
```

## 8.6.Shiro中的授权检查方式有5种

a)编程式

```java
if(subject.hasRole("管理员")){
    //操作某个资源
}
```



b)注解式

在执行指定的方法时，会检测是否具有该权限

```java
@RequiresRoles("管理员")
public void list(){
    //查询数据
}
```



c)标签

```jsp
<%@ taglib %>
<shiro:hasPermission name="user:update">
	<a href="更新"></a>
</shiro:hasPermission>
```

## 8.7.授权的流程

a)获取subject主体

b)判断主体是否通过验证

c)调用subject.isPermitted/hasRole来进行权限的判断

​	-subject是由其实现类DelegatingSubject来调用方法的，该类将处理交给了SecurityManager

​	-SecurityManager是由其实现类DefaultSecurityManager来进行处理，而在该类中isPermitted，其本质是父类AuthorizatingSecurityManager来处理的。该类将处理交给了authorizer（授权器）

​	-Authorizer由其实现类ModularRealmAuthorizer来处理该类可以调用调用相应的Realm类来获取数据，在该类有PermissionResolver对权限字符串进行解析，在对应的Realm中也有对应的PermissionResolver交给WildcardPemissionResolver来进行权限字符串的解析

​	-返回处理结果



# 9.自定义Realm实现授权

## 9.1.分析

仅仅通过配置文件来指定权限不够灵活，并且不方便。在实际的应用中大多数情况下都是将用户信息，角色信息，权限信息保存到了数据库中。所以需要从数据库中去获取相关的数据信息。可以使用shiro提供的jdbcRealm来实现，也可以自定义realm来实现。使用jdbcRealm往往也不够灵活。所以在实际应用大多数情况下都是自定义realm来实现

## 9.2.代码

需要继承AuthorizingRealm

UserRole

```java
package com.hua.realm;

import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.SimpleAuthenticationInfo;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;

import java.util.ArrayList;
import java.util.List;

/**
 * 自定义Realm的实现
 * 该Realm类实现了两个方法
 * 获取认证信息
 * 获取权限信息
 */
public class UserRealm extends AuthorizingRealm {

    @Override
    public String getName(){
        return "userRealm";
    }

    //完成身份认证（从数据库中取数据），并且返回认证信息
    //如果身份认证失败 返回null
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        //获取用户输入的用户名
        String username=(String)token.getPrincipal();//获取身份信息
        System.out.println("username====="+username);
        //根据用户名到数据库查询密码信息-----模拟
        //鉴定从数据库获取的密码为1111
        String pwd="1111";
        //将从数据库中查询的信息封装到authentication中
        SimpleAuthenticationInfo info=new SimpleAuthenticationInfo(username,pwd,this.getName());
        return info;
    }

    //授权的信息
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        String username=principalCollection.getPrimaryPrincipal().toString();
        System.out.println("授权----------");
        System.out.println("username========"+username);
        //根据用户名到数据库查询该用户对应的权限信息----模拟
        List<String> permission=new ArrayList<String>();
        permission.add("user:add");
        permission.add("user:delete");
        permission.add("user:update");
        permission.add("user:find");
        //无参的版本
        SimpleAuthorizationInfo info=new SimpleAuthorizationInfo();
        for(String perm:permission){
            info.addStringPermission(perm);
        }
        return info;
    }

}
```

shiro.ini

```ini
[main]
userRealm=com.hua.realm.UserRealm
securityManager.realm=$userRealm
```

测试

```java
package com.hua.testRealmDemo;

import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.config.IniSecurityManagerFactory;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.subject.Subject;
import org.apache.shiro.util.Factory;


/**
 * @Author lenovo
 * @Version 2018/11/21
 */
public class RealmDemo {


    public static void main(String[] args) {
        Factory<SecurityManager> factory=new IniSecurityManagerFactory("classpath:shiro.ini");
        SecurityManager securityManager=factory.getInstance();
        SecurityUtils.setSecurityManager(securityManager);
        Subject subject=SecurityUtils.getSubject();
        UsernamePasswordToken token=new UsernamePasswordToken("wangwu","1111");
        try {
            subject.login(token);
            if(subject.isAuthenticated()){
                System.out.println("登录成功");
            }
        }catch (AuthenticationException e){
            e.printStackTrace();
            System.out.println("验证失败");
        }

        boolean permitted = subject.isPermitted("user:add");
        System.out.println(permitted);
    }
}
```

# 10.整合ssm



10.1.准备工作

整合ssm框架并且实现用户登录菜单权限

我是把之前的rbac项目拿过来用，然后把原来的拦截器去掉，，，

10.2.加入shiro

①加入相关依赖

pom.xml

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
    <version>1.2.3</version>
</dependency>
```

②加入相关配置

```xml
<!--创建凭证管理器-->
<bean id="credentialMatcher" class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
    <!--配置加密方式-->
    <property name="hashAlgorithmName" value="md5"/>
    <!--配置散列次数-->
    <property name="hashIterations" value="2"/>
</bean>

<!--创建realm-->
<bean id="userRealm" class="com.hua.realm.UserRealm">
    <!--注入凭证管理器-->
    <property name="credentialsMatcher" ref="credentialMatcher"/>
</bean>

<!--配置安全管理器-->
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
    <property name="realm" ref="userRealm"/>
</bean>

<!--配置shiro的web过滤器-->
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
    <!--配置安全管理器-->
    <property name="securityManager" ref="securityManager"/>
    <!--要求登录时的连接（登录页面地址），非必须的属性，默认会自动寻找web工程目录下的/login.jsp-->
    <property name="loginUrl" value="/login"/>
    <!--登录成功后要跳转的连接（本例中此属性用不到，因为登录成功后的处理逻辑在UserConroller里硬编码）-->
    <!--<property name="successUrl" value="/success.action"/>-->
    <!--用户访问未对其授权的资源时，所显示的连接-->
    <!--<property name="unauthorizedUrl" value="/error"/>-->
    <!--过滤器链定义，从上向下顺序执行，一般将/**放在最下边-->
    <property name="filterChainDefinitions">
        <!--不用认证也可以访问的页面-->
        <value>
            /index.jsp*=anon
            /login*=anon
            <!--如果用户访问/logout就使用shiro注销session-->
            /**=anon
        </value>
    </property>
</bean>
```

