#### Java SPI机制详解

##### 一、什么是SPI？

SPI 全称为 (Service Provider Interface) ，是JDK内置的一种服务提供发现机制。SPI接口的定义在调用方，在概念上更依赖调用方；组织上位于调用方所在的包中，实现位于独立的服务提供包中。

当服务的提供者提供了一种接口的实现之后，需要在classpath下的META-INF/services/目录里创建一个以服务接口命名的文件，而文件里的内容就是这个接口的具体的实现类。当调用方程序需要这个服务的时候，就可以通过服务接口查找jar包（一般都是以jar包做依赖）的META-INF/services/中的配置文件，配置文件中有接口的具体实现类名，可以根据这个类名进行加载实例化，就可以使用该服务了。

##### 二、SPI简单使用

###### 1、接口定义

```java
package com.xx.spi;
public interface SPIService {
    void execute();
}
```

###### 2、接口实现

```java
package com.xx.spi.impl;
public class SPIServiceImpl1 implements SPIService{
    public void execute() {
        System.out.println("SPIServiceImpl1.execute()");
    }
}

package com.xx.spi.impl;
public class SPIServiceImpl2 implements SPIService{
    public void execute() {
        System.out.println("SPIServiceImpl2.execute()");
    }
}
```

###### 3、配置文件及内容

![image-20210107120837585](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210107120837585.png)

**本地测试的时候一定要保证配置文件在classpath里面，非常重要！**

```
com.xx.spi.impl.SPIServiceImpl1
com.xx.spi.impl.SPIServiceImpl2
```

Maven项目如何将自定义文件添加到META-INF目录下：

（1）在src/main/resources（必须是资源文件夹）下创建META-INF文件夹，然后将文件放在META-INF下。

（2）使用maven-jar-plugin插件，不让Maven打包时生成maven自己的描述文件，这样就maven就不会覆盖自定义的文件了。

```xml
<build>  
    <plugins>  
        <plugin>  
            <groupId>org.apache.maven.plugins</groupId>  
            <artifactId>maven-jar-plugin</artifactId>  
            <configuration>  
                <archive>  
                    <addMavenDescriptor>false</addMavenDescriptor>  
                </archive>  
            </configuration>  
        </plugin>  
    </plugins>  
</build> 
```

###### 4、调用

我们可以通过 ServiceLoader.load 或者 Service.providers 方法拿到实现类的实例

```java
public class Test {
    public static void main(String[] args) {   
        // Service.providers 方式获取 
        Iterator<SPIService> providers = Service.providers(SPIService.class);       
        while(providers.hasNext()) {
            SPIService next = providers.next();
            next.execute();
        }
        
        // ServiceLoader.load 方式获取       
        ServiceLoader<SPIService> load = ServiceLoader.load(SPIService.class);
        Iterator<SPIService> iterator = load.iterator();
        while(iterator.hasNext()) {
            SPIService next = iterator.next();
            next.execute();
        }
    }
}
```

##### 三、分析 ServiceLoader 原理

###### 1、ServiceLoader基本结构

```java
public final class ServiceLoader<S>
    implements Iterable<S>{
    // 加载文件目录
    private static final String PREFIX = "META-INF/services/"; 
    // 需要加载的服务类或接口
    private final Class<S> service; 
    // 类加载器
    private final ClassLoader loader; 
    private final AccessControlContext acc; 
    // 已加载的服务类集合
    private LinkedHashMap<String,S> providers = new LinkedHashMap<>(); 
    // 内部类，真正加载服务类 惰性迭代器
    private LazyIterator lookupIterator;
   
    //...
}

```

###### 2、load方法初始化话服务加载器ServiceLoader

