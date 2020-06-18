# 搭建Redis 集群

## 准备工作

Redis集群至少需要3个节点，因为投票容错机制要求超过半数节点认为某个节点挂了该节点才是挂了，所以2个节点无法构成集群。要保证集群的高可用，需要每个节点都有从节点，也就是备份节点，所以我们将搭建3主3从的Redis集群。

准备三台服务器，如下图机器的分配情况：

用户名：redis

密码： zaI!Ge6aV@3fw$&b

| IP            | master节点 | salve节点 |
| ------------- | ---------- | --------- |
| 192.168.8.190 | 7001       | 7002      |
| 192.168.8.185 | 7001       | 7002      |
| 192.168.8.183 | 7001       | 7002      |

下载Redis压缩包

官方网站地址: [http://redis.io](http://redis.io/)
下载地址: http://redis.io/download

创建路径module用来存放redis安装文件。

```
mkdir /home/redis/module
```

打开xshell中的文件传输 , 将下载的压缩包传到每台服务器。(路径为/home/redis/module)

我们以192.168.8.190机器安装为例，其他两台只是配置文件的IP地址不一样

## 解压安装

如果centos(linux系统)中没有gcc环境，则需要先安装gcc。如果有就直接下一步。

```bash
#根据linux系统安装对应的版本
sudo apt-get install build-essential #ubuntu
sudo yum install gcc-c++             #centos
```

进入module路径，解压安装文件

```bash
tar -zxvf redis-5.0.8.tar.gz
```

进入刚解压出来的redis目录，开始编译安装

```bash
#进入redis路径
cd redis-5.0.8
make
sudo make install  #需要root权限
#make PREFIX=/home/redis/redisbin install 指定文件路径安装
```

![image-20200513143935458](img/image-20200513143935458.png)

![image-20200513152119444](img/image-20200513152119444.png)

如图显示安装成功



新建myconfig目录，用来存放单机版redis.conf配置文件。

```bash
mkdir /home/redis/module/myconfig
cp /home/redis/module/redis-5.0.8/redis.conf/home/redis/myconfig/
```

## 单主机启动配置

### 修改redis配置文件

```bash
vim myconfig/redis/conf
```

修改 daemonize **yes** （后台运行）



![image-20200513094018837](img/image-20200513094018837.png)

### 启动redis

进入redis启动目录 /usr/local/bin

```bash
cd /usr/local/bin
```



![image-20200513094313552](img/image-20200513094313552.png)

```bash
redis-server /home/yong/redis/module/myconfig/redis.conf
```

![image-20200513094505042](img/image-20200513094505042.png)

查看端口占用情况，redis使用6379端口。

```bash
ps -ef | grep redis
```

![image-20200513094554357](img/image-20200513094554357.png)

### 启动客户端

```bash
redis-cli -p 6379
```

### 关闭客户端

```bash
redis-cli -6379 shutdown
```



## 集群配置

我们还是以192.168.8.190机器安装为例，其他两台只是配置文件的IP不一样

### 创建配置文件

1. 分别创建两个7001和7002的配置文件目录conf，日志目录logs，数据存储目录data在redis_cluster目录中

   ```bash
   mkdir /home/redis/module/redis_cluster
   cd module
   mkdir -p  redis_cluster/7001/conf/
   mkdir -p  redis_cluster/7001/logs/
   mkdir -p  redis_cluster/7001/data/
   
   mkdir -p redis_cluster/7002/conf/
   mkdir -p redis_cluster/7002/logs/
   mkdir -p redis_cluster/7002/data/
   ```

   

2. 创建对应7001redis.conf 配置文件

   ```bash
   vim /home/redis/module/redis_cluster/7001/conf/redis.conf
   ```

   ```bash
   # 绑定服务器域名或IP地址
   bind 192.168.8.190
   # 设置端口，区分集群中Redis的实例
   port 7001
   # 后台运行
   daemonize yes
   # pid进程文件名，以端口号命名
   pidfile /var/run/redis-7001.pid
   # 日志文件名称，以端口号为目录来区分
   logfile /home/redis/module/redis_cluster/7001/logs/redis.log
   # 数据文件存放地址，以端口号为目录名来区分
   dir /home/redis/module/redis_cluster/7001/data
   # 启用集群
   cluster-enabled yes
   # 配置每个节点的配置文件，同样以端口号为名称
   cluster-config-file nodes_7001.conf
   
   ```

   ![image-20200513182111860](img/image-20200513182111860.png)

3. 创建7002的配置文件，并依次修改

   ```bash
   vim /home/redis/module/redis_cluster/7002/conf/redis.conf
   ```

   

   ```bash
   # 绑定服务器域名或IP地址
   bind 192.168.8.190
   # 设置端口，区分集群中Redis的实例
   port 7002
   # 后台运行
   daemonize yes
   # pid进程文件名，以端口号命名
   pidfile /var/run/redis-7002.pid
   # 日志文件名称，以端口号为目录来区分
   logfile /home/redis/module/redis_cluster/7002/logs/redis.log
   # 数据文件存放地址，以端口号为目录名来区分
   dir /home/redis/module/redis_cluster/7002/data
   # 启用集群
   cluster-enabled yes
   # 配置每个节点的配置文件，同样以端口号为名称
   cluster-config-file nodes_7002.conf
   
   ```

   

   ![image-20200513182145328](img/image-20200513182145328.png)

4. 其他两台机器，集群配置跟上面190机器配置只是域名,端口不同，其他设置都是一样的

## 启动集群

1. 在保证上面190，185，183 都配置完成后，分别启动三台服务器的各个节点

   ```bash
   /usr/local/bin/redis-server 7001/conf/redis.conf
   /usr/local/bin/redis-server 7002/conf/redis.conf
   ```

   使用 ps查看端口占用

   ```bash
   ps -ef | grep redis
   ```

   

   ![image-20200513174231713](img/image-20200513174231713.png)

   

   

2. 使用 reids-cli 创建Redis集群 (`使用IP地址`,在任意一台服务器）

   ```bash
   /usr/local/bin/redis-cli --cluster create 192.168.8.190:7001 192.168.8.190:7002 192.168.8.185:7001 192.168.8.185:7002 192.168.8.183:7001 192.168.8.183:7002 --cluster-replicas 1
   
   ```

   

![image-20200513181138317](img/image-20200513181138317.png)

![image-20200513180926193](img/image-20200513180926193.png)



3. 测试集群是否正常：
   在集群中的任意一台测试都可以，如我们可以在190上连接185上的7002节点并添加一个数据。

   ```bash
   /usr/local/bin/redis-cli -c -h 192.168.8.185 -p 7002
   set key001 redis_cluster
   ```


![image-20200513181432802](img/image-20200513181432802.png)

4. 去183上连接7001节点和185上连接7001或者7002看是否可以查询到数据。

   ```bash
   /usr/local/bin/redis-cli -c -h 192.168.8.183 -p 7001
   192.168.8.183> get key001
   
   /usr/local/bin/redis-cli -c -h 192.168.8.185 -p 7002
   192.168.8.185> get key001
   ```

![image-20200513181549266](img/image-20200513181549266.png#pic_center)



![image-20200513181725308](img/image-20200513181725308.png)

出现上面的结果，说明我们搭建的集群运作正常

其他简单查询信息

查看当前集群信息

```bash
cluster info
```

查看集群里有多少个节点

```bash
cluster nodes
```



