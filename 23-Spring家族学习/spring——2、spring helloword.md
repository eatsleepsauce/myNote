#### spring hello world 程序

##### 一、通过 spring initializer 生成骨架

https://start.spring.io/

http://localhost:8080/actuator/beans actuator监控工具



![屏幕快照 2021-01-29 下午10.23.09](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202021-01-29%20%E4%B8%8B%E5%8D%8810.23.09.png)





![屏幕快照 2021-01-29 下午10.24.36](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202021-01-29%20%E4%B8%8B%E5%8D%8810.24.36.png)



##### 二、pom依赖spring-boot-start-parent

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.4.2</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>geektime.spring.hello</groupId>
	<artifactId>hello-spring</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>hello-spring</name>
	<description>Demo project for Spring Boot</description>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```

不依赖 spring-boot-starter-parent

![image-20210129222843910](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210129222843910.png)





