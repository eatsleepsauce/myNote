#### spring mvc 及上下文

##### 一、DispatcherServlet 和常用注解

DispatcherServlet

- Controller
- xxxResolver
  - ViewResolver
  - HandlerExceptionResolver
  - MultipartResolver
- HandlerMapping

常用注解

- @Controller 
  - @RestController

- @RequestMapping
  - @GetMapping / @PostMapping
  - @PutMapping / @DeleteMapping

- @RequestBody / @ResponseBody / @ResponseStatus



##### 二、spring应用上下文

**上下文常用的接口及其实现**：

- BeanFactory
  - DefaultListableBeanFactory

- ApplicationContext  也是扩展了 BeanFactory
  - ClassPathXmlApplicationContext  在classpath寻找xml配置文件构建上下文
  - FileSystemXmlApplicationContext  在文件系统中寻找xml配置文件构建上下文
  - AnnotationConfifigApplicationContext  基于注解的方式构建上下文

- WebApplicationContext

**Web上下文层次：**

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210221120017159.png" alt="image-20210221120017159" style="zoom: 67%;" />

**注意 ：DispatcherServlet的 Servlet WebApplicationContext 的上下文 继承了 root 上下文。**

```xml
<web-app>

<listener>
  <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
<context-param>
	<param-name>contextConfigLocation</param-name>
	<param-value>/WEB-INF/app-context.xml</param-value>
</context-param>

<servlet>
  <servlet-name>app</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <init-param>
  	<param-name>contextConfigLocation</param-name>
  	<param-value></param-value>
  </init-param>
  <load-on-startup>1</Load-on-startup>
</servlet>

<servlet-mapping>
	<servlet-name>app</servlet-name>
	<url-pattern>/app/*</url-pattern>
</servlet-mapping>

</web-app>
```



##### 三、spring mvc的请求处理流程

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210221122138501.png" alt="image-20210221122138501" style="zoom:67%;" />

Front controller 就是 DispatcherServlet

（1）绑定⼀些 **Attribute** 

WebApplicationContext / LocaleResolver / ThemeResolver

（2）处理 **Multipart** 

如果是，则将请求转为 MultipartHttpServletRequest

（3）**Handler** 处理

如果找到对应 Handler，执⾏ Controller 及前后置处理器逻辑

（4）处理返回的 **Model** ，呈现视图



##### 四、定义映射关系

**@Controller** 

**@RequestMapping**  ，有参数：

- path / method 指定映射路径与⽅法
- params / headers 限定映射范围
- consumes / produces 限定请求与响应格式

**⼀些快捷⽅式**

@RestController ，带有 responsebody

@GetMapping / @PostMapping / @PutMapping / @DeleteMapping / @PatchMapping，@RequestMapping的快捷

**其他一些注解和通用：**

@RequestBody / @ResponseBody / @ResponseStatus

@PathVariable / @RequestParam / @RequestHeader

HttpEntity / ResponseEntity

示例代码：

```java
@PostMapping(path="/",consumes = MediaType.APPLICATION_JSON_VALUE,
		produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
@ResponseStatus(HttpStatus.CREATED)
public CoffeeOrder create(@RequestBody NewOrderRequest newOrder){
	Log.info("Receive new Order {}",newOrder);
	Coffee[] coffeeList= coffeeservice.getCoffeeByName(newOrder.getItems())
		.toArray(new Coffee[] {});
	return orderService.create0rder(newOrder.getCustomer(),coffeeList);
}

```

```java
@RequestMapping(path ="/{id}",method= RequestMethod.GET,
		produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
@ResponseBody
public Coffee getById(@PathVariable Long id){
  Coffee coffee = coffeeService.getCoffee(id); 
  return coffee;
}

@GetMapping(path = "/",params = "name")
@ResponseBody
public Coffee getByName(@RequestParam String name){
	return coffeeService.getCoffee(name);
}

```



##### 五、转换、校验 和 Multipart上传

**转换**，⾃⼰实现 WebMvcConfigurer， Spring Boot 在 WebMvcAutoConfiguration 中实现了⼀个

- 添加⾃定义的 Converter
- 添加⾃定义的 Formatter

**校验**，通过 Validator 对绑定结果进⾏校验，Hibernate Validator

- @Valid 注解

- BindingResult

