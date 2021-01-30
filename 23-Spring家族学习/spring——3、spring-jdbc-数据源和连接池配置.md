#### 数据源和连接池配置

##### 一、配置单数据源

###### 1、springboot：

![屏幕快照 2021-01-30 上午12.23.19](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202021-01-30%20%E4%B8%8A%E5%8D%8812.23.19.png)

```java
@SpringBootApplication
@Slf4j
public class DatasourceDemoApplication implements CommandLineRunner {

    @Autowired
    private DataSource dataSource;

    public static void main(String[] args) {
        SpringApplication.run(DatasourceDemoApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        showConnection();
    }

    private void showConnection() throws SQLException{
        log.info(dataSource.toString());
        Connection conn = dataSource.getConnection();
        log.info(conn.toString());
        conn.close();
    }

}
```

###### **2、直接通过spring bean进行配置：**

数据源相关

• DataSource（根据选择的连接池实现决定）

事务相关（可选）

• PlatformTransactionManager（DataSourceTransactionManager） 

• TransactionTemplate

操作相关（可选）

• JdbcTemplate

```java
@Configuration
@EnableTransactionManagement
public class DataSourceDemo {
    @Autowired
    private DataSource dataSource;

    public static void main(String[] args) throws SQLException {
        ApplicationContext applicationContext =
                new ClassPathXmlApplicationContext("applicationContext*.xml");
        showBeans(applicationContext);
        dataSourceDemo(applicationContext);
    }

    @Bean(destroyMethod = "close")
    public DataSource dataSource() throws Exception {
        Properties properties = new Properties();
        properties.setProperty("driverClassName", "org.h2.Driver");
        properties.setProperty("url", "jdbc:h2:mem:testdb");
        properties.setProperty("username", "sa");
        return BasicDataSourceFactory.createDataSource(properties);
    }

    @Bean
    public PlatformTransactionManager transactionManager() throws Exception {
        return new DataSourceTransactionManager(dataSource());
    }

    private static void showBeans(ApplicationContext applicationContext) {
        System.out.println(Arrays.toString(applicationContext.getBeanDefinitionNames()));
    }

    private static void dataSourceDemo(ApplicationContext applicationContext) throws SQLException {
        DataSourceDemo demo = applicationContext.getBean("dataSourceDemo", DataSourceDemo.class);
        demo.showDataSource();
    }

    public void showDataSource() throws SQLException {
        System.out.println(dataSource.toString());
        Connection conn = dataSource.getConnection();
        System.out.println(conn.toString());
        conn.close();
    }
}

```

还可以使用spring xml配置：

```xml
 <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
          destroy-method="close">
        <property name="driverClassName" value="org.h2.Driver"/>
        <property name="url" value="jdbc:h2:mem:testdb"/>
        <property name="username" value="SA"/>
        <property name="password" value=""/>
 </bean>
```



###### 3、springboot原理

**DataSourceAutoConfifiguration** 

• 配置 DataSource

**DataSourceTransactionManagerAutoConfifiguration** 

• 配置 DataSourceTransactionManager

**JdbcTemplateAutoConfifiguration** 

• 配置 JdbcTemplate

符合条件时才进⾏配置

###### 4、springboot数据源相关配置属性

通⽤

• spring.datasource.url=jdbc:mysql://localhost/test

• spring.datasource.username=dbuser

• spring.datasource.password=dbpass

• spring.datasource.driver-class-name=com.mysql.jdbc.Driver（可选）

```properties
management.endpoints.web.exposure.include=*
spring.output.ansi.enabled=ALWAYS

spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.username=sa
spring.datasource.password=
spring.datasource.hikari.maximumPoolSize=5
spring.datasource.hikari.minimumIdle=5
spring.datasource.hikari.idleTimeout=600000
spring.datasource.hikari.connectionTimeout=30000
spring.datasource.hikari.maxLifetime=1800000
```



初始化内嵌数据库

• spring.datasource.initialization-mode=embedded|always|never

• spring.datasource.schema与spring.datasource.data确定初始化SQL⽂件

• spring.datasource.platform=hsqldb | h2 | oracle | mysql | postgresql（与前者对应）

![屏幕快照 2021-01-30 上午12.59.33](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202021-01-30%20%E4%B8%8A%E5%8D%8812.59.33.png)

schema.sql

```sql
CREATE TABLE FOO (ID INT IDENTITY, BAR VARCHAR(64));
```

data.sql

