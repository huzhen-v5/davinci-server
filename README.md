Davinci 是一个 DVaaS（Data Visualization as a Service）平台解决方案，面向业务人员/数据工程师/数据分析师/数据科学家，致力于提供一站式数据可视化解决方案

Davinci源码地址：
[https://github.com/edp963/davinci](https://github.com/edp963/davinci)

Davinci源码大概分为三部分：
* 采用React的前端工程
* 采用Spring Boot的后端工程
* 采用Jekyll + Minmal Mistakes的文档工程，用来介绍Davinci的用户操作方法

本篇文章将介绍如何对Davinci后端部分的代码进行开发
>笔者环境：
系统：Windows10 64位
Davinci：davinci-0.3.0-beta.8
Idea版本：2016.1.1
java版本：jdk1.8.0_131
maven版本：3.5.0
mysql版本：5.7.28
phantomjs版本：2.1.1（windows）

# 一，代码获取
下载Davinci源码，源码地址文章开头已经给出；下载完后去掉一些没有必要的文件：
1. 清空`davinci-ui`文件夹，该文件夹存放的是前端打包后的文件，用于打包整个工程的，开发后端过程中用不上；打包整个Davinci工程的时候会用到这个文件夹，所以只清空，不删除
2. 删除`docs`文件夹，该文件夹是用于开发用户说明文档静态网站的工程，跟后端工程无关，开发用户说明文档的方法可以看笔者的另外一篇文章：[Davinci可视化平台 —— Jekyll+Minimal Mistakes的用户手册工程本地打包发布](https://blog.csdn.net/huzhenv5/article/details/104196806)
3. 删除`webapp`文件夹，该文件夹是前端部分的开发代码，开发后端过程中用不上，如何开发前端部分代码可以看笔者的另外一篇文章：[Davinci可视化平台 ——前端部分代码开发](https://blog.csdn.net/huzhenv5/article/details/104224241)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209183139593.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
# 二、工程目录结构
用户配置在项目根目录 /config/ 下，项目启动脚本和升级补丁脚本在项目根目录 /bin/ 下， 后端代码及核心配置在 server/ 目录下, 日志在项目根目录 /log/ 下
### 1，脚本
```
├── bin                   # 脚本目录
  ├── migration             # 较大版本变动迁移脚本目录
  ├── patch                 # 数据库补丁
  	 ├── 001_beta5.sql        # 已发布补丁（命名规则：“序列_版本”）
  	 └── beta.sql             # 当期未发布补丁（固定名称）
  ├── build.sh
  ├── davinci.sql           # 完整系统数据库脚本（包含所有补丁）
  ├── initdb.bat            # 针对 Windows 环境的初始化数据库批处理脚本
  ├── initdb.sh             # 针对 Linux、Mac 环境的初始化数据库 Shell 脚本
  ├── phantom.js            # 截图脚本（未来版本将不再使用）
  ├── restart-server.sh     # 针对 Linux、Mac 环境的重启服务脚本
  ├── run.bat               # 针对	Windows 环境的服务启停核心脚本
  ├── start.bat             # 针对 Windows 环境的服务启动脚本
  ├── start-server.sh       # 针对 Linux、Mac 环境的服务启动脚本
  ├── stop.bat              # 针对 Windows 环境的服务停止脚本
  └── stop-server.sh        # 针对 Linux、Mac 环境的服务停止脚本
```
### 2，用户配置
用户配置
```
├── config                          # 用户配置目录
  ├── application.yml.example         # 应用配置模板
  ├── datasource_driver.yml.example   # 自定义数据源配置模板
  └── logback.xml                     # 日志配置
```
### 3，server代码
```
├── server                                  # Server 代码根目录
   ├── src                                    # 源码
  	  ├── main
  	  	 ├── java
  	  	 	└── edp
  	  	 	   ├── core                             # 核心配置及通用代码
  	  	 	   ├── davinci                          # Davinci 业务代码
  	  	 	   ├── DavinciServerApplication         # 系统启动类
  	  	 	   └── SwaggerConfiguration             # Swagger 配置类
  	  	 └── resources
  	  	 	├── generator
  	  	 	├── mybatis                           # mybatis mapping 目录
  	  	 	├── templates                         # 邮件、Sql 模板目录
  	  	 	├── application.yml                   # 系统核心配置文件
  	  	 	└── banner.txt
  	  └── test                                # 测试代码目录
   └── pom.xml                              # Davinci Server maven 配置文件，继承自项目根目录pom.xml
```
### 4，日志
日志目录
```
├── logs        # 日志根目录
  ├── sys         # 系统日志目录
  └── user        # 用户日志目录
  	 ├── opt        # 用户操作日志
  	 └── sql        # 用户Sql日志
```
# 三，创建Davinci数据库模型
Davinci的开发者已经将创建数据模型的文件写好，放到了bin目录下，文件名是`bin/davinci.sql`，利用该sql可以快速的在mysql中创建运行davinci的数据模型

执行命令：

```sql
mysql -P 3306 -h localhost -u root -p davinci< D:\Davinci\bin\davinci.sql
```
参数说明：
* -P：mysql数据库端口
* -h：mysql数据库ip地址
* -u：登陆的msql数据库的用户名
* -p：通过密码登陆
* davinci：创建的数据库的名称，可自定义
* D:\Davinci\bin\davinci.sql：davinci.sql文件在本机的绝对路径
# 四，项目导入idea
### 1，打开idea启动页
如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209190348384.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
如果已经打开某个项目，关闭当前项目即可：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209185752119.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
### 2，导入项目
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209190511123.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
选择刚刚下载好的代码（davinci源码不要放到带用中文的路径中）
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020020919062360.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020020919083552.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209190858708.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209190919664.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
选择jdk1.8
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209191025607.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
点击Finish
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209191105526.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
### 3，配置idea的maven环境
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209191223517.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
选择maven路径
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209191343805.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
选择maven的settings.xml文件路径
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020020919151958.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
选择好后，maven开始下载相关依赖，慢慢等...导入完成
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209195148101.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
### 4，platform添加tools.jar
项目有的地方用到了tools.jar包里面的类，所以要添加这个jar包，添加方法：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209195646461.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209195746107.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
选择`jdk1.8`安装目录下`lib`文件夹下的`tools.jar`![目录下的在这里插入图片描述](https://img-blog.csdnimg.cn/20200209195904103.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
点击OK确定
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209200241790.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
### 5，安装lombok插件
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209200345231.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209200426767.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209200516858.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
重启idea
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209200604383.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
# 五，启动项目
### 1，复制application.yml
将config目录下的application.yml.example复制一份，并重命名为`application.yml`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209201111618.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
### 2，修改application.yml数据库配置
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209201525298.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
* url配置修改：mysql数据库的连接配置，注意将ip、端口和数据库名配置正确，如果要连接的mysql数据库版本5.5.45+、 5.6.26+ 或者 5.7.6+，在连接串最后加上`&useSSL=false`，不然在连接mysql数据库的时候一直会有一个SSL相关的警告
* username：mysql数据库登陆名
* password：mysql数据库登陆密码
### 3，修改application.yml邮件配置
邮件配置用于注册用户的时候，通过邮件服务给注册邮箱发送注册邮件
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209202517693.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
如何配置的话，看Davinci源码的`docs`文件夹的说明文档，docs文件夹是用户操作说明文件夹，在文章开头笔者说这个没有用，可以删了，如果真的删了再重新解压下之前下载的zip包吧，参考`davinci/docs/zh/1.1-deployment.md`这个文章：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209203012456.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
### 4，修改application.yml截图配置
截图配置是用在定时任务上，davinci支持定时的截取某个图表视图的页面，并发送到指定邮箱，截图功能需要配置外部的工具，所以如果要用这个功能就需要安装phantomjs截图工具，如果不用这个功能，这里可以不配置，首先安装`phantomjs`，自行百度；然后将`phantomjs.exe`的绝对地址复制到`screenshot.phantomjs_path`这个配置上
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209203515512.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
### 5，idea添加启动配置
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209203717291.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209203815130.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209203843655.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
自定义名字
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209203936632.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209203959865.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
启动类是`server/src/main/java/edp/DavinciServerApplication.java`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209204142699.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
添加环境变量，添加一个环境变量`DAVINCI3_HOME`，值为本项目的绝对路径
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209204443430.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209204622750.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209204713749.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
### 6，启动
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209205129371.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
启动成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209205239570.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
# 六，Swagger
该项目有集成swagger2，swagger-ui的链接为：
[http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209205425135.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1emhlbnY1,size_16,color_FFFFFF,t_70)
关于如何将Swagger接口导入到yapi上，可以看笔者的另外一篇文章：[Spring Boot 1.5.8集成Swagger2 + YApi —— Swagger接口信息导入YApi](https://blog.csdn.net/huzhenv5/article/details/104159710)
# 七，打包
### 1，打完整release包
首先将前端部分代码打包，将打包好的文件复制到`davinci-ui`目录下，然后在根目录下运行：

```powershell
mvn clean package
```
打好的包在`assemby/target`目录下
### 2，单独打server部分的包
进入`server`目录在，在该目录下执行：

```powershell
mvn clean package
```
打好的包在`server/target`目录下
