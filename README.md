Davinci 是一个 DVaaS（Data Visualization as a Service）平台解决方案，面向业务人员/数据工程师/数据分析师/数据科学家，致力于提供一站式数据可视化解决方案

Davinci源码地址：
[https://github.com/edp963/davinci](https://github.com/edp963/davinci)

Davinci源码大概分为三部分：
* 采用React的前端工程
* 采用Spring Boot的后端工程
* 采用Jekyll + Minmal Mistakes的文档工程，用来介绍Davinci的用户操作方法

本篇文章将介绍如何对Davinci后端部分的代码进行开发
>笔者环境：<br>
系统：Windows10 64位<br>
Davinci：davinci-0.3.0-beta.8<br>
Idea版本：2016.1.1<br>
java版本：jdk1.8.0_131<br>
maven版本：3.5.0<br>
mysql版本：5.7.28<br>
phantomjs版本：2.1.1（windows）

# 一、工程目录结构
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
# 二，创建Davinci数据库模型
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
# 三，项目导入idea
祥见笔者另外一篇文章：[Davinci可视化平台 —— 导入idea，利用idea开发后端部分代码](https://blog.csdn.net/huzhenv5/article/details/104238590)
# 四，启动项目
### 1，复制application.yml
将config目录下的application.yml.example复制一份，并重命名为`application.yml`
### 2，修改application.yml数据库配置
* url配置修改：mysql数据库的连接配置，注意将ip、端口和数据库名配置正确，如果要连接的mysql数据库版本5.5.45+、 5.6.26+ 或者 5.7.6+，在连接串最后加上`&useSSL=false`，不然在连接mysql数据库的时候一直会有一个SSL相关的警告
* username：mysql数据库登陆名
* password：mysql数据库登陆密码
### 3，修改application.yml邮件配置
邮件配置用于注册用户的时候，通过邮件服务给注册邮箱发送注册邮件
### 4，修改application.yml截图配置
截图配置是用在定时任务上，davinci支持定时的截取某个图表视图的页面，并发送到指定邮箱，截图功能需要配置外部的工具，所以如果要用这个功能就需要安装phantomjs截图工具，如果不用这个功能，这里可以不配置，首先安装`phantomjs`，自行百度；然后将`phantomjs.exe`的绝对地址复制到`screenshot.phantomjs_path`这个配置上
### 5，idea添加启动配置
# 五，Swagger
该项目有集成swagger2，swagger-ui的链接为：
[http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html)
关于如何将Swagger接口导入到yapi上，可以看笔者的另外一篇文章：[Spring Boot 1.5.8集成Swagger2 + YApi —— Swagger接口信息导入YApi](https://blog.csdn.net/huzhenv5/article/details/104159710)
# 六，打包
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
