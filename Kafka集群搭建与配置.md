# Kafka集群搭建与配置

1. 准备工作

2. Java环境

3. 搭建zookeeper集群

4. 搭建kafka集群



## 1. 准备工作

开发工具 xshell6.0

至少准备三台服务器（Linux系统）

192.168.8.190  server1

192.168.8.185  server2

192.168.8.183  server3



通过xshell 连接三台服务器，新建连接，分别填入名称(server1,server2,server3)及主机地址，连接。

<img src="img/image-20200509085014379.png" alt="image-20200509085014379" style="zoom:67%;" />

接受并保存主机密钥，依次输入用户名及密码

用户名：kafka

密码： zaI!Ge6aV@3fw$&b



## 2. Java 环境

由于zookeeper，kafka集群的运行需要Java运行环境，所以需要首先安装 JDK。

在每台主机下执行下面步骤：

查看是否有默认安装的 Java JDK

```bash
java -vesion
```

确保安装的Java版本是Oracle jdk, 如下图所示，Oracle jdk 显示为 Java HotSpot。（若未安装JDK 或版本不符合，请见附录）

<img src="img/image-20200508173417581.png" alt="image-20200508173417581"  />



## 3. 搭建zookeeper集群

### 3.1 下载zookeeper压缩包

https://zookeeper.apache.org/releases.html

zookeeper版本

[zookeeper-3.6.1-bin.tar.gz]: https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.6.1/apache-zookeeper-3.6.1-bin.tar.gz

找到每台服务器zookeeper对应的安装路径

```bash
cd home/kafka
```

使用xshell将 zookeeper 压缩文件上传到/home/kafka目录中

### 3.2 解压zookeeper

通过如下命令解压

```bash
tar -zxvf apache-zookeeper-3.6.1-bin.tar.gz
```

重命名文件夹为zookeeper

```bash
mv zookeeper-3.6.1-bin zookeeper
```

### 3.3 修改配置文件

新建zookeeper配置文件

```bash
cp /home/kafka/zookeeper/conf/zoo_sample.cfg /home/zookeeper/conf/zoo.cfg 
```

![image-20200511141847124](img/image-20200511141847124.png)



进入到conf目录，修改配置文件zoo.cfg

```bash
vim zoo.cfg
```

修改dataDir 部分以及在配置文件末尾加上这三行，ip填写3台服务器地址，zookeeper服务默认的端口号为2181,2888和3888,添加server 集群信息。（2181：对cline端提供服务，3888：选举leader使用，2888：集群内机器通讯使用（Leader监听此端口））

```bash
#快照日志
dataDir=/home/kafka/zookeeper/data

#server
server.1=192.168.8.190:2888:3888
server.2=192.168.8.185:2888:3888
server.3=192.168.8.183:2888:3888
```

![image-20200511142156114](img/image-20200511142156114.png)



在对应目录创建data文件夹

```
mkdir /home/kafka/zookeeper/data
```



### 3.4 创建myid文件

在dataDir指定的目录下，创建myid文件，内容 与zoo.cfg文件中的序号一致

```bash
cd /home/kafka/zookeeper/data
vim myid
1
```

<img src="img/image-20200511142305027.png" alt="image-20200511142305027" style="zoom:150%;" />

其余服务器依次在相应目录创建myid文件，写上相应配置数字即可。

192.168.8.185 对应 myid 为2
192.168.8.183 对应 myid 为3



### 3.5 启动zookeeper服务

进入zookeeper目录，执行启动zookeeper命令：

```bash
#启动ZK服务: 
./bin/zkServer.sh start
```

 Zookeeper集群需要每台挨个启动。

 可以用jps命令查看线程。

查看集群节点状态

```
#查看ZK服务状态: 
./bin/zkServer.sh status
```

QuorumPeerMain 表示zookeeper启动



![image-20200511143347063](img/image-20200511143347063.png)



其他常用命令

```
#停止ZK服务: 
./bin/zkServer.sh stop
#重启ZK服务: 
./bin/zkServer.sh restart
```



## 4. 搭建kafka 集群

### 4.1 下载kafka压缩包

https://mirror.bit.edu.cn/apache/kafka/2.5.0/kafka_2.12-2.5.0.tgz 

将下载的 kafka 压缩文件上传到集群中的每台机器home/kafka目录中

### 4.2 解压kafka

执行如下命令进行解压。

```bash
tar -zxvf kafka_2.12-2.5.0.tgz 
```

