#### WEB编码问题

##### 一、什么是URL编码

URL编码是一种浏览器用来打包表单输入的格式，浏览器从表单中获取所有的name和其对应的value，将他们以name/value编码方式作为URL的一部分或者分离的发送到服务器上。(URL编码非常关键，是需要指定编码方式的)

URL编码规则
每对name/value由&分开，每对来自表单的name/value用=分开。如果用户没有输入值的那个name依旧会出现不过就是没有值。URL编码是在字符ASCII码的十六进制数的前面加上%。例如\（她的十六进制数表示为5c）的URL编码就是%5c。

##### 二、乱码问题

Web应用URL组成如下：
[http://domain:port/contextPath/servletPath/pathInfo?queryString](http://domainport/)
其中各个部分含义如下：
Domain、Port：分别是域名和端口；
contextPath：应用上下文路径，默认为应用名称，可以通过应用服务器的相关配置进行修改，一般线上环境会修改成/，此时相当于contextPath为空；
servletPath：Servlet路径，一般在应用的web.xml文件中配置servlet-mapping；但由于现在的web应用一般都会用一些框架，比如Struts、Webwork等，此时各框架都会对此进行封装，会在另外的配置文件中进行设置；但原理都是一样的；
pathInfo：可以理解为最终接收用户请求的具体执行类，比如我们常说的Action；
queryString：get方式传入的请求参数；

**乱码问题是web开发过程中经常遇到的问题，主要原因就是URL中使用了非ASCII码造成服务器后台程序解析出现乱码的问题。 URL中最容易出现中文的地方就是在QueryString的参数值还有pathInfo中。**

浏览器里http请求的流程：
第一步：浏览器把URL经过编码送给服务器；
第二步：服务器把这些请求解码处理完毕之后将显示的内容进行编码发送给客户端浏览器；
第三步：浏览器按照指定的编码显示网页

##### 三、GET提交如何编码以及服务器如何解码以及乱码解决方案

对于GET方式，我们知道它的提交是将请求数据附加到URL后面作为参数，这样依赖乱码就会很容易出现，因为数据name和value很有可能就是传递的为非ASCII码。

当URL拼接后，浏览器对其进行encode，然后发送到服务器。

URL encode的字符一般都是非ASCII码字符，所以我们就能知道出现乱码主要是URL中附加了中文或特殊字符做成的，另一个要知道URL encode到底是以什么样的编码方式对字符进行编码的，其实这个编码方式是由浏览器决定的，不同的浏览器和同一浏览器的不同设置影响了URL的编码。所以很多网站的做法都是先把url里面的中文或特殊字符用 javascript做URL encode，然后再拼接url提交数据，也就是替浏览器做了URL encode，好处就是网站可以统一get方法提交数据的编码方式。

完成了URL encode之后URL就成了ASCII范围内的字符了，然后就以iso-8859-1的编码方式转换为二进制随着请求头一起发送出去。

到了服务器之后，首先服务器一般默认会先用iso-8859-1进行解码，服务器获取的数据都是ASCII范围内的请求头字符，其中请求URL里面带有参数数据，如果是中文或特殊字符，那么encode后的%XX（编码规则中的十六进制数）通过request.setCharacterEncoding()是不管用的（这个只适用于设置post提交的request body的编码而不是设置get方法提交的queryString的编码）。

这时候我们就能发现出现乱码的根本原因就是客户端一般是通过用UTF-8或GBK等对数据进行encode的，到了服务器却用iso-8859-1方式decoder显然不行。

**两种解决方案：**

（1）通过String类的getBytes方法进行编码转换
new String(request.getParameter(“name”).getBytes(“iso-8859-1”),“客户端编码方式”)

（2）在服务器xml代码中改配置信息：
<Connector port="8080"protocol="HTTP/1.1" maxThreads="150" connectionTimeout="20000"
redirectPort="8443"URIEncoding="客户端编码"/>

##### 四、POST提交如何编码以及服务器如何解码以及乱码解决方案

对于POST方式，表单中的参数值对是通过request body发送给服务器，此时浏览器会根据网页的ContentType("text/html; charset=utf-8")中指定的编码进行对表单中的数据进行编码，然后发给服务器。

在服务器端的程序中我们可以通过Request.setCharacterEncoding()设置编码，然后通过request.getParameter获得正确的数据。这里出现乱码可以通过Request.setCharacterEncoding()直接解决。