```java
/**
* 服务加载程序
* @param  <S> 需要发现的服务类型 SPIService
* @param  service 需要发现服务接口或抽象类 SPIService.class
*/    
public static <S> ServiceLoader<S> load(Class<S> service) {
    //获取当前线程上下文类加载器
    ClassLoader cl = Thread.currentThread().getContextClassLoader();
    return ServiceLoader.load(service, cl);
}

/**
* 服务加载程序
* @param  <S> 需要发现的服务类型 SPIService
* @param  service 需要发现服务接口或抽象类 SPIService.class
* @param  loader 需要使用的类加载器
*/  
public static <S> ServiceLoader<S> load(Class<S> service, ClassLoader loader){
    return new ServiceLoader<>(service, loader);
}
 
/**
* 初始化服务加载器
*/
private ServiceLoader(Class<S> svc, ClassLoader cl) {
    //设置需要加载的服务类，判断是否为空
    service = Objects.requireNonNull(svc, "Service interface cannot be null");
    //设置加载器，为空则使用默认AppClassLoader加载器
    loader = (cl == null) ? ClassLoader.getSystemClassLoader() : cl;
    //设置加载器访问上下文，无权写返回null
    acc = (System.getSecurityManager() != null) ? AccessController.getContext() : null;
    //重置加载器
    reload();
}

/**
* 清除当前已加载的服务，并初始化延时迭代器
*/
public void reload() {
    providers.clear();
    lookupIterator = new LazyIterator(service, loader);
}
```

###### 3、iterator方法获取服务迭代器

```java
/**
* 获取已加载服务迭代器，默认为空，通过hasNext方法加载
*/
public Iterator<S> iterator() {
    return new Iterator<S>() {
        // 获取当前已加载服务迭代器
        Iterator<Map.Entry<String,S>> knownProviders = providers.entrySet().iterator();
        // 重写Iterator.hasNext方法,判断已加载迭代器中是否存在下一个，存在则返回true, 否则返回待加载迭代器中是否存在下一个
        public boolean hasNext() {
            if (knownProviders.hasNext())
                return true;
            return lookupIterator.hasNext();
        }
        // 重写Iterator.next方法,判断已加载迭代器中是否存在下一个，存在则返回这个实现类,否则返回待加载迭代器中的下一个，为空抛出异常
        public S next() {
            if (knownProviders.hasNext())
               return knownProviders.next().getValue();
            return lookupIterator.next();
        }
        //重写方法,不允许溢出
        public void remove() {
            throw new UnsupportedOperationException();
        }
    }
}
```

###### 4、延迟迭代器 LazyIterator 

```java
//ServiceLoader的私有类,服务迭代器的hasNext和next最终都是调用这个延迟迭代器的方法
private class LazyIterator implements Iterator<S> {
    Class<S> service;// 提供者服务类    
    ClassLoader loader;// 当前服务加载器    
    Enumeration<URL> configs = null;// 配置文件地址
    Iterator<String> pending = null;// 待加载迭代器
    String nextName = null;// 服务类名
 
    private LazyIterator(Class<S> service, ClassLoader loader) {
        this.service = service;
        this.loader = loader;
    }
 
    private boolean hasNextService() {
        if (nextName != null) {
            return true;
        }
        if (configs == null) {
            try {
                //META-INF/services/ 加上接口的全限定类名，就是文件服务类的文件
                String fullName = PREFIX + service.getName();
                // 将文件路径转成URL对象
                if (loader == null)
                    configs = ClassLoader.getSystemResources(fullName);
                else
                    configs = loader.getResources(fullName);
            } catch (IOException x) {
                fail(service, "Error locating configuration files", x);
            }
        }
        while ((pending == null) || !pending.hasNext()) {
            if (!configs.hasMoreElements()) {
                return false;
            }
            // 解析URL文件对象，读取内容，最后返回
            pending = parse(service, configs.nextElement());
        }
        // 待加载迭代中的下一个
        nextName = pending.next();
        return true;
    }
 
    private S nextService() {
        if (!hasNextService())
            throw new NoSuchElementException();        
        String cn = nextName;
        nextName = null;
        Class<?> c = null;
        try {
            // 加载服务实现类
            c = Class.forName(cn, false, loader);
        } catch (ClassNotFoundException x) {
            fail(service,"Provider " + cn + " not found");
        }
        // 判断是否为实现类是否为服务接口的子类
        if (!service.isAssignableFrom(c)) {
            fail(service, "Provider " + cn  + " not a subtype");
        }
        try {
            // 实例化服务实现类
            S p = service.cast(c.newInstance());
            // 放入已加载服务集合
            providers.put(cn, p);
            return p;
        } catch (Throwable x) {
            fail(service, "Provider " + cn + " could not be instantiated", x);
        }
        throw new Error();  
    }
    // 是否存在下一个服务
    public boolean hasNext() {
        if (acc == null) {
            return hasNextService();
        } else {
            PrivilegedAction<Boolean> action = new PrivilegedAction<Boolean>() {
                public Boolean run() { return hasNextService(); }
            };
            return AccessController.doPrivileged(action, acc);
        }
    }
    // 读取下一个服务
    public S next() {
        if (acc == null) {
            return nextService();
        } else {
            PrivilegedAction<S> action = new PrivilegedAction<S>() {
                public S run() { return nextService(); }
            };
            return AccessController.doPrivileged(action, acc);
        }
    }
    // 不允许移除服务
    public void remove() {
        throw new UnsupportedOperationException();
    }
 
}
```



