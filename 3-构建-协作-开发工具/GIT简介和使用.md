##1、git简介
git是一个分布式版本控制系统（如git、bitkeeper），集中化版本控制系统有cvs、svn等。相对而言，**集中化版本控制系统**存在单点故障问题，并且本地存放的是最新版本。集中化版本控制系统的版本关注的是和上个版本的差异，如果要恢复之前的某个版本则需要一个一个的版本按顺序还原。
![svn中心化.png](https://upload-images.jianshu.io/upload_images/9025957-1bf7d85670003cd3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![svn的版本.png](https://upload-images.jianshu.io/upload_images/9025957-dc4b73272579a04c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**分布式版本控制**（分布式指的是本地库和远程库，每台电脑都是一个版本仓库，虽然git是分布式的，但在实际工作中，一般git还是会有一个集中的服务中心，目前比较主流的GITHUB和码云，当然也可以搭建git服务器），客户端并不只是提取最新的版本快照，而是把代码库完整地镜像下来（这么一来，任何一处协同工作用的服务器发生故障，事后都可以用任何一个镜像出来的本地仓库恢复）。可以指定和若干不同的远端仓库进行交互。分布式版本控制系统的版本存放的是索引而不是差异。
![分布式.png](https://upload-images.jianshu.io/upload_images/9025957-56a5395507377afd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Git的版本.png](https://upload-images.jianshu.io/upload_images/9025957-7a1650b21c53ae5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##2、git本地结构
git整体分为三个部分：工作区、暂存区和本地库。
工作区：工作区就是我们工作的文件夹目录
暂存区：修改后的文件放入到这个暂存区，而不是直接发布版本（设置暂存区的作用？为了分段提交，暂存区是可以随意的将各种文件的修改放进去或撤回，最后一次性提交？这个解释感觉不够全面）
本地库：存放的是我们发布的各个版本
![git本地结构.png](https://upload-images.jianshu.io/upload_images/9025957-5aa397b07545cd46.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##3、git协作
**团队内部协作**，下图中只是简单操作，现实中的步骤肯定比这个多得多。这种情况远程库的托管中心可以使用gitlab之类的。

![团队内部协作.png](https://upload-images.jianshu.io/upload_images/9025957-da36cb9340d934e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**跨团队合作（跨公司）**，下图中也只是简单操作，现实的中步骤肯定更多。远程库的托管中心可以使用github或者gitee（码云）。

![跨团队协作.png](https://upload-images.jianshu.io/upload_images/9025957-dfdfaeb4474f9ee8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##4、本地库及分支
**一、本地库初始化**
（1）本地创建文件
（2）使用终端，如果首次使用需要设置用户名及邮箱
```
git config --global user.name "用户名"
git config --global user.email "邮箱"
```
（3）本地初始化
```
git init
```
（4）常用命令（git命令很多，文中命令只涉及少量）
```
git add 文件名  //加文件提交到暂存区
git commit -m "提交描述" 文件名  //将暂存区到文件提交到本地库
git status  //查看工作区和暂存区到状态信息

***************日志和恢复非常重要，此处后续补图说明 *****************
git log  //查看提交日志，由近及远
//log日志过多时，查看日志有分页效果（左下脚展示冒号），下一页 空格，上一页 b，退出 q
git log --pretty = oneline  //日志展示优化，展示索引号
git log --oneline  //日志展示优化，索引号只展示部分
git reflog  //展示优化，展示回退指针索引号

git reset  // 结合reflog进行版本恢复
git reset --hard 索引号  //本地库指针移动的同时重置工作区、暂存区 
git reset --mixed 索引号  //本地库指针移动的同时，只重置暂存区
git reset --soft 索引号  //本地库指针移动的同时，不重置任何区域

git diff  // 将工作区文件和暂存区的文件比较
git diff [文件名可选]  // 有文件名的时候是按文件名比较，git是按行管理文件的
git diff HEAD 文件名 // 暂存区和本地库的对比，HEAD是本地库的索引号别名，可以直接使用索引号
```

**二、分支**
在版本控制过程中，使用多条线程同时推进多个任务，这里的多条线程即分支。
![分支.png](https://upload-images.jianshu.io/upload_images/9025957-e0d7f1e35f492c61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

（1）查看分支，“*”星号代表目前所在的分支   git branch -v
（2）创建分支，git branch 分支名
（3）切换分支，git checkout 分支名

**三、分支冲突解决**
git是以行来管理文件的，所以在不同分支上修改同一个文件的同一行，合并两个分支的时候就会发生冲突。例如，master分支和branch1分支两个分支都修改来同一个文件的同一行，在master中合并branch1的时候发生冲突。
```
git checkout master  // 切换到主分支
git merge branch1  // 将branch1合并到主分支上
发生冲突后只能手动解决，解决后
git add xxx // 将冲突文件提交到暂存区
git commit -m "冲突解决备注"  // 将冲突修改提交到本地库，注意：此处不需要文件名
冲突解决完后可以通过，git status 查看工作区和暂存区到状态

```

##5、本地库和远程库
**一、本地库和远程库关联**
（1）初始化本地库（上面已有步骤）
（2）创建远程库，在github上创建（可以使用别的代码托管中心），获取远程库的地址信息
（3）本地新增远程库别名（因为远程库地址信息太长） 
```
git remote add origin xxx.git  // origin远程库别名，xxx.git远程库地址信息
git remote -v  // 查看本地的远程库别名信息
```
（4）将本地库分支push推送到远程库中
```
git push origin master // origin为远程库别名，也可以直接使用远程库地址信息xxx.git 代替，master 为本地分支
```
（5）推送成功后可以在远程库中产看推送的内容信息

**二、远程库clone 、加入团队、pull以及push冲突解决**
（1）clone操作，clone的作用：初始化了一个本地库，克隆了远程库的内容，在本地创建了远程库的别名。
```
git clone xxx.git  // xxx.git 远程库地址
git remote -v  // 查看本地的远程库别名信息
```
（2）加入团队，以githup为例，创建远程库的可以在库的settings -> manage access -> invited a collaborator 生成邀请链接，将邀请链接发送给被邀请者让其接收邀请。被邀请者加入邀请后方可进行push操作。
（3）pull操作，pull相当于fetch和merge两个操作，fetch是将远程库下载到本地，可以使用命令 git checkout origin/master 切换查看（远程库的master分支，origin依旧是远程库别名）。
使用merge操作时记得先切回到本地master分支 git checkout master，然后再使用 git merge origin/master 将远程库的分支合并到本地master中。
直接pull的操作使用命令，git pull origin master 。
（4）push时发生冲突，此时无法push成功
```
首先pull拉去到本地，解决冲突
git pull origin master // 远程库别名 和 本地分支 命名自定
解决冲突后
git add xxx 
git commit -m "解决冲突备注"  //此时不需要文件
git push origin master
```

##6、跨团队协作
和上面的内容相差不是很大，结合之前的协作示意图可以发现，其实就多了两个内容，一个fork操作以及另一个pull request（另外一个团队还需要进行 审核 以及 merge操作，pull request的内容可在 filechanges中查看）。

##7、ssh免密登录
其实，clone远程仓库的时候并不需要登录，只有push这些操作才需要。
正常操作步骤：
（1）进入用户主目录 cd ～
（2）执行命令，生成一个.ssh目录， ssh -keygen -r rsa -C  邮箱 
（3）在生成的目录中，打开id_rad.pub文件，复制其中的内容
（4）githup账号，settings->SSH and GPG keys 里面粘贴复制的内容信息
（5）尝试正常push

##8、idea中的简单应用
（1）在已有的项目中增加git管理，vcs->import into version control -> create git repository
（2）push的时候正常push，不过会提示维护远程库信息
（3）直接clone远程库 new-> project from version control -> git


