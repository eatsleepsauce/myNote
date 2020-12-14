#### Windows环境使用
第一步：打开 文件》偏好设置》图像 安装Picgo-core
![image.png](https://upload-images.jianshu.io/upload_images/9025957-d884aa0efb924ed6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
第二步：配置Pigo-core 设置图床配置信息，可以使用阿里云OSS，价格还行，不过要做好防盗链，万一被盗链，流量和请求数被刷爆就不好了，当然不发布到网络的话自己用没什么问题的。如果发布到网上最好迁移图床(有钱可以忽略)，正常Pigo-core 配置如下：
```
{
  "picBed": {
    "uploader": "aliyun",
    "aliyun": {
    "accessKeyId": "阿里云账号里面可以看到",
    "accessKeySecret": "阿里云账号里面可以看到",
    "bucket": "使用的bucket",
    "area": "阿里云oss的区域",
    "path": "img/(自己随便定义)",
     "customUrl": "阿里云外网访问地址",
     "options": ""
    }
  },
  "picgoPlugins": {}
}
```
![image.png](https://upload-images.jianshu.io/upload_images/9025957-4d6b9d8d54bf510f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第三步：验证是否成功设置
![image.png](https://upload-images.jianshu.io/upload_images/9025957-b9327cae91b6657c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### Mac环境使用
MAC 可以安装 PicGo-Core.app，安装完成后配置图床信息，填写内容和上面windows的PicGo-Core(command line)一样。验证同样类似。
不过 MAC环境下的typora有一个iPic.app 可以使用，使用阿里云oss的话要花钱升级才行，其实这个不是重点，重点是它的另外一个app，只要安装了iPic就可以使用iPic Mover这款工具，这款工具可以对图床进行批量迁移，可以选择文件夹或则单个文件进行迁移，目标图床就是iPic这个app里面设置的默认图床。