##### 四、SPI的用途

JDBC的DriverManager、Spring、ConfigurableBeanFactory等都用到了SPI机制，以DriverManager为例，看一下其实现的内幕。

JDBC获取数据库连接的过程。在早期版本中，需要先设置数据库驱动的连接，再通过DriverManager.getConnection获取一个Connection。

```java
String url = "jdbc:mysql:///consult?serverTimezone=UTC";
String user = "root";
String password = "root";

Class.forName("com.mysql.jdbc.Driver");
Connection connection = DriverManager.getConnection(url, user, password);
```

有了SPI后，设置数据库驱动连接，这一步骤就不再需要。Class.forName("com.mysql.jdbc.Driver");不再需要。DriverManager在getConnection之前会执行静态块中的初始化代码。

```java
public class DriverManager {
    static {
        loadInitialDrivers();
        println("JDBC DriverManager initialized");
    }
}
```
而静态块中执行的就是SPI发现服务提供方并创建它的对象。

```java
public class DriverManager {
    private static void loadInitialDrivers() {
        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            public Void run() {
                //很明显，它要加载Driver接口的服务类，Driver接口的包为:java.sql.Driver
                //所以它要找的就是META-INF/services/java.sql.Driver文件
                ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
                Iterator<Driver> driversIterator = loadedDrivers.iterator();
                try{
                    //查到之后创建对象
                    while(driversIterator.hasNext()) {
                        driversIterator.next();
                    }
                } catch(Throwable t) {
                    // Do nothing
                }
                return null;
            }
        });
    }
}
```

当调用driversIterator.next方法时，就会创建这个类的实例。而创建示例前这个类又完成了一件事，向DriverManager注册自身的实例。

```java
public class Driver extends NonRegisteringDriver implements java.sql.Driver {
    static {
        try {
            //调用DriverManager类的注册方法往registeredDrivers集合中加入实例
            java.sql.DriverManager.registerDriver(new Driver());
        } catch (SQLException E) {
            throw new RuntimeException("Can't register driver!");
        }
    }
    public Driver() throws SQLException {
        // Required for Class.forName().newInstance()
    }
}
```

DriverManager.getConnection()方法就是创建连接的地方，通过遍历已注册的数据库驱动程序，调用其connect方法，获取连接并返回。

```java
private static Connection getConnection(
        String url, java.util.Properties info, Class<?> caller) throws SQLException {   
    //registeredDrivers中就包含com.mysql.cj.jdbc.Driver实例
    for(DriverInfo aDriver : registeredDrivers) {
        if(isDriverAllowed(aDriver.driver, callerCL)) {
            try {
                //调用connect方法创建连接
                Connection con = aDriver.driver.connect(url, info);
                if (con != null) {
                    return (con);
                }
            }catch (SQLException ex) {
                if (reason == null) {
                    reason = ex;
                }
            }
        } else {
            println("skipping:" + aDriver.getClass().getName());
        }
    }
}
```



