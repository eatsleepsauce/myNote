## 1、常用设置
**（1）编辑区字体大小随鼠标滚动大小设置**
preference -> editor ->  gerneral  勾选

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145638.png" alt="编辑区字体滚动大小.png" style="zoom:67%;" />

**（2）鼠标悬浮提示**
preference -> editor ->  gerneral  勾选

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145639.png" alt="鼠标悬浮提示.png" style="zoom:67%;" />

**（3）自动导包和优化多余包**
preference -> editor ->  gerneral -> auto import  勾选

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145640.png" alt="自动导包优化.png" style="zoom:67%;" />

**（4）同一个包下类，超过多少后import合成星号**
preference -> editor -> codestyle -> java -> imports

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145642.png" alt="合成星号.png" style="zoom:67%;" />

**（5）展示行号以及方法间的分隔符**
preference -> editor -> gerneral -> appearance 勾选

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145643.png" alt="展示行及方法分隔线.png" style="zoom:67%;" />

**（6）代码提示时忽略大小写匹配**
preference -> editor -> gerneral -> code completion 去掉勾选

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145644.png" alt="忽略大小写进行提示.png" style="zoom:67%;" />

**（7）多个文件打开时能够展示更多文件tab，并且文件tab可以分行展示**
preference -> editor -> gerneral -> editor tabs

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145645.png" alt="多文件展示.png" style="zoom:67%;" />

**（8）修改类头文档注释信息**
preference -> editor -> file and code templates

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145646.png" alt="文档头注释设置.png" style="zoom:67%;" />

**（9）设置项目编码**
preference -> editor -> file encodings

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145647.png" alt="编码设置.png" style="zoom:67%;" />

**（10）自动编译**
preference -> build,execution,deployment -> compiler 勾选

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145649.png" alt="自动编译.png" style="zoom:67%;" />

**（11）省电模式，不能默认勾选，否则影响一些自动编译**

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145650.png" alt="省电模式.png" style="zoom:67%;" />

**（12）生成序列号版本号**
preference -> editor -> inspections 勾选

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145652.png" alt="自动生成序列化版本号.png" style="zoom:67%;" />

##2、常用快捷键
**第一个为windows环境快捷键，第二个为mac环境快捷键**
>创建内容：alt + insert  |  command + n
复制一行：ctrl + d  |  commad + d
删除一行：ctrl + y  |  command + delete
代码上移：ctrl + shift + up  |  command + shift + up
代码下移：ctrl + shift + down  |  command + shift + down
上下选择：alt + up/down  |  option + up/down
撤销：ctrl + z  |  command + z
反向撤销：ctrl + shift + z  |  command + shift + z
搜索类：ctrl + n   |  command + o
代码生成(构造器、getter setter等)：alt + insert  |   command + n
意向动作：alt + enter  |   option + enter
单行注释：ctrl + /   |  command + /
多行注释：ctrl + shift + /  |  control + shift + /
生成代码块包围：ctrl + alt + t  |  command + option + t
代码自动提示，自动补齐：需要修改热键，preference->key map->completion basic 修改成alt + /  | option + /
进入类：ctrl + 鼠标点击  |  command + 鼠标点击
看具体实现类：ctrl + alt + 鼠标点击  | command + option + 鼠标点
前进：ctrl + alt + 右箭头  |  command + opiton + 右箭头
后退：ctrl + alt + 左箭头  |  command + option + 左箭头
手动导包：ctrl + alt + o  |  command + option + o
自动缩进：control + option + i
格式化：ctrl + alt + l  |  control + option + l
自动结束：ctrl + shift + enter  | command + shift + enter
方法覆盖，生成实现方法：ctrl + o ，ctrl + i  |  control + o ，control + i
显示方法调用树：ctrl + h，ctrl + alt + h   |   control + h，control + option + h

##3、postfix completion 和 live templates 
postfix completion 在 preference -> editor  -> gerneral -> postfix completion中设置
live templates 在 preference -> editor -> live templates中设置
常见的（输完后tab自动生成）：
main函数生成：main 或 psvm
system.out.println输出：sout  或者 xxx.sout
for循环：fori  或 xxx.fori  xxx.forr(逆向)
增强for：iter  或  xxx.for
null 判断：ifn 或 xxx.null
```
// ifn +tab
if( xxx == null ){
}
```
非null判断：inn 或 xxx.nn
private static final ： prsf
public static final ：psf

##4、导入maven工程
idea中的project 相当于eclipse中的工作空间，modules相当与eclipse中的project。
首次导入maven项目后，需要在pom.xml上右击然后reimport 加载jar的依赖。
