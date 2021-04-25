#### web后端知识

##### 一、servlet简介

- servlet是一种web服务器端的编程技术
- 由支持servlet的web服务器调用和启动运行（如：Tomcat）
- 一个servlet负责对应的一个或一组url访问请求，并返回相应的内容

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145739.png" alt="servlet.png" style="zoom: 67%;" />

###### 1、servlet实现使用

创建一个普通的java文件，继承HttpServlet，可以重写service方法，或重写doXX方法。在WEB-INFO下的web.xml中添加请求与servlet类的映射关系。

```
    <!--配置servlet的别名，同时在servlet-class配置项中添加servlet类的完全限定名  包名+类名-->
    <servlet>
        <servlet-name>myServlet</servlet-name>
        <servlet-class>com.xxx.MyServlet</servlet-class>
    </servlet>
    <!--配置servlet跟请求的映射关系-->
    <servlet-mapping>
        <servlet-name>myServlet</servlet-name>
        <url-pattern>/xx</url-pattern>
    </servlet-mapping>
```
###### 2、servlet的生命周期

servlet生命周期如上图的请求流程，首次请求时由容器创建servlet对象（配置了 load-on-startup的情况有所不同，不一定是请求时创建servlet对象了，有可能是容器启动的时候创建servlet对象，例如 Spring mvc的 DispatcherServlet），执行init()方法，然后调用service方法，容器销毁时会调用destroy方法，后续请求过来时只会执行service方法。init和destroy只会执行一次，init在servlet对象创建后执行（servlet对象在首次请求的时候创建，后面请求过来复用对象）。

##### 二、http协议简介

HTTP：超文本传输协议(Hyper Text Transfer Protocol)
作用：规范了浏览器和服务器的数据交互，还有其它...
特点：1、简单快速   2、灵活  3、无连接   4、无状态   5、支持B/S和C/S架构
**注意：HTTP1.1版本之后支持可持续连接**

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145740.png" alt="http1.1.png" style="zoom: 67%;" />

###### 1、http请求格式

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145741.png" alt="请求.png" style="zoom: 50%;" />

###### 2、http请求方法

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145742.png" alt="方法.png" style="zoom: 50%;" />

1、get请求参数是直接显示在地址栏的，而post在地址栏不显示
2、get方式不安全，post安全
3、get请求参数有长度限制，post没有限制

###### 3、http响应格式

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145743.png" alt="响应.png" style="zoom:50%;" />

###### 4、http响应状态码

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145744.png" alt="image.png" style="zoom:67%;" />

>200 OK //客户端请求成功
400 Bad Request //客户端请求有语法错误，不能被服务器所理解
401 Unauthorized //请求未经授权，这个状态代码必须和WWW-Authenticate 报头域一起使用
403 Forbidden //服务器收到请求，但是拒绝提供服务
404 Not Found //请求资源不存在，eg：输入了错误的 URL
500 Internal Server Error //服务器发生不可预期的错误
503 Server Unavailable //服务器当前不能处理客户端的请求，一段时间后可能恢复正常



##### 三、Tomcat原理简介

Tomcat服务器是一款轻量级应用服务器。我们正常开发完一个web应用后，编译打包后得到一个war包，这个war包放入Tomcat的应用程序路径下，启动Tomcat就可以通过http请求访问这个web应用了。

###### 1、Tomcat基本逻辑：

**(1)、Tomcat启动以后，其实在操作系统中看到的是一个JVM虚拟机进程。**这个虚拟机启动后，加载class进来执行，首先加载的是**org.apache.catalina.startup.Bootstrap类**，这个类里面有一个main()函数，是整个Tomcat的入口函数，JVM虚拟机会启动一个主线程从这个入口函数开始执行。

**(2)、Bootstrap的main()函数开始执行，初始化Tomcat运行环境。**这时候主要是创建一些线程，如负责监听80端口的线程，处理客户端连接请求的线程，以及执行用户请求的线程。创建这些线程的代码是Tomcat代码的一部分。

**(3)、初始化运行环境之后**，Tomcat就会扫描web程序路径，扫描到开发的war包后，再加载war包里面的类到JVM(因为web应用是被Tomcat加载运行的，所以也称Tomcat为Web容器)。

**(4)、外部请求发送到Tomcat**，也就说外部程序通过80端口和Tomcat进行http通信的时候，Tomcat会根据war包中的web.xml配置，决定这个请求url应该由哪个servlet处理，然后Tomcat就会分配一个线程去处理这个请求，实际上，就是这个线程执行响应的servlet代码。

