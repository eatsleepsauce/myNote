#### spring-jdbc使用以及spring常用注解

##### 一、spring的JDBC操作类

• core，JdbcTemplate 等相关核⼼接⼝和类

• datasource，数据源相关的辅助类

• object，将基本的 JDBC 操作封装成对象

• support，错误码等其他辅助⼯具

##### 二、常用注解

###### 1、常用的bean注解

• @Component

• @Repository

• @Service

• @Controller

• @RestController

###### 2、**Java Confifig** 相关注解

• @Configuration  标明当前ava类是配置类

• @ImportResource  注入资源文件

• @ComponentScan 扫描哪些包

• @Bean 在java configuration的类中标明在一个方法上时，返回值就做一个bean 存在于 spring applicationcontex中

• @ConfigurationProperties  把properties中的配置绑定过来

###### 3、定义相关注解

• @Component / @Repository / @Service

• @Controller / @RestController（带有responsebody 的@Controller）

• @RequestMapping

###### 4、注⼊相关注解

• @Autowired（注入，默认类型注入 配合@Qualifier 变成名字注入） / @Qualifier （指定bean的名字）/ @Resource（按指定bean的名字注入）

• @Value 在在我们的bean注入一些常量或则spel表达式

##### 三、JDBC操作

```java
@SpringBootApplication
@Slf4j
public class SimpleJdbcDemoApplication implements CommandLineRunner {
    @Autowired
    private FooDao fooDao;
    @Autowired
    private BatchFooDao batchFooDao;

    public static void main(String[] args) {
        SpringApplication.run(SimpleJdbcDemoApplication.class, args);
    }

    @Bean
    @Autowired
    public SimpleJdbcInsert simpleJdbcInsert(JdbcTemplate jdbcTemplate) {
        return new SimpleJdbcInsert(jdbcTemplate)
                .withTableName("FOO").usingGeneratedKeyColumns("ID");
    }

    @Bean
    @Autowired
    public NamedParameterJdbcTemplate namedParameterJdbcTemplate(DataSource dataSource) {
        return new NamedParameterJdbcTemplate(dataSource);
    }

    @Override
    public void run(String... args) throws Exception {
        fooDao.insertData();
        batchFooDao.batchInsert();
        fooDao.listData();
    }

}
```

###### 1、JdbcTemplate 

• query

• queryForObject

• queryForList

• update

• execute

```java
@Slf4j
@Repository
public class FooDao {
    @Autowired
    private JdbcTemplate jdbcTemplate;
    @Autowired
    private SimpleJdbcInsert simpleJdbcInsert;

    public void insertData() {
        Arrays.asList("b", "c").forEach(bar -> {
            jdbcTemplate.update("INSERT INTO FOO (BAR) VALUES (?)", bar);
        });

        HashMap<String, String> row = new HashMap<>();
        row.put("BAR", "d");
        Number id = simpleJdbcInsert.executeAndReturnKey(row);
        log.info("ID of d: {}", id.longValue());
    }

    public void listData() {
        log.info("Count: {}",
                jdbcTemplate.queryForObject("SELECT COUNT(*) FROM FOO", Long.class));

        List<String> list = jdbcTemplate.queryForList("SELECT BAR FROM FOO", String.class);
        list.forEach(s -> log.info("Bar: {}", s));

        List<Foo> fooList = jdbcTemplate.query("SELECT * FROM FOO", new RowMapper<Foo>() {
            @Override
            public Foo mapRow(ResultSet rs, int rowNum) throws SQLException {
                return Foo.builder()
                        .id(rs.getLong(1))
                        .bar(rs.getString(2))
                        .build();
            }
        });
        fooList.forEach(f -> log.info("Foo: {}", f));
    }
}
```

###### 2、批处理

（1）JdbcTemplate 

- batchUpdate

- BatchPreparedStatementSetter

（2）NamedParameterJdbcTemplate 

- batchUpdate

- SqlParameterSourceUtils.createBatch

```java
@Repository
public class BatchFooDao {
    @Autowired
    private JdbcTemplate jdbcTemplate;
    @Autowired
    private NamedParameterJdbcTemplate namedParameterJdbcTemplate;

    public void batchInsert() {
        jdbcTemplate.batchUpdate("INSERT INTO FOO (BAR) VALUES (?)",
                new BatchPreparedStatementSetter() {
                    @Override
                    public void setValues(PreparedStatement ps, int i) throws SQLException {
                        ps.setString(1, "b-" + i);
                    }

                    @Override
                    public int getBatchSize() {
                        return 2;
                    }
                });

        List<Foo> list = new ArrayList<>();
        list.add(Foo.builder().id(100L).bar("b-100").build());
        list.add(Foo.builder().id(101L).bar("b-101").build());
        namedParameterJdbcTemplate
                .batchUpdate("INSERT INTO FOO (ID, BAR) VALUES (:id, :bar)",
                        SqlParameterSourceUtils.createBatch(list));
    }
}
```

##### 四、spring的事务抽象

**⼀致的事务模型**，针对下面的都是一个事务模型

- JDBC/Hibernate/myBatis
- DataSource/JTA

**事务抽象的核⼼接⼝**

- **PlatformTransactionManager** 
  -  DataSourceTransactionManager
  -  HibernateTransactionManager
  -  JtaTransactionManager

- **TransactionDefifinition** 
  - Propagation
  - Isolation
  - Timeout
  - Read-only status

![image-20210130205201177](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210130205201177.png)

![image-20210130205220272](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210130205220272.png)

