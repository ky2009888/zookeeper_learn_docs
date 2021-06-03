  Zookeeper 集群中只要有过半的节点是正常的情况下，那么整个集群对外就是可用的。正是基于这个

特性，要将 ZK 集群的节点数量要为奇数（2n+1：如 3、5、7 个节点）较为合适。

**系统环境：CentOS 6.6\_x64 jdk1.8**

  **服务器IP和端口规划：**

服务器 1：192.168.1.11 端口：2181、2888、3888

服务器 2：192.168.1.12 端口：2181、2888、3888

服务器 3：192.168.1.13 端口：2181、2888、3888

**1、 修改操作系统的/etc/hosts 文件，添加 IP 与主机名映射：**

```
vim /etc/hosts
# zookeeper cluster servers
192.168.1.11 zk-01
192.168.1.12 zk-02
192.168.1.13 zk-03
```

**2、 下载或上传 zookeeper-3.4.6.tar.gz 到/home/wusc/zookeeper 目录：**

```
cd /usr/local/src
wget http://apache.fayea.com/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
```

**3、 解压 zookeeper 安装包，并建立软连接**：

```
tar -zxvf zookeeper-3.4.6.tar.gz -C /usr/local/
ln -sv /usr/local/zookeeper-3.4.6 /usr/local/zookeeper
```

**4、 在各 zookeeper 节点下创建以下目录：**

```
mkdir -pv /data/zookeeper/{data,log} #数据目录和日志目录
```

**5、 将/usr/local/zookeeper/conf 目录下的 zoo\_sample.cfg 文件拷贝一份，命名为 zoo.cfg:**

```
cp zoo_sample.cfg zoo.cfg
```

**6、 修改 zoo.cfg 配置文件：#3台的配置一样**

```
vim /usr/local/zookeeper/conf/zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data/zookeeper/
dataLogDir=/data/zookeeper/log
clientPort=2181
server.1=zk-01:2888:3888
server.2=zk-02:2888:3888
server.3=zk-03:2888:3888
```

参数说明:

tickTime=2000

tickTime 这个时间是作为 Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔,也就是每

个 tickTime 时间就会发送一个心跳。

initLimit=10

initLimit 这个配置项是用来配置 Zookeeper 接受客户端（这里所说的客户端不是用户连接 Zookeeper

服务器的客户端,而是 Zookeeper 服务器集群中连接到 Leader 的 Follower 服务器）初始化连接时最长

能忍受多少个心跳时间间隔数。当已经超过 10 个心跳的时间（也就是 tickTime）长度后 Zookeeper 服

务器还没有收到客户端的返回信息,那么表明这个客户端连接失败。总的时间长度就是 10\*2000=20 秒。

syncLimit=5

syncLimit 这个配置项标识 Leader 与 Follower 之间发送消息,请求和应答时间长度,最长不能超过多少个 tickTime 的时间长度,总的时间长度就是 5\*2000=10 秒。

dataDir=/data/zookeeper/

dataDir顾名思义就是Zookeeper保存数据的目录,默认情况下Zookeeper将写数据的日志文件也保存在

这个目录里。

clientPort=2181

clientPort 这个端口就是客户端（应用程序）连接 Zookeeper 服务器的端口,Zookeeper 会监听这个端

口接受客户端的访问请求。

server.A=B:C:D

server.1=zk-01:2888:3888

server.2=zk-02:2888:3888

server.3=zk-03:2888:3888

A 是一个数字,表示这个是第几号服务器；

B 是这个服务器的 IP 地址（或者是与 IP 地址做了映射的主机名） ；

C 第一个端口用来集群成员的信息交换,表示这个服务器与集群中的 Leader 服务器交换信息的端口；

D 是在 leader 挂掉时专门用来进行选举 leader 所用的端口。

注意：如果是伪集群的配置方式，不同的 Zookeeper 实例通信端口号不能一样，所以要给它们分配不

同的端口号。

**7、 在 dataDir=/data/zookeeper/ 下创建 myid 文件**

编辑 myid 文件，并在对应的 IP 的机器上输入对应的编号。如在 node-01 上，myid 文件内容就是

1,node-02 上就是 2，node-03 上就是 3：

```
echo 1 > /data/zookeeper/myid
echo 2 > /data/zookeeper/myid
echo 3 > /data/zookeeper/myid
```

8、 在防火墙中打开要用到的端口 2181、2888、3888

```
vim /etc/sysconfig/iptables
## zookeeper
-A INPUT -m state --state NEW -m tcp -p tcp --dport 2181 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 2888 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3888 -j ACCEPT
```

重启防火墙：

```
service iptables restart
```

查看防火墙端口状态：

```
iptables -L
```

9、 启动并测试 zookeeper:

```
/usr/local/zookeeper/bin/zkServer.sh start
```

```
jps #查看进程
247 QuorumPeerMain #QuorumPeerMain是zookeeper进程,说明启动正常
tailf -200 /data/zookeeper/log/zookeeper.out #查看日志，没报错，有集群配置信息就说明集群配置成功
```

注意：zookeeper集群配置每个节点的配置都是一样的