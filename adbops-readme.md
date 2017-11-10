
##	ADBOPS模块简介
### 功能和用途
ADBOPS支持用户配置动态监控ADB集群及主机资源信息， launcher进程循环扫描任务表job，启动定时处理任务，执行指定的资源采集监控脚本，并将执行结果插入历史记录表中，并在前台portal展示。


## 系统运行环境

### 软件环境
|项目|值|
|-|-|
|操作系统|Red Hat Enterprise Linux Server release 6.3 (Santiago)+|
|内核版本|2.6.32-279.el6.x86_64|
|ADB版本 |v2.2以上|
|adbops版本|已经和adb打包一起发布，安装时随adb一起安装完成|
|JDK|1.7.0以上|
|TOMCAT|7.0以上|
|adbops.war|部署在TOMCAT之上|

在adbmgr、adb集群均正常运行的提前下，安装部署adbops。
##	安装配置JDK和TOMCAT
### 安装配置JDK
#### 查看Linux自带的JDK是否已安装
```
java -version
```
#### 查看JDK信息
```
rpm -qa | grep java
```
显示：
```
java-x.x.x-openjdk-headless-x.x.x.x-x.x.bxx.exx.xxx
java-x.x.x-openjdk-x.x.x.x-x.x.bxx.exx.xxx
```
--如果上述命令没有返回结果，或返回的jdk版本在1.7以下，则需要全新安装或卸载后新安装。
#### 卸载
```
rpm -e --nodeps java-x.x.x-openjdk-headless-x.x.x.x-x.x.bxx.exx.xxx
rpm -e --nodeps java-x.x.x-openjdk-x.x.x.x-x.x.bxx.exx.xxx
```
如果出现找不到openjdk source的话，那么还可以这样卸载
```
yum -y remove java ava-x.x.x-openjdk-headless-x.x.x.x-x.x.bxx.exx.xxx
yum -y remove java java-x.x.x-openjdk-x.x.x.x-x.x.bxx.exx.xxx
```
#### 安装JDK(这里以安装JDK1.7为例，JDK1.7以上的版本都是可用的)
下载地址：（JDK1.7）
[http://www.oracle.com/technetwork/indexes/downloads/index.html](http://www.oracle.com/technetwork/indexes/downloads/index.html)    
放在/usr/local/src下执行以下安装命令：
```
rpm -ivh jdk-7u79-linux-x64.rpm
```
默认会安装在/usr/java下
#### 检查是否安装 
```
java -version
```
显示：`java version"1.7.0_79"` ...表示成功
#### 配置环境变量
`vi /etc/profile`
在最后加入以下几行：
```
JAVA_HOME=/usr/java/jdk1.7.0_71
JRE_HOME=$JAVA_HOME/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
PATH=$JAVA_HOME/bin: $JRE_HOME/bin:$PATH
export JAVA_HOME CLASS_PATH PATH JRE_HOME
```
执行下面命令立即生效
```
source /etc/profile
```
### 安装配置TOMCAT
#### 安装TOMCAT
安装TOMCAT
下载地址：(TOMCAT7.0)
[http://mirror.bit.edu.cn/apache/tomcat/](http://mirror.bit.edu.cn/apache/tomcat/)
放在adbops安装用户的根目录下，执行以下安装命令：
```
$ tar -zxvf apache-tomcat-7.0.77.tar.gz
$ mv apache-tomcat-7.0.77 tomcat
```
#### 配置TOMCAT
##### 配置环境变量
```
export CATALINA_HOME=/home/adb/tomcat/
```
##### 配置catalina选项的 $ CATALINA_HOME/bin/catalina.sh
以下配置可选，默认配置也可。
--配置catalina监听端口以tcp协议启动，而非tcp6协议
```
JAVA_OPTS="$JAVA_OPTS -Dorg.apache.catalina.security.SecurityListener.UMASK=`umask`"

```
--配置catalina虚拟内存参数
```
JAVA_OPTS="-Xms256m -Xmx512m"

```
### 安装配置ADBOPS portal
#### 	安装ADBOPS portal
##### 上传 adbops.war至如下目录$CATALINA_HOME/webapps
##### 确认上传成功
```
$ll $CATALINA_HOME/webapps
总用量 11023
-rw-r--r--  1 postgres postgres 11426820 4月  10 13:44 adbops.war
drwxr-xr-x 14 postgres postgres     4096 4月  11 09:28 docs
drwxr-xr-x  7 postgres postgres      105 4月  11 09:28 examples
drwxr-xr-x  5 postgres postgres       82 4月  11 09:28 host-manager
drwxr-xr-x  5 postgres postgres       97 4月  11 09:28 manager
drwxr-xr-x  3 postgres postgres     4096 4月  11 09:28 ROOT
```
##### 重新启动tomcat，即可自动解压war包adbops.war
```
$ ll $CATALINA_HOME/webapps
总用量 11168
drwxrwxr-x  6 postgres postgres       73 4月  11 10:33 adbops
-rw-r--r--  1 postgres postgres 11426820 4月  10 13:44 adbops.war
drwxr-xr-x 14 postgres postgres     4096 4月  11 09:28 docs
drwxr-xr-x  7 postgres postgres      105 4月  11 09:28 examples
drwxr-xr-x  5 postgres postgres       82 4月  11 09:28 host-manager
drwxr-xr-x  5 postgres postgres       97 4月  11 09:28 manager
drwxr-xr-x  3 postgres postgres     4096 4月  11 09:28 ROOT
```
#### 配置ADBOPS portal
###### 配置数据库连接参数
配置文件路径： `$CATALINA_HOME/webapps/adbops/WEB-INF/classes/db.properties`
填写adbmgr的连接地址：
```
jdbc.driverClassName=org.postgresql.Driver
jdbc.username=postgres
jdbc.password=postgres
jdbc.url=jdbc:postgresql://192.168.111.141:6432/postgres?useUnicode=true
```
--此处连接url为adbmgr的连接端口
##### 配置tomcat的日志级别
配置文件路径： `$CATALINA_HOME/webapps/adbops/WEB-INF/classes/log4j.properties`

## 安装配套工具
#### 安装python-psutil工具
##### 下载psutil版本
版本要求：psutil-4.1.0以上
```
wget git clone https://github.com/giampaolo/psutil.git
```
##### 安装psutil
```
cd psutil
python setup.py install
```
##### 验证psutil安装是否成功
--退出psutil安装目录，切换至任意其他目录，否则执行下述命令会报错。
```
#python
>>> import psutil
```
--执行python命令，然后执行import psutil 语句，没有任何返回表示安装成功。
#### 安装lsb_release工具
```
yum -y install redhat-lsb
```

### 配置ADB
#### 在coordinator节点操作：
创建插件`pg_stat_statements`
```sql
psql –p 6433
postgres=# create extension pg_stat_statements;
```
#### 修改coordinator节点的配置
> 注意：
> shared_preload_libraries的参数，需要查看该参数原来的值是否为空，如果为空，则可以直接设置，如果不为空，则需要将pg_stat_statements追加到参数值中，比如：shared_preload_libraries='xxx,pg_stat_statements'.

登录adbmgr，查看coordinator的shared_preload_libraries参数值：
```
show coord1 shared_preload_libraries;
```
登录adbmgr，修改coordinator的如下配置：
```
set coordinator all (shared_preload_libraries='pg_stat_statements');
set coordinator all (pg_stat_statements.max=100);
set coordinator all (pg_stat_statements.track=all);
set coordinator all (pg_stat_statements.save=true);
```

### 安装验证
停止tomcat
```
sh $CATALINA_HOME/bin/shutdown.sh
```
确认已经完全停止tomcat
```
netstat -an|grep 8080
```
清理tomcat缓存
```
rm -Rf $CATALINA_HOME/work/*
```
启动tomcat
```
sh $CATALINA_HOME/bin/startup.sh
```
确认已经成功启动tomcat
```
netstat -an|grep 8080
tcp6       0      0 :::8080                 :::*                    LISTEN
```
打开浏览器(建议使用chrome或firefox)访问adbops portal
http://192.168.111.141:8080/ adbops
输入默认用户和密码(adbmonitor/admin)
成功则返回如下信息界面：



## 配置定时采集任务
### 配置主机资源采集任务
```
add job usage_for_host (interval= 60,command = 'select monitor_get_hostinfo();');
```
---参数说明
任务名称：host
采集间隔：60 秒
采集命令：select monitor_get_hostinfo();

###	配置数据库资源采集任务
```
add job usage_for_adb (interval= 60,command = 'select monitor_databaseitem_insert_data();');
add job tps_for_adb (interval= 60,command = 'select monitor_databasetps_insert_data();');
```
### 配置数据库慢SQL监控任务
```
add job slowlog_for_adb (interval= 60,command = 'select monitor_slowlog_insert_data();');
```
###	配置coordinator监控任务
```
ADD JOB  mon_coord (INTERVAL = 5, STATUS = TRUE,COMMAND =' SELECT MONITOR_HANDLE_COORDINATOR()' );
```

### 管理任务
#### 暂停任务
```
alter job usage_for_host (status = false);
alter job usage_for_adb (status = false);
alter job tps_for_adb (status = false);
alter job slowlog_for_adb (status = false);
alter job mon_coord (status = false);

```
#### 删除任务
```
drop job usage_for_host;
drop job usage_for_adb;
drop job tps_for_adb;
drop job slowlog_for_adb;
drop job mon_coord;
```
### 相关表
|模块|Schema|表名|
|-|-|-|
|主机资源|pg_catalog|monitor_host|
|||monitor_cpu|
|||monitor_mem|
|||monitor_disk|
|||monitor_net|
|数据库资源采集|pg_catalog|monitor_databaseitem|
|||monitor_databasetps
|慢sql采集|pg_catalog|monitor_slowlog|
|告警信息|pg_catalog|monitor_alarm|
|||monitor_host_threshold|
|||monitor_resolve|
|任务管理|pg_catalog|monitor_job|
|||monitor_jobitem|
|用户管理|pg_catalog|monitor_user|



## 常见问题FAQ
### IE访问portal没有响应
方法：将url加入可信任列表，重启IE即可


