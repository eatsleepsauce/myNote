URL encode 非常关键，是需要指定 编码方式的

1. 什么是URL编码。
URL编码是一种浏览器用来打包表单输入的格式，浏览器从表单中获取所有的name和其对应的value，将他们以name/value编码方式作为URL的一部分或者分离的发送到服务器上。
2. URL编码规则。
每对name/value由&分开，每对来自表单的name/value用=分开。如果用户没有输入值的那个name依旧会出现不过就是没有值。
URL编码是在字符ASCII码的十六进制数的前面加上%。例如\（她的十六进制数表示为5c）的URL编码就是%5c。
3. 简单介绍乱码和http请求
其实做web开发乱码问题是经常出现的，有了上面编码的基础之后下面来看看乱码。
1) 乱码问题是web开发过程中经常遇到的问题，主要原因就是URL中使用了非ASCII码造成服务器后台程序解析出现乱码的问题。
2) URL中最容易出现中文的地方就是在QueryString的参数值还有Servletpath中。
3) 简单用一个图来说明一下http请求的流程：
第一步：浏览器把URL经过编码送给服务器；
第二步：服务器把这些请求解码处理完毕之后将显示的内容进行编码发送给客户端浏览器；
第三步：浏览器按照指定的编码显示网页
4) 详细剖析GET提交如何编码以及服务器如何解码以及乱码解决方案
对于GET方式，我们知道它的提交是将请求数据附加到URL后面作为参数，这样依赖乱码就会很容易出现，因为数据name和value很有可能就是传递的为非ASCII码。
当URL拼接后，浏览器对其进行encode，然后发送到服务器。具体规则见URL编码规则。
这里详细说一下encode的过程中容易出现的问题，在这个过程中我们要明白需要URL encode的字符一般都是非ASCII码字符，所以我们就能知道出现乱码主要是URL中附加了中文或特殊字符做成的，另一个要知道URL encode到底是以什么样的编码方式对字符进行编码的，其实这个编码方式是由浏览器决定的，不同的浏览器和同一浏览器的不同设置影响了URL的编码，所以为了避免我们不需要的编码，我们可以通过java代码或javaspcript代码统一进行控制。
完成了URL encode之后URL就成了ASCII范围内的字符了，然后就以iso-8859-1的编码方式转换为二进制随着请求头一起发送出去。
到了服务器之后，首先服务器会先用iso-8859-1进行解码，服务器获取的数据都是ASCII范围内的请求头字符，其中请求URL里面带有参数数据，如果是中卫或特殊字符，那么encode后的%XY（编码规则中的十六进制数）通过request.setCharacterEncoding()是不管用的。这时候我们就能发现出现乱码的根本原因就是客户端一般是通过用UTF-8或GBK等对数据进行encode的，到了服务器却用iso-8859-1方式decoder显然不行。
这里的解决方式有两种，
一种：是通过String类的getBytes方法进行编码转换，具体java代码是：
new String(request.getParameter(“name”).getBytes(“iso-8859-1”),“客户端编码方式”)
第二种：在服务器xml代码中改配置信息：
<Connector port="8080"protocol="HTTP/1.1" maxThreads="150" connectionTimeout="20000"
redirectPort="8443"URIEncoding="客户端编码"/>
5) 详细剖析POST提交如何编码以及服务器如何解码以及乱码解决方案
对于POST方式，表单中的参数值对是通过request包发送给服务器，此时浏览器会根据网页的ContentType("text/html; charset=GBK")中指定的编码进行对表单中的数据进行编码，然后发给服务器。
在服务器端的程序中我们可以通过
Request.setCharacterEncoding()设置编码，然后通过
request.getParameter获得正确的数据。
这里出现乱码可以通过Request.setCharacterEncoding()直接解决。
