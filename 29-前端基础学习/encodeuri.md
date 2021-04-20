# [encodeURI和 encodeURIComponent 的作用及应用](https://www.cnblogs.com/jasonxu19900827/p/5286790.html)

首先解释下 encodeURIComponent 的作用：将文本字符串编码为一个有效的统一资源标识符 (URI)。
为什么要用这个是因为我想把 username 整个当做参数传递给 CGI, 而不让 CGI 将 username 分割掉。这话听不明白的话我换种方式来说，如果 username = 'a&foo=boo' 而不用 encodeURIComponent 的话，整个参数就成了 name=a&foo=boo, 这样 CGI 就获得两个参数 name 和 foo. 这不是我们想要的。

Javascript 里还有个同样功能的函数 encodeURI, 但是此方法不会对下列字符进行编码：":"、"/"、";" 和 "?"。

再来解析下他们的应用：

js 对文字进行编码涉及3个函数：escape,encodeURI,encodeURIComponent，相应3个解码函数：unescape,decodeURI,decodeURIComponent

1、 传递参数时需要使用encodeURIComponent，这样组合的url才不会被#等特殊字符截断。

例如：<script language="javascript">document.write('<a href="http://passport.baidu.com/?logout&aid=7& u='+encodeURIComponent("http://cang.baidu.com/bruce42")+'">退出</a& gt;');</script>

2、 进行url跳转时可以整体使用encodeURI

例如：Location.href="/encodeURI"("http://cang.baidu.com/do/s?word=百度&ct=21");

3、 js使用数据时可以使用escape

例如：搜藏中history纪录。

4、 escape对0-255以外的unicode值进行编码时输出%u****格式，其它情况下escape，encodeURI，encodeURIComponent编码结果相同。

最多使用的应为encodeURIComponent，它是将中文、韩文等特殊字符转换成utf-8格式的url编码，所以如果给后台传递参数需要使用encodeURIComponent时需要后台解码对utf-8支持（form中的编码方式和当前页面编码方式相同）

escape不编码字符有69个：*，+，-，.，/，@，_，0-9，a-z，A-Z

encodeURI不编码字符有82个：!，#，$，&，'，(，)，*，+，,，-，.，/，:，;，=，?，@，_，~，0-9，a-z，A-Z

encodeURIComponent不编码字符有71个：!， '，(，)，*，-，.，_，~，0-9，a-z，A-Z

在介绍encodeURI()、encodeURIComponent()、decodeURI()、decodeURIComponent()方法前我们需要了解Global对象的概念:
   Global（全局）对象可以说是ECMAScript中最特别的一个对象了，因为不管你从什么角度上看，这个对象都是不存在的。ECMAScript中的Global对象在某种意义上是作为一个终极的“兜底儿对象”来定义的。换句话说，不属于任何其他对象的属性和方法，最终都是它的属性和方法。事实上，没有全局变量或全局函数；所有在全局作用域中定义的属性和函数，都是Global对象的属性。本书前面介绍过的那些函数，诸如isNaN()、isFinite()、parselnt()以及parseFloat()，实际上全都是Global对象昀方法。除此之外，Global对象还包含其他一些方法。
   URI编码方法
    Global对象的encodeURI()和encodeURIComponent()方法可以对URI (Uniform ResourceIdentifiers,通用资源标识符)进行编码，以便发送给浏览器。有效的URI中不能包含某些字符，例如空格。而这URI编码方法就可以对URI进行编码，它们用特殊的UTF-8编码替换所有无效的字 符，从而让浏览器能够接受和理解。
   其中encodeURI()主要用于整个URI(例如，http://www.jxbh.cn/illegal value.htm)，而encode-URIComponent()主要用于对URI中的某一段(例如前面URI中的illegal value．htm)进行编码。它们的主要区别在于，encodeURI()不会对本身属于URI的特殊字符进行编码，例如冒号、正斜杠、问号和井字号；而encodeURIComponent()则会对它发现的任何非标准字符进行编码。来看下面的例子：
       var uri="http://www.jxbh.cn/illegal value.htm#start";
      //”http: //www.jxbh.cn/illegal%20value .htm#s tart”
      alert(encodeURI (uri)):
      //”http% 3A%2F%2Fwww.jxbh.cn%2 Fillegal%2 0value. htm%23 start”
      alert( encodaURIComponent (uri));
   使用encodeURI()编码后的结果是除了空格之外的其他字符都原封不动，只有空格被替换成了%20。而encodeURIComponent()方法则会使用对应的编码替换所有非字母数字字符。这也正是可以对整个URI使用encodeURI()，而只能对附加在现有URI后面的字符串使用encodeURIComponent()的原因所在。一般来说,我们使用encodeURIComponent()方法的时候要比使用encodeURI()更多,因为在实践中更常见的是对查询字符串参数而不是对基础URL进行编码.
   与encodeURI()和encodeURIComponent()方法对应的两个方法分别是decodeURI()和decodeURIComponent()。其中，decodeURI()只能对使用encodeURI()替换的字符进行解码。例如， 它可将%20替换成一个空格，但不会对%23作任何处理，因为%23表示井字号(#)，而井字号不是使用encodeUR工()替换的。同样地，decodeURIComponent()能够解码使用encodeURIComponent()编码的所有字符，即它可以解码任何特殊字符的编码。来看下面的例子：
      var uri=”http%3A%2F%2Fwww.jxbh.cn%2Fillegal%2 0value.htm%23 start”；
      //http% 3A%2F%2Fwww. jxbh.cn%2 Fillegal value .htm%23 start 
      alert( decodeURI(uri));
      //http: //www.jxbh.cn/illegal value .htm# start
       alert( decodeURIComponent (uri));
   这里，变量uri包含着一个由encodeURIComponent()编码的字符串。在第一次调用decodeURI()
输出的结果中，只有%20被替换成了空格。而在第二次调用decodeURIComponent()输出的结果中,所有特殊字符的编码都被替换成了原来的字符，得到了一个未经转义的字符串（但这个字：一个有效的URI）。
    URI方法encodeURI()、encodeURIComponent()、decodeURI()和decodeURIComponent()用于替代已经被ECMA-262第3版废弃的escape()和unescape()方法,URI方法能够编码所有Unicode字符,而原来的方法只能正确地编码ASCII字符.因为在开发实践中,特别是在产品级的代码中,一定要使用URI方法.不要使用escape()和unescape()方法.