![image-20210130205234588](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210130205234588.png)

**事务传播特性**

![image-20210130205500320](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210130205500320.png)

**REQUIRES_NEW**，始终启动⼀个新事务，两个事务没有关联。

**NESTED**，在原事务内启动⼀个内嵌事务，两个事务有关联，外部事务回滚，内嵌事务也会回滚。内嵌事务回滚，外部事务不会滚。



**事务隔离特性**，默认时 -1 表示取决于数据库

![image-20210130205800489](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210130205800489.png)

###### 1、编程式事务

**TransactionTemplate** 

• TransactionCallback

• TransactionCallbackWithoutResult

**PlatformTransactionManager** 

• 可以传⼊TransactionDefifinition进⾏定义

```java
@SpringBootApplication
@Slf4j
public class ProgrammaticTransactionDemoApplication implements CommandLineRunner {
	@Autowired
	private TransactionTemplate transactionTemplate;
	@Autowired
	private JdbcTemplate jdbcTemplate;

	public static void main(String[] args) {
		SpringApplication.run(ProgrammaticTransactionDemoApplication.class, args);
	}

	@Override
	public void run(String... args) throws Exception {
		log.info("COUNT BEFORE TRANSACTION: {}", getCount());
		transactionTemplate.execute(new TransactionCallbackWithoutResult() {
			@Override
			protected void doInTransactionWithoutResult(TransactionStatus transactionStatus) {
				jdbcTemplate.execute("INSERT INTO FOO (ID, BAR) VALUES (1, 'aaa')");
				log.info("COUNT IN TRANSACTION: {}", getCount());
				transactionStatus.setRollbackOnly();
			}
		});
		log.info("COUNT AFTER TRANSACTION: {}", getCount());
	}

	private long getCount() {
		return (long) jdbcTemplate.queryForList("SELECT COUNT(*) AS CNT FROM FOO")
				.get(0).get("CNT");
	}
}
```



###### 2、声明式事务

![image-20210130210331670](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210130210331670.png)



可以基于xml配置，也可以基于注解配置：

开启事务注解的⽅式

- 使用注解 @EnableTransactionManagement

- xml配置<tx:annotation-driven/>

配置

- proxyTargetClass
- mode
- order

**@Transactional** 

- transactionManager
- propagation
- isolation
- timeout
- readOnly

```java
@Component
public class FooServiceImpl implements FooService {
    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Override
    @Transactional
    public void insertRecord() {
        jdbcTemplate.execute("INSERT INTO FOO (BAR) VALUES ('AAA')");
    }

    @Override
    @Transactional(rollbackFor = RollbackException.class)
    public void insertThenRollback() throws RollbackException {
        jdbcTemplate.execute("INSERT INTO FOO (BAR) VALUES ('BBB')");
        throw new RollbackException();
    }

    @Override
    public void invokeInsertThenRollback() throws RollbackException {
        insertThenRollback();
    }
}
```

###### 3、事务的本质

**Spring** 的声明式事务本质上是通过 **AOP** 来增强了类的功能，**Spring** 的 **AOP** 本质上就是为类做了⼀个代理，看似在调⽤⾃⼰写的类，实际⽤的是增强后的代理类。

##### 五、spring的jdbc异常抽象

**Spring** 会将数据操作的异常转换为 **DataAccessException** ⽆论使⽤何种数据访问⽅式，都能使⽤⼀样的异常

![image-20210130211254578](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210130211254578.png)



通过 **SQLErrorCodeSQLExceptionTranslator** 解析错误码，**ErrorCode** 定义

- org/springframework/jdbc/support/sql-error-codes.xml

- Classpath 下的 sql-error-codes.xml

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202021-01-30%20%E4%B8%8B%E5%8D%889.20.45.png" alt="屏幕快照 2021-01-30 下午9.20.45" style="zoom:50%;" />

**自定一个错误码解析逻辑**：

classpath下的 sql-error-codes.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN 2.0//EN" "http://www.springframework.org/dtd/spring-beans-2.0.dtd">

<beans>

    <bean id="H2" class="org.springframework.jdbc.support.SQLErrorCodes">
        <property name="badSqlGrammarCodes">
            <value>42000,42001,42101,42102,42111,42112,42121,42122,42132</value>
        </property>
        <property name="duplicateKeyCodes">
            <value>23001,23505</value>
        </property>
        <property name="dataIntegrityViolationCodes">
            <value>22001,22003,22012,22018,22025,23000,23002,23003,23502,23503,23506,23507,23513</value>
        </property>
        <property name="dataAccessResourceFailureCodes">
            <value>90046,90100,90117,90121,90126</value>
        </property>
        <property name="cannotAcquireLockCodes">
            <value>50200</value>
        </property>
        <property name="customTranslations">
            <bean class="org.springframework.jdbc.support.CustomSQLErrorCodesTranslation">
                <property name="errorCodes" value="23001,23505" />
                <property name="exceptionClass"
                          value="geektime.spring.data.errorcodedemo.CustomDuplicatedKeyException" />
            </bean>
        </property>
    </bean>

</beans>

```

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class ErrorCodeDemoApplicationTests {
	@Autowired
	private JdbcTemplate jdbcTemplate;

	@Test(expected = CustomDuplicatedKeyException.class)
	public void testThrowingCustomException() {
		jdbcTemplate.execute("INSERT INTO FOO (ID, BAR) VALUES (1, 'a')");
		jdbcTemplate.execute("INSERT INTO FOO (ID, BAR) VALUES (1, 'b')");
	}
}
```