**上传**，配置 MultipartResolver，Spring Boot ⾃动配置 MultipartAutoConfiguration

- ⽀持类型 multipart/form-data

- MultipartFile 类型



##### 六、视图解析机制以及常用的视图

**相关的 ViewResolver** 与 **View** 接⼝

- AbstractCachingViewResolver
- UrlBasedViewResolver
- FreeMarkerViewResolver
- ContentNegotiatingViewResolver
- InternalResourceViewResolver

**DispatcherServlet 中的视图解析逻辑**

**（1）使用modelAndView的逻辑**

- initStrategies()， initViewResolvers() 初始化了对应 ViewResolver

- doDispatch()，processDispatchResult()
  - 没有返回视图的话，尝试 RequestToViewNameTranslator
  - resolveViewName() 解析 View 对象

**（2）使用 @ResponseBody的逻辑**

- 在 HandlerAdapter.handle() 的中完成了 Response 输出
  -  RequestMappingHandlerAdapter.invokeHandlerMethod()
    -  HandlerMethodReturnValueHandlerComposite.handleReturnValue()
      -  RequestResponseBodyMethodProcessor.handleReturnValue()

**常用的视图**

官方：https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/web.html#mvc-view

- Jackson-based JSON / XML  
- Thymeleaf & FreeMarker

配置 MessageConverter，通过 WebMvcConfigurer 的 configureMessageConverters()，Spring Boot ⾃动查找 HttpMessageConverters 进⾏注册。

```java
public class WebConfiguration implements WebMvcConfigurer {

  @Override
  public void configureMessageConverters(List<HttpMessageConverter<?>> converters){ 						Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder()
  				.indentOutput(true)
  				.dateFormat(new SimpleDateFormat("yyyy-MM-dd"))
  				.modulesToInstall(new ParameterNamesModule());
  		converters.add(new MappingJackson2HttpMessageConverter(builder.build())); 							converters.add(new
 			MappingJackson2XmlHttpMessageConverter(builder.createXmlMapper(true).build()));
  }
}

```

Spring Boot 对 Jackson 的⽀持：

- JacksonAutoConfiguration
  - Spring Boot 通过 @JsonComponent 注册 JSON 序列化组件
  - Jackson2ObjectMapperBuilderCustomizer

- JacksonHttpMessageConvertersConfiguration
  - 增加 jackson-dataformat-xml 以⽀持 XML 序列化

##### 七、静态资源与缓存

**Spring Boot 中的静态资源配置**，核⼼逻辑WebMvcConfigurer.addResourceHandlers()

常⽤配置

- spring.mvc.static-path-pattern=/** 
- spring.resources.static-locations=classpath:/META-INF/

resources/,classpath:/resources/,classpath:/static/,classpath:/public/

**Spring Boot 中的缓存配置**，常⽤配置（默认时间单位都是秒）

ResourceProperties.Cache

- spring.resources.cache.cachecontrol.max-age=时间
- spring.resources.cache.cachecontrol.no-cache=true/false
- spring.resources.cache.cachecontrol.s-max-age=时间

**注意：一般情况下，不在应用中做这些静态资源和缓存的控制。**

**建议的资源访问⽅式：**

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210221210325168.png" alt="image-20210221210325168" style="zoom: 50%;" />

##### 八、异常处理

核⼼接⼝

- HandlerExceptionResolver

实现类

- SimpleMappingExceptionResolver
- DefaultHandlerExceptionResolver
- ResponseStatusExceptionResolver
- ExceptionHandlerExceptionResolver

处理⽅法

- @ExceptionHandler

添加位置

- @Controller / @RestController
- @ControllerAdvice / @RestControllerAdvice

##### 九、拦截器

**实现核⼼接⼝， HandlerInteceptor**，方法：

- boolean preHandle()
- void postHandle()
- void afterCompletion()

针对 **@ResponseBody** 和 **ResponseEntity** 的情况，使用 ResponseBodyAdvice

针对异步请求的接⼝，使用 AsyncHandlerInterceptor，方法 void afterConcurrentHandlingStarted()

**拦截器的配置方式**：

常规⽅法

- WebMvcConfigurer.addInterceptors()

**Spring Boot** 中的配置

- 创建⼀个带 @Configuration 实现 WebMvcConfigurer接口的配置类

- 不能带 @EnableWebMvc（想彻底⾃⼰控制 MVC 配置除外）