```sql
INSERT INTO FOO (ID, BAR) VALUES (1, 'aaa');
INSERT INTO FOO (ID, BAR) VALUES (2, 'bbb');
```



##### 二、多数据源配置

不同数据源的配置要分开。

1、配置两组DataSource
```properties
management.endpoints.web.exposure.include=*
spring.output.ansi.enabled=ALWAYS

foo.datasource.url=jdbc:h2:mem:foo
foo.datasource.username=sa
foo.datasource.password=

bar.datasource.url=jdbc:h2:mem:bar
bar.datasource.username=sa
bar.datasource.password=
```

2、排除 spring boot的自动配置

```java
@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class,
        DataSourceTransactionManagerAutoConfiguration.class,
        JdbcTemplateAutoConfiguration.class})
@Slf4j
public class MultiDataSourceDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(MultiDataSourceDemoApplication.class, args);
    }

    @Bean
    @ConfigurationProperties("foo.datasource")
    public DataSourceProperties fooDataSourceProperties() {
        return new DataSourceProperties();
    }

    @Bean
    public DataSource fooDataSource() {
        DataSourceProperties dataSourceProperties = fooDataSourceProperties();
        log.info("foo datasource: {}", dataSourceProperties.getUrl());
        return dataSourceProperties.initializeDataSourceBuilder().build();
    }

    @Bean
    @Resource
    public PlatformTransactionManager fooTxManager(DataSource fooDataSource) {
        return new DataSourceTransactionManager(fooDataSource);
    }

    @Bean
    @ConfigurationProperties("bar.datasource")
    public DataSourceProperties barDataSourceProperties() {
        return new DataSourceProperties();
    }

    @Bean
    public DataSource barDataSource() {
        DataSourceProperties dataSourceProperties = barDataSourceProperties();
        log.info("bar datasource: {}", dataSourceProperties.getUrl());
        return dataSourceProperties.initializeDataSourceBuilder().build();
    }

    @Bean
    @Resource
    public PlatformTransactionManager barTxManager(DataSource barDataSource) {
        return new DataSourceTransactionManager(barDataSource);
    }
```



##### 三、连接池——HikariCP光

###### 1、HikariCP为什么块

- 字节码级别优化（很多⽅法通过 **JavaAssist** ⽣成）

- ⼤量⼩改进
  - ⽤ FastStatementList 代替 ArrayList
  - ⽆锁集合 ConcurrentBag
  - 代理类的优化（⽐如，⽤ invokestatic 代替了 invokevirtual）

**spring boot2.x 默认使用 HikariCP**

**Spring Boot 1.x** 

• 默认使⽤ Tomcat 连接池，需要移除 tomcat-jdbc 依赖

• 添加spring.datasource.type=com.zaxxer.hikari.HikariDataSource

###### 2、HikariCP常用配置参数

DataSourceConfiguration 这个生成数据源的类中读取了  spring.datasource.hikari

```java
/**
	 * Hikari DataSource configuration.
	 */
	@ConditionalOnClass(HikariDataSource.class)
	@ConditionalOnMissingBean(DataSource.class)
	@ConditionalOnProperty(name = "spring.datasource.type", havingValue = "com.zaxxer.hikari.HikariDataSource", matchIfMissing = true)
	static class Hikari {

		@Bean
		@ConfigurationProperties(prefix = "spring.datasource.hikari")
		public HikariDataSource dataSource(DataSourceProperties properties) {
			HikariDataSource dataSource = createDataSource(properties,
					HikariDataSource.class);
			if (StringUtils.hasText(properties.getName())) {
				dataSource.setPoolName(properties.getName());
			}
			return dataSource;
		}

	}
```

常⽤配置

• spring.datasource.hikari.maximumPoolSize=10

• spring.datasource.hikari.minimumIdle=10

• spring.datasource.hikari.idleTimeout=600000

• spring.datasource.hikari.connectionTimeout=30000

• spring.datasource.hikari.maxLifetime=1800000

其他配置详⻅ **HikariCP** 官⽹

• https://github.com/brettwooldridge/HikariCP

##### 四、连接池——Alibaba Druid

Druid连接池是阿⾥巴巴开源的数据库连接池项⽬。Druid连接池为监控⽽⽣，

内置强⼤的监控功能，监控特性不影响性能。功能强⼤，能防SQL注⼊，内置

Logging能诊断Hack应⽤⾏为。

###### 1、实⽤的功能

• 详细的监控（真的是全⾯）

• ExceptionSorter，针对主流数据库的返回码都有⽀持

