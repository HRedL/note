# 通用Mapper学习

### 1.在maven中引入依赖

#### ①springboot中的启动器

```xml
<dependency>
   <groupId>tk.mybatis</groupId>
   <artifactId>mapper-spring-boot-starter</artifactId>
   <version>1.2.3</version>
</dependency>
```



#### ②ssm项目中的依赖

```xml
<dependency>
  <groupId>tk.mybatis</groupId>
  <artifactId>mapper</artifactId>
  <version>4.0.4</version>
</dependency>
```

### 2.修改spring中的自动扫描包

#### ①springboot修改主启动类上的@MapperScan注解的所在包

``` java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import tk.mybatis.spring.annotation.MapperScan;

//从原来的org改成tk
@MapperScan
@SpringBootApplication
public class TssApplication {

   public static void main(String[] args) {
      SpringApplication.run(TssApplication.class, args);
   }
}
```

#### ②ssm项目修改spring配置文件中自动扫描包类型

``` xml
<!--4 自动扫描对象关系映射 -->
<!--还是把org改成tk-->
<bean id="mapperScannerConfigurer" class="tk.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    <property name="basePackage" value="com.tss"/>
    <property name="annotationClass" value="com.tss.common.annotation.MybatisDao"/>
</bean>
```

### 3.在实体类上加入@Table注解

因为此处是初步使用：请将数据库中表名设置成和实体类属性名相同，或者说下划线命名和驼峰命名相对应（即emp_id对应empId）

```java
import javax.persistence.Table;

@Table(name = "building")
public class Building  {

}
```

### 4.把dao接口继承Mapper<T>接口

T类型表示实体类的类型，然后继承这个接口

```java 
import com.hua.tss.modules.building.entity.Building;
import tk.mybatis.mapper.common.Mapper;

public interface BuildingDao extends Mapper<Building>{
    
}
```

### 5.在service使用dao接口中自动生成的方法即可





# 注解

### 1.@Table注解

作用：建立实体类和数据库表之间的对应关系。

默认规则：实体类名对应表名

用法：注解在实体类上，在@Table注解的name属性中指定目标数据库表的表名

### 2.@Column注解

作用：建立实体类属性和数据库表字段之间的对应关系

默认规则：实体类属性使用驼峰命名，数据库表用下划线

用法：注解在实体类属性上，在@Column注解的name属性中指定目标数据库表中的字段名

### 3.@Id注解

作用：标注主键是哪个字段

默认规则：不指定主键，那么以全字段作为联合主键

用法：注解在实体类属性上，@Id注解的字段作为主键，在调用xxxByPrimarykey的时候做正确的where字句拼接

### 4.@GeneratedValue注解

作用：让通用Mapper在执行insert操作之后将数据库自动生成的主键值回写到实体类对象中

用法：自增主键

``` java
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Integer id;
```

​	基于序列生成的主键（oracle数据库）

```java
@GeneratedValue（
	strategy = GenerationType.IDENTITY,
	generator = "select SEQ_ID.nextval from dual"
）
private Integer id
```

官方给的例子，也就是需要再给一个查询主键的sql语句

### 5.@Transient注解

一般情况下，实体中的字段和数据库表中的字段量--对应的，但是也有很多情况我们会在实体中增加一些额外的属性，这种情况下，就需要使用@Transient注解来告诉通用Mapper这不是表中的字段

默认情况下，只有简单类型会被自动认为是表中的字段（可以通过配置中的useSimpleType控制）

对于类中的复杂对象，以及Map，List等属性不需要配置这个注解



# 方法

### 1.Entity   selectOne（Entity e）

##### -根据条件查询单个实体

##### -实体类封装查询条件生成WHERE子句的规则

​	使用非空的值生成WHERE子句

​	在条件表达式中使用=进行比较

##### -要求必须返回一个实体类结果，如果有多个，则会抛出异常



类似方法

List<Entity> select(Entity e)，，，根据条件查询多个实体

List<Entity> selectAll()，，，，，查询所有实体

int selectCount(Entity e)，，，，，根据条件查询计数

### 2.Entity   selectByPrimarykey(Object o)

通用Mapper在执行**xxxByPrimaryKey（）**方法时，如果没有用@Id注解指定主键，那么就把所有字段联合起来作为表的联合主键进行查询（这样在大多数情况下是不正确的）

所以说在用这个方法的时候，不要忘了在实体类的主键属性上加上@Id注解



### 3.int  insert(Entity e)

向数据库中插入数据，如果想在插入数据之后立刻使用这个对象的主键（id），也就是插入之后回写主键，那么需要在主键上加上@GeneratedValue注解

### 4.int insertSelective(Entity e)

非主键字段不为null才加入插入语句中

进而，

xxxSelective

即非主键字段不为null才加入sql语句中



### 5.QBC查询

即Query By Criteria

Criteria是Criterion的复数，意思是：标准，准则。在sql语句中意思是查询条件

QBC查询

```java
//1.创建Example对象
Example example=new Example(Employee.class);
//i.设置排序信息
example.orderBy("empSalary").asc().orderBy("empAge").desc()
//ii.设置“去重”
example.setDistinct(true);
//iii.设置select字段
example.selectProperties("empName","empSalary");

//2.通过Example对象创建Criteria对象
Criteria criteria01=example.createCriteria();
Criteria criteria02=example.createCriteria();
//在两个Criteria对象中分别设置查询条件
criteria01.andGreaterThan("empSalary",3000).andLessThan("empAge",25);
criteria02.andLessThan("empSalary",5000).andGreaterThan("empAge",30);
//4.使用OR关键词组装两个Criteria对象
example.OR
```

# 自定义Mapper

自定义一个接口，继承任意想继承的通用Mapper工具里面的父接口

在spring配置文件中修改如下配置

```xml
<!--4 自动扫描对象关系映射 -->
<!--还是把org改成tk-->
<bean id="mapperScannerConfigurer" class="tk.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    <property name="basePackage" value="com.tss"/>
    <property name="annotationClass" value="com.tss.common.annotation.MybatisDao"/>
    <property name="properties">
    	<value>
        	mappers=com.hua.mapper.MyMapper
        </value>
    </property>
</bean>
```

注意：自定义的接口不要和dao的哪个Mapper接口放到同一个包下，上面的配置也不要写上范型

# 开启二级缓存

首先，要求mybatis自己开启了二级缓存

第二，实体类实现序列化

第三，在dao接口上加上注解@CacheNamespace

这个东西用的不是特别，一般都用redis做缓存，它自己自带的这种不太常用。



# 类型处理器：TypeHandler

首先先写一个自定义类型转换器

其次，使用自定义类型转换器（）

​	方法一 ：字段级别：@ColumnType注解

​	方法二：全局级别：在Mybatis配置文件中配置typeHandlers











