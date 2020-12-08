##1、maven简介
Maven使用项目对象模型(POM Project  Object Model)的概念，可以通过一小段描述信息来管理项目的构建，报告和文档的软件项目管理工具。在Maven中每个项目都相当于一个对象，对象与对象之间的关系包含：依赖、继承和聚合。

##2、仓库
Maven仓库是基于简单文件系统存储的，集中化管理构件(java api资源)的一个服务。仓库中的任何一个构件都有其唯一的坐标，任何maven项目使用任何一个构件的方式都是相同的。

**Maven可以在某个位置统一存储所有的Maven项目共享的构件，这个统一的位置就是仓库，项目构建完毕后生成的构件可以安装（本地）或者部署（远程）到仓库中，供其它项目使用。**

**仓库分类：本地仓库和远程仓库**

![仓库分类.png](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145700.png)

**远程仓库**，不在本地的仓库都是远程仓库，而远程仓库又分为：中央仓库和私服仓库。远程仓库是指通过各种协议，如file://、http://或其它访问的仓库。这些仓库可能是第三方搭建的真实远程仓库（例如repo.maven.apache.org和uk.maven.org是maven的中央仓库），用来提供他们的构件下载。其它远程仓库可能是自己的公司的内部仓库（公司局域网内搭建的maven仓库，也就是私服仓库），用来在开发团队间共享私有构件和管理发布的。
默认的远程仓库使用的是apache提供的中央仓库：https://mvnrepository.com/
Maven必须要知道至少一个可用的远程仓库，中央仓库就是这样一个默认的远程仓库。

**私服**是一种特殊的远程Maven仓库，它是架设在局域网内的仓库服务，私服一般被配置为互联网远程仓库的镜像，供局域网内的Maven用户使用。当Maven需要下载构件的时候，先向私服请求，如果私服上不存在该构件，则从外部的远程仓库下载，同时缓存在私服之上，然后为Maven下载请求提供下载服务，另外，对于自定义或第三方的jar可以从本地上传到私服，供局域网内其他maven用户使用。

**本地仓库**，指的本地一份拷贝，包含缓存的远程下载以及自己生成的临时构件。

**仓库优先级（简单粗暴，未考虑存在多个远程库的情况）**，最先从本地仓库开始查询构件，如果没有查到则从远程仓库中查询，远程库分两种情况，一种是配置了镜像仓库，另一种是没有配置镜像仓库。
![仓库优先级.png](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145701.png)


##3、常用配置
**这些配置都是修改的 maven目录下conf/settings.xml文件**
**一、本地库配置**
本地仓库就是本地的一个目录，用于缓存远程库下载的构件以及自己本地生成的构件。
```
 <!-- 本地仓库配置 -->
  <localRepository>具体的本地仓库位置</localRepository>
```

**二、镜像仓库配置**
镜像的主要目的是替换原来远程仓库访问达到较快的访问速度，如阿里云的公共仓库替换中央仓库的访问。
```
 <mirror>
      <!-- 镜像的id，可以自己指定-->
      <id>aliyunmaven</id>
      <!-- 匹配模式  这里的central是默认中央仓库的id -->
      <mirrorOf>central</mirrorOf>
      <!-- 镜像名称，可以自己指定 -->
      <name>阿里云公共仓库</name>
      <!-- 镜像路径 -->
      <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```
>mirrorOf 标签里面放置的是 repository 配置的 id,为了满足一些复杂的需求，Maven还支持更高级的镜像配置：
external:\*        不在本地仓库的文件才从该镜像获取
repo,repo1      远程仓库 repo 和 repo1 从该镜像获取
\*,!repo1          所有远程仓库都从该镜像获取，除 repo1 远程仓库以外
\*                     所用远程仓库都从该镜像获取

**三、jdk配置**
指定编译和运行时的jdk，这里是全局性的，也可以在具体的maven项目中指定某个版本的jdk。
```
<profile>
        <id>jdk-1.8</id>
        <activation>
            <activeByDefault>true</activeByDefault>
            <jdk>1.8</jdk>
        </activation>
        <properties>
            <maven.compiler.source>1.8</maven.compiler.source>
            <maven.compiler.target>1.8</maven.compiler.target>
            <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
        </properties>
</profile>

```