###### 2、Tomcat的目录结构：

- /bin 存放启动和关闭tomcat的可执行文件
- /conf 存放tomcat的配置文件
- /lib 存放库文件
- /logs 存放日志文件
- /temp 存放临时文件
- /webapps 存放web应用
- /work 存放jsp转换后的servlet文件

###### 3、简单Tomcat实现：

**(1)、首先需要一个类似bootstrap的功能类**

```
public class MyServer {

    // 请求和servlet映射关系
    public static HashMap<String,String> mapping = new HashMap<String,String>();
    static {
        mapping.put("/test","com.xx.MyServlet");
    }

    // 容器启动入口
    public static void main(String[] args) {
        try {
            startServer(8888);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    /**
     * 定义服务端的接受程序，接受socket请求
     * @param port
     */
    public static void startServer(int port) throws Exception{
        //定义服务端套接字
        ServerSocket serverSocket = new ServerSocket(port);
        //定义客户端套接字
        Socket socket = null;

        while (true){
            socket = serverSocket.accept();

            //获取输入流和输出流
            InputStream inputStream = socket.getInputStream();
            OutputStream outputStream = socket.getOutputStream();

            //封装一个请求对象  --  参照http协议请求的格式
            MyRequest request = new MyRequest(inputStream);
            //封装一个响应对象 --  参照http协议响应的格式
            MyResponse response = new MyResponse(outputStream);

            //根据请求获取对应的处理类，即servlet（这里的映射关系简单的用了一个Map表示，真实情况是用web.xml配置的）
            String clazz = new mapping.get(request.getRequestUrl());
            if(clazz!=null){
                Class<MyServlet> myServletClass = (Class<MyServlet>)Class.forName(clazz);
                //根据myServletClass创建对象
                MyServlet myServlet = myServletClass.newInstance();
                myServlet.service(request,response);
            }

        }
    }

}
```
**(2)、request和response对象封装**
*request:*
```
public class MyRequest {

    //请求方法  GET/POST
    private String requestMethod;
    //请求地址
    private String requestUrl;

    public MyRequest(InputStream inputStream) throws Exception{
        //缓冲区域
        byte[] buffer = new byte[1024];
        //读取数据的长度
        int len = 0;
        //定义请求的变量
        String str = null;

        if((len = inputStream.read(buffer))>0){
            str = new String(buffer,0,len);
        }
        // GET / HTTP/1.1
       String data =  str.split("\n")[0];
       String[] params =  data.split(" ");
        this.requestMethod = params[0];
        this.requestUrl = params[1];
    }

    public String getRequestMethod() {
        return requestMethod;
    }

    public void setRequestMethod(String requestMethod) {
        this.requestMethod = requestMethod;
    }

    public String getRequestUrl() {
        return requestUrl;
    }

    public void setRequestUrl(String requestUrl) {
        this.requestUrl = requestUrl;
    }
}
```
*response:*
```
public class MyResponse {

    private OutputStream outputStream;

    public MyResponse(OutputStream outputStream) {
        this.outputStream = outputStream;
    }

    public void write(String str) throws  Exception{
        StringBuilder builder = new StringBuilder();
        builder.append("HTTP/1.1 200 OK\n")
                .append("Content-Type:text/html\n")
                .append("\r\n")
                .append("<html>")
                .append("<body>")
                .append("<h1>"+str+"</h1>")
                .append("</body>")
                .append("</html>");
        this.outputStream.write(builder.toString().getBytes());
        this.outputStream.flush();
        this.outputStream.close();
    }
}
```
**(3)、servlet相关封装**
**类似HttpServlet的类:**
```
public abstract class MyHttpServlet {

    //定义常量
    public static final String METHOD_GET = "GET";
    public static final String METHOD_POST = "POST";

    public abstract void doGet(MyRequest request,MyResponse response) throws  Exception;
    public abstract void doPost(MyRequest request,MyResponse response) throws  Exception;

    /**
     * 根据请求方式来判断调用哪种处理方法
     * @param request
     * @param response
     */
    public void service(MyRequest request,MyResponse response) throws Exception{
        if(METHOD_GET.equals(request.getRequestMethod())){
            doGet(request,response);
        }else if(METHOD_POST.equals(request.getRequestMethod())){
            doPost(request,response);
        }
    }
}
```
*定义的servlet：*
```
public class MyServlet extends MyHttpServlet{
    @Override
    public void doGet(MyRequest request, MyResponse response) throws Exception {
        response.write("mytomcat get test");
    }

    @Override
    public void doPost(MyRequest request, MyResponse response) throws Exception {
        response.write("mytomcat post test");
    }
}
```
**(4)、测试**
运行MyServer，浏览器中输入 “http://127.0.0.1:8888/test”，测试结果：
![测试结果.png](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145745.png)