• SQL 防注⼊

• 内置加密配置

• 众多扩展点，⽅便进⾏定制

###### 2、使用配置

**直接配置 DruidDataSource**

```xml
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close"> 
     <property name="url" value="${jdbc_url}" />
     <property name="username" value="${jdbc_user}" />
     <property name="password" value="${jdbc_password}" />
     <property name="filters" value="stat" />
     <property name="maxActive" value="20" />
     <property name="initialSize" value="1" />
     <property name="maxWait" value="60000" />
     <property name="minIdle" value="1" />
     <property name="timeBetweenEvictionRunsMillis" value="60000" />
     <property name="minEvictableIdleTimeMillis" value="300000" />
     <property name="testWhileIdle" value="true" />
     <property name="testOnBorrow" value="false" />
     <property name="testOnReturn" value="false" />
     <property name="poolPreparedStatements" value="true" />
     <property name="maxOpenPreparedStatements" value="20" />
     <property name="asyncInit" value="true" />
 </bean>
```

配置项：https://github.com/alibaba/druid/wiki/DruidDataSource%E9%85%8D%E7%BD%AE%E5%B1%9E%E6%80%A7%E5%88%97%E8%A1%A8

或

**springboot使用，注意：**需要将默认的连接池排除掉

```xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jdbc</artifactId>
			<exclusions>
				<exclusion>
					<artifactId>HikariCP</artifactId>
					<groupId>com.zaxxer</groupId>
				</exclusion>
			</exclusions>
</dependency>

……
<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid-spring-boot-starter</artifactId>
			<version>1.1.10</version>
</dependency>

```

配置通过 spring.datasource.druid.*

```properties
spring.output.ansi.enabled=ALWAYS

spring.datasource.url=jdbc:h2:mem:foo
spring.datasource.username=sa
spring.datasource.password=n/z7PyA5cvcXvs8px8FVmBVpaRyNsvJb3X7YfS38DJrIg25EbZaZGvH4aHcnc97Om0islpCAPc3MqsGvsrxVJw==

spring.datasource.druid.initial-size=5
spring.datasource.druid.max-active=5
spring.datasource.druid.min-idle=5
spring.datasource.druid.filters=conn,config,stat,slf4j

spring.datasource.druid.connection-properties=config.decrypt=true;config.decrypt.key=${public-key}
spring.datasource.druid.filter.config.enabled=true

spring.datasource.druid.test-on-borrow=true
spring.datasource.druid.test-on-return=true
spring.datasource.druid.test-while-idle=true

public-key=MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBALS8ng1XvgHrdOgm4pxrnUdt3sXtu/E8My9KzX8sXlz+mXRZQCop7NVQLne25pXHtZoDYuMh3bzoGj6v5HvvAQ8CAwEAAQ==

```

**Filter** 配置

• spring.datasource.druid.fifilters=stat,confifig,wall,log4j （全部使⽤默认值）

**密码加密**

• spring.datasource.password=<加密密码> 

• spring.datasource.druid.fifilter.confifig.enabled=true

• spring.datasource.druid.connection-properties=confifig.decrypt=true;confifig.decrypt.key=<public-key>

**SQL** 防注⼊

• spring.datasource.druid.fifilter.wall.enabled=true

• spring.datasource.druid.fifilter.wall.db-type=h2

• spring.datasource.druid.fifilter.wall.confifig.delete-allow=false

• spring.datasource.druid.fifilter.wall.confifig.drop-table-allow=false

###### 3、Druid Filter

• ⽤于定制连接池操作的各种环节

• 可以继承 FilterEventAdapter 以便⽅便地实现 Filter

• 修改 META-INF/druid-fifilter.properties 增加 Filter 配置

druid-filter.properties

```properties
druid.filters.conn=geektime.spring.data.druiddemo.ConnectionLogFilter
```

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202021-01-30%20%E4%B8%8A%E5%8D%882.08.04.png" alt="屏幕快照 2021-01-30 上午2.08.04" style="zoom:25%;" />

```java
@Slf4j
public class ConnectionLogFilter extends FilterEventAdapter {

    @Override
    public void connection_connectBefore(FilterChain chain, Properties info) {
        log.info("BEFORE CONNECTION!");
    }

    @Override
    public void connection_connectAfter(ConnectionProxy connection) {
        log.info("AFTER CONNECTION!");
    }
}
```

![屏幕快照 2021-01-30 上午2.10.46](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202021-01-30%20%E4%B8%8A%E5%8D%882.10.46.png)