##4、工程类型以及工程结构
```
    <groupId>com.test.ly</groupId>
    <artifactId>MavenDemo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <!-- 这里的还可以是 jar  war-->
    <packaging>pom</packaging>
```
**一、POM工程**
POM工程是逻辑工程，用在父级工程或聚合工程中，用来做jar包的版本控制。
```
    <modules>
        <module>SubModule</module>
        <module>SubWeb</module>
    </modules>


    <properties>
        <mybatis.version>3.5.4</mybatis.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis</artifactId>
                <version>${mybatis.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

**二、JAR工程**
将会打包成jar，用作jar包使用。

**三、WAR工程**
将会打包成war包，发布在应用服务器上的工程。

**四、Maven工程结构**
![项目结构.png](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145702.png)
*src/main/java*  这个目录下存储java源码
*src/main/resources*  存储主要的资源文件，比如xml配置文件以及properties文件
*src/test/java*  存储测试用的类，比如junit的测试类
*src/test/resources*  存储测试环境用的资源文件，一般需要自己创建
*target*  项目编译后内容存放位置
*pom.xml*  maven的基础配置文件，配置项目和项目之间的关系，包括配置依赖关系等

##5、工程关系：依赖、继承、聚合
**maven工程的关系都在工程中的pom.xml配置文件中体现。**
**一、依赖**
依赖很简单，就是说工程需要依赖什么构件(jar)，代码示例如下：
```
    <dependencies>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.4</version>
        </dependency>
    </dependencies>
```
**依赖具有传递性**，如果A工程依赖了B工程，而B工程依赖了xx.jar，那么xx.jar也会加入到A工程中，这个很好理解。

**依赖的两个冲突解决原则：**
**(1)最短路径优先原则**   举例：a->b->c->d(2.0版本)，a->e->d(1.0)  那么工程中生效的d是1.0的版本。
**(2)最先声明原则** 举例：先是a->b->e(1.0)，然后a->c->e(2.0)，那么生效的e是1.0版本。

**排除依赖**，有些情况下，我们不希望引入因为传递性引入的一些jar包，因为版本不可靠等原因，我们可以对具体的一些jar包进行排除不让它们被传递引入进来。
```
<dependencies>
        <dependency>
            <groupId>xxx.xx</groupId>
            <artifactId>xxx</artifactId>
            <version>1.0</version>
            <exclusions>
                <exclusion>
                    <groupId>yyy.yy</groupId>
                    <artifactId>yy</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
```
**依赖范围**，依赖的范围就是决定了依赖的构件什么时候生效，什么时候无效。
```
<dependencies>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.4</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
```
**(1)compile**  这是默认范围，表示该依赖在编译和运行是都生效。
**(2)provided**  已提供依赖范围。使用此依赖范围典型例子servlet-api，编译和测试项目的时候需要该依赖，但在运行项目的时候，由于容器已经提供，就不需要maven提供引入了。
**(3)runtime**  runtime范围表明编译时不需要生效，而只在运行时生效。典型的例子是jdbc驱动实现，项目主代码的编译只需要jdk提供的jdbc接口，只有在执行测试或则运行项目的时候才需要实现上述接口的具体jdbc驱动。
**(4)system**  与provided类似，不过必须显示指定一个本地系统路径的jar，此类依赖一直有效，maven不会去仓库中寻找它。但是，使用system范围依赖时，必须通过systemPath元素显示地指定依赖文件的路径。
**(5)test**  使用此范围的依赖，只在编译测试代码和运行测试代码的时候需要，应用的正常运行不需要此类依赖。典型的例子就是junit。
**(6)import**  用于pom文件中<dependencyManagement>部分，主要是强制子工程必须使用父工程中的指定版本。
```
 <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis</artifactId>
                <version>${mybatis.version}</version>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```


**二、继承**
如果A工程继承B工程，则代表A工程默认依赖B工程依赖的所有资源，被继承的工程一般只能是POM工程。
一般情况下POM工程都是将依赖放置到<dependencyManagement>中，在此标签中的依赖，只有在子工程明确依赖的时候才会生效，否则不能直接使用。如果直接使用<dependencies>则可以保证所有的子工程都继承依赖关系并且是依赖直接生效的，无需再添加依赖。
依赖内容放置在<dependencyManagement>中的主要目的是进行依赖的版本管理。

**三、聚合**
如果我们开发的工程拥有2个以上的module时，每个module都是一个独立的功能集合。这时候我们就需要一个聚合工程了。创建聚合工程的过程中，总工程必须是一个pom工程(jar和war类型的工程没有办法做聚合工程)，各个子模块可以是任意类型。聚合包含了继承的特性。

![总工程pom.png](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145703.png)

![子工程pom.png](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145704.png)


##6、常见插件
**一、编译器插件**
**二、资源插件**
**三、tomcat插件**

##7、常见命令
**install**    本地安装，包含编译、打包和安装到本地仓库
**compile**   仅编译
**package**  包含编译和打包
**deploy**   将包部署到远程仓库中
**clean**  清除已编译信息，删除工程中target目录