##### 四、HttpServletRequest和HttpServletResponse

###### 1、HttpServletRequest

HttpServletRequest对象代表客户端的请求，当客户端通过http协议访问服务器时，http请求中的所有信息都封装在这个对象中，可以通过这个对象获取请求中的数据。
**常用方法：**

- getRequestURL:获取客户端的完整url
- getRequestURI:获取请求行中的资源名部分
- getQueryString:获取请求行的参数部分
- getMethod:获取请求方式
- getSchema:获取请求的协议
- getRemoteAddr:获取客户端的ip地址
- getRemoteHost:获取客户端的完整主机名
- getRemotePort:获取客户端的网络端口号

- 获取请求头信息
  getHeader(String name)
  getHeaders(String name)

- 获取客户端请求参数
  getParameter(name)
  getParameterValues(String name)
  getParameterNames()
  getParameterMap()

###### 2、HttpServletResponse

HttpServletResponse对象是服务器的响应对象，这个对象中封装了向客户端发送的数据、响应头以及响应状态码。
**常用方法：**

- 设置响应头
  setHeader(String key, String value) 添加响应信息，key相同则覆盖
  addHeader(String key, String value) 添加响应信息，key相同不会覆盖
- 设置响应头
  sendError(int num, String msg)  添加响应状态
- 设置响应实体
  getWriter().write(msg) 响应具体的数据

**(3)、乱码问题解决**
- get请求乱码：request.setCharacterEncoding("utf-8");在server.xml中添加属性useBodyEncodingForURI=true
- post请求乱码：request.setCharacterEncoding("utf-8");
- response乱码：response.setCharacterEncoding("utf-8");response.setContentType("text/html;charset=utf-8");

##### 五、servlet请求转发和重定向

###### 1、servlet请求转发 

request.getRequestDispatcher("servletname").forward(request, response);

- 用户只请求一次

- 浏览器地址栏不变化

- request和response对象只有一个，请求转发过程中servlet之间共享。

- 对客户端透明

- 只能跳转本站资源

  

  <img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145746.png" alt="请求转发.png" style="zoom:67%;" />


**不同的servlet之间共享数据可以通过request.setAttribute("","")和request.getAttribute("");实现。**

###### 2、重定向 

response.sendRedirect("url");

- 浏览器发送两次请求
- 浏览器地址栏发送变化
- 请求过程中产生新的request和response对象
- servlet之间不共享request和response对象

##### 六、Cookie简介

*http是一个无状态的协议，当一个客户端向服务端发送请求，在服务器返回响应后，连接就关闭了，在服务器端不保留连接信息。也就是请求A之后的请求B不知道请求A干了什么。*

**当客户端发送多次请求，且需要相同的请求参数时，这时候cookie就非常有用了。**比如，某段时间内免登录。
**cookie说明：**
- cookie是一种在客户端保存http状态信息的技术。

- cookie是在浏览器访问服务器的某个资源时，由web服务器在响应头传送给浏览器的数据。

- cookie只能记录一种信息，是key-value信息。

- 一个web站点可以给浏览器发送多个cookie，一个浏览器也可以存储多个站点的cookie。

  

  <img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145747.png" alt="cookie.png" style="zoom:50%;" />

**cookie基本操作：**

- cookie创建
```
  Cookie cookie = new Cookie(String key,String value);
```
- cookie设置
**cookie分两类，一类是临时的，另一类是持久化的，不设置最大年龄的是临时cookie，默认创建的cookie都是临时cookie，随着浏览器的关闭而消失。**
```
  // 设置了最大年龄的cookie是持久化的，如果不设置则默认保存在内存中，随着浏览器关闭而消失
  cookie.setMaxAge(int seconds);
  // 给cookie设置请求路径，只有请求了设置的路径是才会带上这个cookie信息给服务器
  cookie.setPath(String url);
```
- 响应增加cookie
```
response.addCookie(Cookie cookie);
```
- 获取cookie信息
```
Cookie[] cookies = request.getCookies();

if (cookies != null) { 
   for (Cookie  c: cookies) {
       String key = c.getName();
       //......  cookie 是key value的，这里省略一堆判断～～～
       c.getValue();
   }
}
```
##### 七、Session简介

