---
layout: post
category: MLSQL
---
题目要求：https://github.com/allwefantasy/mlsql/issues/1404
```text
通读MLSQL语法手册,基于之前构建的环境,在Console上完成下面几道题目。
软件依赖（可以本机通过docker安装）：
1.Kafka
题目:
1.连接console依赖的数据库中的任意一个表，将数据写到delta lake中，并且查询100条展示出来。
2.通过MLSQL mock一些Json格式数据，然后批量写入数据到Kafka中
3.在MLSQL中使用Scala完成一个UDF的演示使用例子
4.离线安装一个excel插件 参考离线安装插件,并上传任意excel文件，然后读取显示出来。
5.（可选题） 写一个流程序参考MLSQL流编程,消费前面写入到Kafka的数据。
```
参考：https://www.notion.so/MLSQL-zookeeper-kafka-mlsql-def1dffab2da4a4d960d6dd47f4cb4c8
# 安装依赖
## 1.安装zookeeper
使用docker拉取zookeeper：
```shell
# 拉取官方源里 指定 tag 为 3.4.14 版本的 zookeeper
docker pull zookeeper:3.4.14

# 执行启动 zookeeper 服务
docker run --name single-zookeeper --restart always -d -p 2181:2181 zookeeper:3.4.14
```
安装效果：
```shell
~ docker pull zookeeper:3.4.14
3.4.14: Pulling from library/zookeeper
45b42c59be33: Already exists
a91c0c19c848: Already exists
b96c0cc1b186: Pull complete
3dc650bc70ed: Pull complete
88d0e7168567: Pull complete
519fc828228a: Pull complete
0792cb630173: Pull complete
da5817fda266: Pull complete
Digest: sha256:97a6eda0ddfcaa1e24e2573a73fbf7396b2c8719516669d5dafe996bdc303c4a
Status: Downloaded newer image for zookeeper:3.4.14
docker.io/library/zookeeper:3.4.14
```
```shell
~ docker run --name single-zookeeper --restart always -d -p 2181:2181 zookeeper:3.4.14
866093fe90024e44537877400adb2165741f226178ac243ac36da780d29bef61
```
查看zookeeper运行情况：
```shell
# 查看当前运行的实例
~ docker ps -a
CONTAINER ID   IMAGE              COMMAND                  CREATED         STATUS         PORTS                                                           NAMES
866093fe9002   zookeeper:3.4.14   "/docker-entrypoint.…"   9 seconds ago   Up 6 seconds   2888/tcp, 0.0.0.0:2181->2181/tcp, :::2181->2181/tcp, 3888/tcp   single-zookeeper
2e7bf0759dcd   mysql              "docker-entrypoint.s…"   4 days ago      Up 4 days      0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp            mysql-latest

# 进入容器
~ docker exec -it 866093fe9002 /bin/bash
root@866093fe9002:/zookeeper-3.4.14# 

# 执行zkCli.sh命令
~ ./bin/zkCli.sh
Connecting to localhost:2181
2021-06-28 07:21:25,615 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.14-4c25d480e66aadd371de8bd2fd8da255ac140bcf, built on 03/06/2019 16:18 GMT
2021-06-28 07:21:25,620 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=866093fe9002
2021-06-28 07:21:25,620 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.8.0_292
2021-06-28 07:21:25,626 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
2021-06-28 07:21:25,626 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/usr/local/openjdk-8
2021-06-28 07:21:25,627 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/zookeeper-3.4.14/bin/../zookeeper-server/target/classes:/zookeeper-3.4.14/bin/../build/classes:/zookeeper-3.4.14/bin/../zookeeper-server/target/lib/*.jar:/zookeeper-3.4.14/bin/../build/lib/*.jar:/zookeeper-3.4.14/bin/../lib/slf4j-log4j12-1.7.25.jar:/zookeeper-3.4.14/bin/../lib/slf4j-api-1.7.25.jar:/zookeeper-3.4.14/bin/../lib/netty-3.10.6.Final.jar:/zookeeper-3.4.14/bin/../lib/log4j-1.2.17.jar:/zookeeper-3.4.14/bin/../lib/jline-0.9.94.jar:/zookeeper-3.4.14/bin/../lib/audience-annotations-0.5.0.jar:/zookeeper-3.4.14/bin/../zookeeper-3.4.14.jar:/zookeeper-3.4.14/bin/../zookeeper-server/src/main/resources/lib/*.jar:/conf:
2021-06-28 07:21:25,627 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
2021-06-28 07:21:25,627 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/tmp
2021-06-28 07:21:25,627 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
2021-06-28 07:21:25,627 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Linux
2021-06-28 07:21:25,628 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=amd64
2021-06-28 07:21:25,628 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=5.10.25-linuxkit
2021-06-28 07:21:25,628 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=root
2021-06-28 07:21:25,628 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/root
2021-06-28 07:21:25,628 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/zookeeper-3.4.14
2021-06-28 07:21:25,631 [myid:] - INFO  [main:ZooKeeper@442] - Initiating client connection, connectString=localhost:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@5ce65a89
Welcome to ZooKeeper!
2021-06-28 07:21:25,665 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1025] - Opening socket connection to server localhost/127.0.0.1:2181. Will not attempt to authenticate using SASL (unknown error)
JLine support is enabled
2021-06-28 07:21:25,751 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@879] - Socket connection established to localhost/127.0.0.1:2181, initiating session
2021-06-28 07:21:25,763 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1299] - Session establishment complete on server localhost/127.0.0.1:2181, sessionid = 0x1000440e0b20001, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null

# 查看目录
[zk: localhost:2181(CONNECTED) 0] ls /
[zookeeper]

# 退出
[zk: localhost:2181(CONNECTED) 1] quit
Quitting...
2021-06-28 07:21:32,999 [myid:] - INFO  [main:ZooKeeper@693] - Session: 0x1000440e0b20001 closed
2021-06-28 07:21:33,001 [myid:] - INFO  [main-EventThread:ClientCnxn$EventThread@522] - EventThread shut down for session: 0x1000440e0b20001

```
## 2.安装kafka
使用docker官方源安装kafka并启动
```shell
# 搜索kafka版本
~ docker search kafka

# 安装kafka
~ docker pull wurstmeister/kafka

# 使用docker启动
~ docker run -d --restart=always --log-driver json-file --log-opt max-size=100m --log-opt max-file=2 --name single-kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=10.3.1.62:2181/kafka -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://10.3.1.62:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 wurstmeister/kafka
# 参数说明：
# -e KAFKA_BROKER_ID=0  在kafka集群中，每个kafka都有一个BROKER_ID来区分自己

# -e KAFKA_ZOOKEEPER_CONNECT=172.16.0.13:2181/kafka 配置zookeeper管理kafka的路径172.16.0.13:2181/kafka

# -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://172.16.0.13:9092  把kafka的地址端口注册给zookeeper，如果是远程访问要改成外网IP,类如Java程序访问出现无法连接。

# -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 配置kafka的监听端口

# -v /etc/localtime:/etc/localtime 容器时间同步虚拟机的时间

# 验证kafka是否可以使用
~ docker exec -it single-kafka /bin/bash
# 进入到 docker kafka 容器内
# cd 到对应的 kafka 安装目录，具体根据自己安装版本进入，但是基本默认路径是在 /opt 下
# ls /opt
# kafka kafka_2.13-2.7.0 overrides
~ cd /opt/kafka_2.13-2.7.0/bin

# 使用生产者生产一条消息, topic name为 test
$ ./kafka-console-producer.sh --broker-list localhost:9092 --topic test
> {"datas":[{"channel":"","metric":"temperature","producer":"xxxx","sn":"IJA0101-00002245","time":"1543207156000","value":"80"}],"ver":"1.0"}

# 退出该终端，并使用消费者消费 topic 'test'
$./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test  --from-beginning
{"datas":[{"channel":"","metric":"temperature","producer":"xxxx","sn":"IJA0101-00002245","time":"1543207156000","value":"80"}],"ver":"1.0"}
# 可正常查看到对应的 topic test 信息正常显示，则该kafka 服务启动正常
```
# Questions & Solutions
## 1.连接console依赖的数据库中的任意一个表，将数据写到delta lake中，并且查询100条展示出来
```
set user="admond";
set password="123456";

connect jdbc where
url="jdbc:mysql://10.3.1.62:3306/mlsql_console?characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&tinyInt1isBit=false"
and driver="com.mysql.jdbc.Driver"
and user="${user}"
and password="${password}"
as db_1;

load jdbc.`db_1.mlsql_job` as table1;

save append table1 as delta.`/table1`;

load delta.`/table1` as show_table1;

select * from show_table1 limit 100 as output;
```
结果展示：
[结果展示1](/public/img/task2-1.png)
[结果展示2](/public/img/task2-2.png)

## 2.通过MLSQL mock一些Json格式数据，然后批量写入数据到Kafka中
```SQL
set abc='''
{ "x": 100, "y": 201, "z": 204 ,"dataType":"A group"},
{ "x": 101, "y": 202, "z": 205 ,"dataType":"B group"},
{ "x": 102, "y": 203, "z": 206 ,"dataType":"C group"},
{ "x": 103, "y": 204, "z": 207 ,"dataType":"D group"},
''';

load jsonStr.`abc` as table1;

select to_json(struct(*)) as value from table1 as table2;
-- save data to kafka
save append table2 as kafka.`xtt` where kafka.bootstrap.servers="10.3.1.62:9092";
```
[结果展示3](/public/img/task2-3.png)

## 3.在MLSQL中使用Scala完成一个UDF的演示使用例子
```SQL
register ScriptUDF.`` as plusFun where
and lang="scala"
and udfType="udf"
and code='''
def apply(a:Double,b:Double)={
   a + b
}
''';

set data='''
{"a":1}
{"a":1}
{"a":1}
{"a":1}
''';
load jsonStr.`data` as table1;

select plusFun(1,2) as res from table1 as output;
```
[结果展示](/public/img/task2-4.png)
