# **exchange安装文档**


# 概述
stone项目的后台服务分两大模块
- stone服务端
- exchange

本文档描述exchange的安装过程；exchange基于开源交易所[CoinExchange](https://gitee.com/cexchange/CoinExchange) ，购买了极少量的收费服务；

安装过程总体参考`CoinExchange`的[总体安装文档](https://gitee.com/cexchange/CoinExchange/tree/master/09_DOC)


# 环境

## 操作系统
- ubuntu 18.04 LTS server

## mysql
- 5.7

## 建议配置
- CPU  8核
- 内存 16G
- 硬盘（机械） 300G

## 端口冲突
- 6010
  > exchange-admin 用到了6010端口；能空出这个端口最好

  > 这个端口一般是x11-ssh的默认端口，关掉影响不大；

## 修改ubuntu时区

详见[Ubuntu修改时区和更新时间](https://blog.csdn.net/zhengchaooo/article/details/79500032)

## 常用目录


#### /opt/
（二进制包手工）安装根目录（即不含 apt-get install安装的第三方软件）
其中`/opt/download`为源码tar包目录

#### 配置目录
- /etc/
  > 通用配置目录；apt-get install安装的通用配置根目录
    >> /etc/mongodb.conf

    >> /etc/mysql/

    >> /etc/redis/

    >> /etc/nginx/

- zookeeper
  > /opt/kafka_2.12-1.0.1/config

- kafka
  > /opt/kafka_2.12-1.0.1/config


#### 日志目录
- /var/log/ 
  > 通用日志目录；apt-get install安装的通用日志根目录
    >> /var/log/mongodb
    
    >> /var/log/mysql

    >> /var/log/redis 

    >> /var/log/nginx

- zookeeper
  > /tmp/zookeeper/

- kafka
  > /tmp/kafka-logs




# 安装oracle的java8

```
## 到oracle官网下载 jdk-8u271-linux-x64.tar.gz
```

相关安装链接[Ubuntu中JAVA安装](https://www.cnblogs.com/ich1990/p/4982806.html)

# 安装kafka

```
# wget
wget http://mirrors.shu.edu.cn/apache/kafka/1.0.1/kafka_2.12-1.0.1.tgz

# wget有问题直接到官网[kafka.apache.org](http://kafka.apache.org/downloads.html)下载kafka_2.12-1.0.1.tgz
```
`总体安装文档`...

## 解压、安装
```
# tar -zxvf kafka_2.12-1.0.1.tgz
# cd kafka_2.12-1.0.1
```
## 9092端口
- 9092端口只在localhost监听；

- 9092端口不在公网IP开放
  > 即注释掉配置文件中的advertised.listeners
  
  > 否则kafka会因公网9092端口访问不到频繁报错；

```
$ pwd
/opt/kafka_2.12-1.0.1/config
$ cat server.properties  | grep 9092
port=9092

listeners=PLAINTEXT://localhost:9092
#advertised.listeners=PLAINTEXT://122.228.226.116:9092
```

## 关于zookeeper
以上kafka安装包以及`总体安装文档`已经包括了zookeeper的内容

## 启停

启动

```
bin/zookeeper-server-start.sh config/zookeeper.properties &
bin/kafka-server-start.sh config/server.properties &
```

## 验证

创建生产者

```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
```

创建消费者

```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test
```

# 安装mongoDB

`总体安装文档` 安装mongodb会有动态库依赖问题，不好解决；直接用apt-get安装；

```
$ sudo apt-get update
$ sudo apt-get install mongodb
```

ubuntu 18、ubuntu 20安装后的mongoDB版本分别是3.6.3、3.6.8

```
$ lsb_release -a 
Description:    Ubuntu 18.04.1 LTS

$ mongod --version
db version v3.6.3
```

根据需要，创建DB，创建表

```
$ mongo --port 27017

## help
> db.help()

## 新建数据库vipcoin
> use vipcoin

## 数据库vipcoin新建tmpTest表，用于测试验证
> db.tmpTest.insert({'foo':'bar'});

## 查看数据库vipcoin已经建立

> show dbs;
admin    0.000GB
config   0.000GB
local    0.000GB
vipcoin  0.000GB

> db
vipcoin

## 查看新表已经建立

> show collections
tmpTest

```

新建用户名、密码
```
## 新建用户
> db.createUser(
   {
      user: "vipcoin",
      pwd: "123456",
      roles: [ { role: "readWrite", db: "vipcoin"} ]
    }
 )

## 查看已经创建的用户
> show users


## 修改配置，使用认证模式，重启mongodb
$ cat /etc/mongodb.conf  | grep auth
auth = true

$ sudo service mongodb restart


## 尝试使用用户密码，访问mongodb

$ mongo -port 27017 -u "vipcoin"  -p "123456" --authenticationDatabase "vipcoin"

```

# mysql
相关参考[Ubuntu18.04 安装MySQL](https://blog.csdn.net/weixx3/article/details/80782479)


安装
```
$ sudo apt update
$ sudo apt-get install mysql-server mysql-client


# mysql版本是5.7
$ mysql --version
mysql  Ver 14.14 Distrib 5.7.32 ...
```

设置监听，localhost 改为 0.0.0.0
```
$ cat /etc/mysql/mysql.conf.d/mysqld.cnf  
...
bind-address            = 0.0.0.0
...
```

设置 root密码等

**不要安装密码校验组件，否则后续带来过多麻烦**
```
$ sudo mysql_secure_installation


VALIDATE PASSWORD PLUGIN 
: No

Disallow root login remotely?
: No

Reload privilege tables now?
: Yes 

```

配置root@localhost访问权限

```
## MySQL初始化安装后， root用户跟其他用户不一样，plugin是auth_socket

mysql> SELECT User, Host, HEX(authentication_string),plugin  FROM mysql.user;
+------------------+-----------+------------------------------------------------------------------------------------+-----------------------+
| User             | Host      | HEX(authentication_string)                                                         | plugin                |
+------------------+-----------+------------------------------------------------------------------------------------+-----------------------+
| root             | localhost |                                                                                    | auth_socket           |
| mysql.session    | localhost | 2A5448495349534E4F544156414C494450415353574F52445448415443414E42455553454448455245 | mysql_native_password |
| mysql.sys        | localhost | 2A5448495349534E4F544156414C494450415353574F52445448415443414E42455553454448455245 | mysql_native_password |
| debian-sys-maint | localhost | 2A45373034393642373734363743393232394634323935424631353033353743383530344538344433 | mysql_native_password |
+------------------+-----------+------------------------------------------------------------------------------------+-----------------------+


## 配置root@localhost访问权限
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
mysql> flush privileges;

## 再次查询的结果就是这样
mysql> SELECT User, Host, HEX(authentication_string),plugin  FROM mysql.user;
+------------------+-----------+------------------------------------------------------------------------------------+-----------------------+
| User             | Host      | HEX(authentication_string)                                                         | plugin                |
+------------------+-----------+------------------------------------------------------------------------------------+-----------------------+
| root             | localhost | 2A44393736393830333845334536333736433236393341443831414536343733393236374542393938 | mysql_native_password |
| mysql.session    | localhost | 2A5448495349534E4F544156414C494450415353574F52445448415443414E42455553454448455245 | mysql_native_password |
| mysql.sys        | localhost | 2A5448495349534E4F544156414C494450415353574F52445448415443414E42455553454448455245 | mysql_native_password |
| debian-sys-maint | localhost | 2A45373034393642373734363743393232394634323935424631353033353743383530344538344433 | mysql_native_password |
+------------------+-----------+------------------------------------------------------------------------------------+-----------------------+
```


尝试在本地访问mysql

```
$ sudo mysql -hlocalhost  -uroot -p[PASSWORD]

```

配置root@'%'访问权限
```
$ sudo mysql  -uroot -p
mysql> grant all privileges  on *.* to root@'%' identified by "123456";
mysql> flush privileges;
```

尝试用公网IP访问mysql

```
$ sudo mysql -h122.228.226.23  -uroot -p[PASSWORD]

```

mysql启停
```
$ sudo systemctl status mysql.service
$ sudo systemctl start  mysql.service
$ sudo systemctl stop  mysql.service

$ sudo systemctl restart mysql.service
```
# mysql-vipcoin库

创建DB vipcoin
```
# 创建DB
mysql> CREATE DATABASE vipcoin;

# 字符集默认是utf8
mysql>  show variables like 'character%';
```

设置访问权限
```
mysql> grant all privileges  on vipcoin.* to vipcoin@'%' identified by "123456";

mysql> flush privileges;

```

初始化vipcoin库

```
$ tar zxvf vipcoin.sql.tar.gz 
vipcoin.sql

$ mysql -h127.0.0.1   -Dvipcoin    -uvipcoin -p123456

mysql> source //home/gaojie/dbInit/vipcoin.sql
```

# redis

```
$ sudo apt-get install redis-server

# 版本
$ redis-server --version
Redis server v=4.0.9

```

根据需要，修改 redis密码

```
$ cat /etc/redis/redis.conf
...
requirepass 11223344
...
```

# nginx

```
$ sudo apt-get install nginx

# 版本
$ nginx -v
nginx version: nginx/1.14.0 (Ubuntu)
```

# maven
```
$ sudo apt-get install maven
```

maven配置在/etc/maven/settings.xml，可以增加阿里云 mirror，使依赖下载加快

```
  <mirror>
    <id>nexus-aliyun</id>
    <name>Nexus aliyun</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    <mirrorOf>central</mirrorOf>
  </mirror>
  ```

# exchange源码编译
## 源码下载
git clone  [exchange源码](https://git.intchain.io/wanlian/vipcoin-framework)


## 修改依赖包路径
修改exchage-api的spark-core依赖包所在路径

```
$ pwd
/home/gaojie/source/git.intchain.io/wanlian/vipcoin-framework/exchange-api

$ cat pom.xml
...
        <dependency>
            <groupId>com.sparkframework</groupId>
            <artifactId>spark-core</artifactId>
            <scope>system</scope>
            <systemPath>/home/gaojie/source/git.intchain.io/wanlian/vipcoin-framework/jar/spark-core-2.6.0.jar</systemPath>
        </dependency>
    </dependencies>
...

```

## 编译源码

```
$ pwd
/home/gaojie/source/git.intchain.io/wanlian/vipcoin-framework/

## 清理编译结果
$ mvn clean

## 编译；  profils("-P")三个可选项 "prod"、"dev"、"test"
$ mvn install -Pprod

```

编译完成后，有jar包生成

```
$ ls -l */target/*.jar
-rw-rw-r-- 1 gaojie gaojie 108216628 Nov  3 14:10 admin/target/admin-api.jar
-rw-rw-r-- 1 gaojie gaojie 110531519 Nov  3 14:10 bitrade-job/target/job.jar
-rw-rw-r-- 1 gaojie gaojie 113155852 Nov  3 11:54 chat/target/chat.jar
-rw-rw-r-- 1 gaojie gaojie  40721445 Nov  3 14:10 cloud/target/cloud.jar
-rw-rw-r-- 1 gaojie gaojie    716432 Nov  3 14:10 core/target/core-1.0.jar
-rw-rw-r-- 1 gaojie gaojie 105135579 Nov  3 14:07 exchange-api/target/exchange-api.jar
-rw-rw-r-- 1 gaojie gaojie     77199 Nov  3 14:10 exchange-core/target/exchange-core-1.0.jar
-rw-rw-r-- 1 gaojie gaojie 105141254 Nov  3 11:54 exchange/target/exchange.jar
-rw-rw-r-- 1 gaojie gaojie 106954241 Nov  3 11:54 market/target/market.jar
-rw-rw-r-- 1 gaojie gaojie 107156580 Nov  3 14:07 otc-api/target/otc-api.jar
-rw-rw-r-- 1 gaojie gaojie      5199 Nov  3 14:07 otc-core/target/otc-core-1.0.jar
-rw-rw-r-- 1 gaojie gaojie 109486390 Nov  3 14:07 ucenter-api/target/ucenter-api.jar
-rw-rw-r-- 1 gaojie gaojie 103637829 Nov  3 11:54 wallet/target/wallet.jar
```

# 修改应用配置
- DB用户密码
- redis用户密码
- mongoDB用户密码
- 

- 更多细节，查看 `*/main/resources/prod/applications.properties`