*Session是一种在服务器端保存http状态信息的技术。*

- Session表示会话，在一段时间内，用户与服务器之间的一系列交互操作。
- session对象，用户发送不同请求的时候，在服务器端保存不同请求共享数据的存储对象。

###### 1、session说明

- session是依赖cookie技术的服务器端的数据存储技术

- 由服务端进行创建

- 每个用户独立拥有session对象

- 默认存储的时间是30分钟

  <img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145748.png" alt="session.png" style="zoom: 50%;" />

###### 2、session基本操作

- session创建和获取
```
HttpSession session = request.getSession();
```
- session设置
```
session.setMaxInactiveInterval(5);//设置存活时间，时间单位秒
session.invalidate(); //session强制失效
```
- session添加值和取值
```
sessoin.setAttribute(String name,Object object);

session.getAttribute(String name);
```

###### 3、session常见问题

- session创建时机
(1)、客户端第一次访问jsp文件，jsp被翻译成Servlet时会自动创建Session，此后客户端再次访问就会带着JSESSIONID过来。
(2)、当客户端重启浏览器时，客户端的JSESSIONID被销毁(此时服务端的Session没有受影响)，客户端再次访问浏览器没有带着JSESSIONID，服务端将再次为客户创建Session。
(3)、在jsp文件page指令里设置session="false"，客户端访问此jsp将不会创建Session。
(4)、客户端访问Servlet时不会创建Session，只有在通过request.getSession()或是跳转到jsp文件时才创建Session。

  **通过session对象的isNew()方法判断session是否为新创建的。**

- cookie禁用
cookie禁用后(这个操作不会影响服务端创建cookie添加到响应)，客户端每次请求都不会带jsessionid了，服务器端也无法识别客户端对应哪个session，也就说如果服务端在接收请求后，有创建session的逻辑则会一直创建session对象。

  **解决方法主要是URL重写：**
  response.encodeRedirectURL(java.lang.String url) 用于对sendRedirect方法后的url地址进行重写。
  response.encodeURL(java.lang.String url)用于对表单action和超链接的url地址进行重写

