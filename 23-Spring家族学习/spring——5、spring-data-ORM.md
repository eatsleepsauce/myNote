

#### JPA 和 mybatis

##### 一、Spring Data

保留底层存储 特性的同时，提供了相对一致的抽象模型，主要模块：

- spring data commons
- spring data jdbc
- spring data jpa
- spring data redis
- ...

spring依赖：

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.data</groupId>
		<artifactId>spring-data-releasetrain</artifactId>
		<version>Lovelace-SR4</version>
		<scope>import</scope>
		<type>pom</type>
  </dependency>
</dependencies>
```

springboot依赖：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```



##### 二、Java Persistence API

**JPA** 为对象关系映射提供了⼀种基于 **POJO** 的持久化模型，2006年，Hibernate 3.2 成为 JPA 实现。

- 简化数据持久化代码的开发⼯作
- 为 Java 社区屏蔽不同持久化 API 的差异

**常用JPA注解:**

- 实体
  - @Entity、@MappedSuperclass
  - @Table(name)
- 主键 @Id
  - @GeneratedValue(strategy, generator)
  - @SequenceGenerator(name, sequenceName)
- 映射
  - @Column(name, nullable, length, insertable, updatable)
  - @JoinTable(name)、@JoinColumn(name)
- 关系
  - @OneToOne、@OneToMany、@ManyToOne、@ManyToMany
  - @OrderBy

```java
@Entity(name ="Product") 
public static class Product {
  @Id
  @GeneratedValue(strategy = GenerationType.SEQUENCE,generator ="sequence-generator")
  @SequenceGenerator(name = "sequence-generator",sequenceName ="product_sequence")
  private Long id;

  @Column(name ="product_name") 
  private String name;

  // ...
}
```

**spring data jpa 操作数据库：**

在config类上加上注解  **@EnableJpaRepositories**  就可以发现扩展了 **Repository<T, ID>** 的接⼝：

- CrudRepository<T, ID>
- PagingAndSortingRepository<T, ID>
- JpaRepository<T, ID>

查询定义：

- find…By… / read…By… / query…By… / get…By…
- count…By…
- …OrderBy…[Asc / Desc]
- And / Or / IgnoreCase
- Top / First / Distinct

分页查询：

- PagingAndSortingRepository<T, ID>
- Pageable / Sort
- Slice<T> / Page<T>

保存：调用repository的save方法

**@NoRepositoryBean** 不生成 repostitory bean，通常来做其他repository的基类

```java
@NoRepositoryBean
pubtic interface BaseRepository<T,Long> extends PagingAndSortingRepositorye<T,Long> {
	List<T>findTop3By0rderByupdateTimeDescIdAsc();
}


pubLic interface CoffeeOrderRepository extends BaseRepository<CoffeeOrder,Long>{ 
  List<CoffeeOrder> findByCustomerOrderById(String customer);
  List<Coffeeorder> findByItems_Name(String name);
}

```



##### 三、Repository 是怎么从接口变成bean的

**JpaRepositoriesRegistrar**

• 激活了 @EnableJpaRepositories

• 返回了 JpaRepositoryConfigExtension

**RepositoryBeanDefifinitionRegistrarSupport.registerBeanDefifinitions** 

• 注册 Repository Bean（类型是 JpaRepositoryFactoryBean ）

**RepositoryConfifigurationExtensionSupport.getRepositoryConfifigurations** 

• 取得 Repository 配置

**JpaRepositoryFactory.getTargetRepository** 

• 创建了 Repository

接口中的方法执行：

**RepositoryFactorySupport.getRepository** 添加了**Advice** 

• DefaultMethodInvokingMethodInterceptor

• QueryExecutorMethodInterceptor

**AbstractJpaQuery.execute** 执⾏具体的查询

语法解析在 **Part** 中



##### 四、MyBatis操作数据库

在 **Spring** 中使⽤ **MyBatis** ：

- MyBatis Spring Adapter（https://github.com/mybatis/spring） 

- MyBatis Spring-Boot-Starter（https://github.com/mybatis/spring-boot-starter）

配置：

- mybatis.mapper-locations = classpath*:mapper/**/*.xml
- mybatis.type-aliases-package = 类型别名的包名
- mybatis.type-handlers-package = TypeHandler扫描包名，类型转换
- mybatis.configuration.map-underscore-to-camel-case = true  下划线转驼峰映射

Mapper的定义与扫描：

- @MapperScan 配置扫描位置
- @Mapper 定义接⼝
- 映射的定义—— XML 与注解

```java
@Mapper
public interface CoffeeMapper {

  @Insert("insert into t_coffee (name,price,create_time,update_time)"
  +"values(#{name},#{price},now(),now())")
  @Options(useGeneratedKeys = true) 
  int save(Coffee coffee);

  @Select("select * from t_coffee where id = #(id}")
  @Results({
  @Result(id = true,column = "id",property ="id"),
  @Result(column = "create_time",property ="createTime"),
  // map-underscore-to-camel-case= true 可以实现一样的效果
  // @Resutt(coLumn ="update_time",property ="updateTime"), 
  })
  Coffee findById(@Param("id")Long id);
}


```



##### 五、MyBatis更好用的工具

###### 1、MyBatis Generator

**MyBatis Generator**（**http://www.mybatis.org/generator/**） ，MyBatis 代码⽣成器，根据数据库表⽣成相关代码：

- POJO
- Mapper 接⼝
- SQL Map XML

**运行MyBatis Generator:**

- 命令⾏   java -jar mybatis-generator-core-x.x.x.jar -confifigfifile generatorConfifig.xml 

- **Maven Plugin**（**mybatis-generator-maven-plugin**） 

  - mvn mybatis-generator:generate 

  - ${basedir}/src/main/resources/generatorConfifig.xml

    

配置......

**generatorConfifiguration** 

- **context** 
  - jdbcConnection
  - javaModelGenerator
  - sqlMapGenerator
  - javaClientGenerator （ANNOTATEDMAPPER / XMLMAPPER / MIXEDMAPPER） 
  - table

context中还可以增加内置插件：

内置插件都在 **org.mybatis.generator.plugins** 包中

- FluentBuilderMethodsPlugin
- ToStringPlugin
- SerializablePlugin
- RowBoundsPlugin

###### 2、MyBatis PageHelper

**MyBatis PageHepler**（**https://pagehelper.github.io**） 

- ⽀持多种数据库
- ⽀持多种分⻚⽅式
- SpringBoot ⽀持（https://github.com/pagehelper/pagehelper-spring-boot ） pagehelper-spring-boot-starter