重命名文件夹为kafka

```bash
mv kafka_2.12-2.5.0 kafka
```

### 4.3 修改配置文件

进入kafka的config目录，修改配置文件server.properties

```bash
vim server.properties
```

修改broker.id,listeners以及zookeeper集群的ip地址，修改日志文件路径，并创建 kafka 日志目录。

```bash
#三台服务器分别对应1，2，3
broker.id=1
#对应自身ip地址
listeners=PLAINTEXT://192.168.8.190:9092
#zookeeper集群地址
zookeeper.connect=192.168.8.190:2181,192.168.8.185:2181,192.168.8.183:2181
log.dirs=/home/kafka/kafka/kafka-logs
```

第一个 broker.id 后面的值和搭建 zookeeper 集群中 myid 一样，是一个集群中唯一的数，要求是正数。需要保证kafka集群中设置的都不一样。

![image-20200511143906102](img/image-20200511143906102.png)

第二个设置监听器，后面的 IP 地址对应当前的 ip 地址。

![image-20200511143745214](img/image-20200511143745214.png)

第三个是配置 zookeeper 集群的 IP 地址。

![image-20200511143711071](img/image-20200511143711071.png)

第四个修改日志文件路径。

![image-20200511144923676](img/image-20200511144923676.png)

依次修改192.168.8.185,192.168.8.183两台服务器server.properties配置文件

192.168.8.185 

```bash
broker.id=2
listeners=PLAINTEXT://192.168.8.185:9092
zookeeper.connect=192.168.8.190:2181,192.168.8.185:2181,192.168.8.183:2181
log.dirs=/home/kafka/kafka/kafka-logs
```

192.168.8.183

```bash
broker.id=3
listeners=PLAINTEXT://192.168.8.183:9092
zookeeper.connect=192.168.8.190:2181,192.168.8.185:2181,192.168.8.183:2181
log.dirs=/home/kafka/kafka/kafka-logs
```



### 4.4 启动kafka集群

分别进入到三台服务器的kafka目录，执行启动kafka命令

```bash
cd home/kafka/kafka

./bin/kafka-server-start.sh –daemon config/server.properties
```

如果正常的话，则应该不会有任何输出信息



三个节点均要启动；启动无报错，即搭建成功，可以生产和消费消息。



### 4.5 测试kafka集群



任意一台机器上创建话题，例如创建一个名为`first`的话题（指定分区为2）：

```bash
./bin/kafka-topics.sh --create --zookeeper 192.168.8.190:2181,192.168.8.185:2181,192.168.8.183:2181 --replication-factor 2 --partitions 2 --topic first
```

得到如下的输出

<img src="img/image-20200511145236123.png" alt="image-20200511145236123" style="zoom: 200%;" />

（2）查看所有分区

```bash
./bin/kafka-topics.sh --zookeeper 192.168.8.190:2181 --list
```



在一台服务器创建发布者

```bash
./bin/kafka-console-producer.sh --broker-list 192.168.8.190:9092 --topic first
```

在一台服务器创建一个消费者

```
./bin/kafka-console-consumer.sh --bootstrap-server 192.168.8.190:9092 --topic first --from-beginning

```

发布者发送

![image-20200511150104331](img/image-20200511150104331.png)



消费者接收

![image-20200511150132589](img/image-20200511150132589.png)

其他命令

```bash
#停止服务
./bin/kafka-server-stop.sh
```





















附录

若服务器未安装Java 或Java版本为Open JDK，需卸载默认安装的 Open JDK，安装Oracle JDK

```bash
 rpm -qa | grep jdk
```

安装Oracle Java 

下载

https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html

在 安装目录下，将 下载的 JDK压缩文件上传到kafka路径下

解压文件

```bash
tar -zxvf jdk-8u251-linux-x64.tar.gz
```

重命名文件夹为java

```bash
mv jdk-8u251-linux-x64 java
```

用vim 打开/etc/profile 文件

```bash
vim /etc/profile
```

按i进入编辑模式，在配置文件末尾添加JAVA环境变量

```bash
export JAVA_HOME=/home/kafka/java
export JRE_HOME=/home/kafka/java/jre
export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
```

添加环境变量后，需要让该环境变量生效，执行如下代码：

```bash
source /etc/profile
```

检验Java是否安装成功

```bash
echo $JAVA_HOME     
java -version
```

如果设置正确的话，java -version 会输出 java 的版本信息