- 分布式session管理
网上有篇说得不错，可以参照： [https://www.cnblogs.com/saoyou/p/11107488.html](https://www.cnblogs.com/saoyou/p/11107488.html)
目前大致是这几种方法：
(1)、session复制
(2)、session会话保持（黏滞会话）会话保持是利用负载均衡的原地址Hash算法实现，负载均衡服务器总是将来源于同一IP的请求分发到同一台服务器上
(3)、利用cookie记录session
(4)、session服务器（集群）利用分布式缓存，数据库等，在这些产品的基础上进行包装，使其符合session的存储和访问要求

##### 八、ServletContext和ServletConfig

###### 1、ServletContext 

**ServletContext说明：**
- ServletContext 对象由服务器进行创建， 一个项目只有一个对象。 不管在项目的任意位置进行获取得到的都是同一个对象， 那么不同用户发起的请求获取到的也就是同一个对象了， 该对象由用户共同拥有。
- 运行在JVM上的每一个web应用程序都有一个与之对应的Servlet上下文（Servlet运行环境）
- Servlet API提供ServletContext接口用来表示Servlet上下文，ServletContext对象可以被web应用程序中的所有servlet访问
- ServletContext对象是web服务器中的一个已知路径的根

**ServletContext对象基本操作：**
- servlet中ServletContext对象的获取
```
ServletContext context = this.getServletContext();
//其它几种方式
this.getServletConfig().getServletContext();
request.getSession().getServletContext();
```
- 获取上下文初始化参数
```
String getInitParameter(String paramName);
```
**上下文参数在web.xml中设置**
```
 <context-param>
      <param-name>myparam</param-name>
      <param-value>xxxx</param-value>
 </context-param>
```
- 公共属性设置、获取以及删除
```
context.setAttribute("name","value");
context.getAttribute("name");
context.removeAttribute("name");
```
- 获取web程序的上下文路径
```
context.getContextPath();
```
- 获取资源在服务器上的真实路径
```
context.getRealPath("web.xml");
```

###### 2、ServletConfig

**ServletConfig说明：**
*使用ServletContext对象可以获取web.xml中的全局配置文件，在web.xml中，每个Servlet也可以进行单独的配置。*
```
<servlet>
        <servlet-name>ServletConfigServlet</servlet-name>
        <servlet-class>com.xx.ServletConfigServlet</servlet-class>
        <init-param>
            <param-name>paramName1</param-name>
            <param-value>value1</param-value>
        </init-param>
        <init-param>
            <param-name>paramName2</param-name>
            <param-value>value2</param-value>
        </init-param>
</servlet>
```
**ServletConfig基本操作：**
- 获取ServletConfig对象
```
ServletConfig config = this.getServletConfig();
```
- 获取初始化的参数
```
config.getInitParameter("paramName");

//获取所有的key
Enumeration<String> initParameterNames = config.getInitParameterNames();
while (initParameterNames.hasMoreElements()){
    String key = initParameterNames.nextElement();
    String value = config.getInitParameter(key);
}
```

##### 九、过滤器

**过滤器是能对web请求和web响应的头属性和内容进行操作的一种特殊web组件。**过滤器本身并不直接生成web响应，而是拦截web请求和响应，以便查看、提取或以某种方式操作客户端和服务器之间交换的数据。
**过滤器主要功能：**

- 分析web请求，对输入数据进行预处理
- 阻止web请求和响应进行
- 根据功能改动请求的头信息和数据体
- 与其它web资源协作

**filter过滤器的使用：**
(1)、定义filter过滤器需要实现javax.servlet.Filter接口，该接口定义了init()、doFilter()和destory()三个方法。这三个方法分别对应了过滤器生命周期中的初始化、过滤和销毁这三个阶段。
```
public class MyFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("filter init");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("filter逻辑处理开始");
        //  ......过滤逻辑
        // 过滤链往下走，最后执行的是被过滤的servlet
        filterChain.doFilter(servletRequest,servletResponse);
        // ......过滤逻辑
        System.out.println("filter逻辑处理完成");
    }

    @Override
    public void destroy() {
        System.out.println("我是filter销毁功能");
    }
}
```
(2)、web.xml配置
```
<filter>
        <filter-name>filter</filter-name>
        <filter-class>com.xx.filter.MyFilter</filter-class>
</filter>
<filter-mapping>
        <filter-name>filter</filter-name>
        <url-pattern>/*</url-pattern>
</filter-mapping>
<!-- 可配置多个filter -->
```
**应用场景：**编码格式、登录拦截等

##### 十、监听器

**Servlet监听器用于监听一些重要事件的发生，监听器可以在事情发生前、发生后可以做一些必要的处理。**监听器可以通过实现Servlet API提供的Listense接口来创建。

**监听对象**
- ServletContext对象——监听ServletContext对象，可以使web应用得知web组件的加载和下载情况
- HttpSession对象——监听HttpSession对象，可以使web应用了解会话期间的状态并作出反映
- ServletRequest对象——监听ServletRequest对象，可以使web应用控制web请求

**Servlet API 提供的一些监听器接口**

- ServletContext相关的监听器接口：
(1)、ServletContextListener  提供了contextInitialized方法和contextDestroyed方法，分别对应了ServletContext的创建和销毁事件，也就是服务器启动和关闭的事件。
(2)、ServletContextAttributeListener 提供了attributeAdded方法、attributeRemoved方法和attributeReplaced方法，分别对应了ServletContext中属性的添加、删除和替换事件。

- HttpSession相关的监听器接口：
(1)、HttpSessionListener 提供了sessionCreated方法和sessionDestroyed方法，分别对应了session对象的创建和销毁事件。
(2)、HttpSessionAttributeListener 提供了attributeAdded方法、attributeRemoved方法和attributeReplaced方法，分别对应了session中属性的添加、删除和替换事件。
(3)、HttpSessionActivationListener、HttpSessionBindingListener等不常用~~~

- ServletRequest相关等监听器接口：
(1)、ServletRequestListener 提供了requestInitialized方法和requestDestroyed方法，分别对应了request对象的创建和销毁事件。
(2)、HttpSessionAttributeListener 提供了attributeAdded方法、attributeRemoved方法和attributeReplaced方法，分别对应了request中属性的添加、删除和替换事件。

**监听器web.xml配置**
```
<listener>
        <listener-class>com.xx.listener.MyListener</listener-class>
</listener>
```
