## 软硬件基础信息 ##
这些机子是 `资源平台` 自己运维的长乐机子，暂时闲置，所以接过来，用于做 `storm` 集群化部署的测试。
**机子配置信息**
操作系统：CentOS release 6.5 (Final)		
cpu：Intel(R) Xeon(R) CPU E5-2630 v2 @ 2.60GHz
内存：32G
磁盘大小：1.5T
**storm 软件包**
apache-storm-1.0.0.tar.gz
**Python 版本**
Centos 6.5 系统自带有如下版本 Python，满足 `storm` 部署的要求
Python 2.6.6 (r266:84292, Nov 22 2013, 12:16:22)
[GCC 4.4.7 20120313 (Red Hat 4.4.7-4)] on linux2
**JDK**
`storm1.0.0` 官方要求 `JDK1.7` 及以上，本次实验使用 `JDK1.8`
java version "1.8.0_91"
**zookeeper 版本**
zookeeper-3.4.9.tar.gz

## 分配机子职能 ##
由于机子数量不足，一台机子可能有多个职能。
**zookeeper**
172.24.132.173  
172.24.132.143
172.24.132.142
**nimbus**
172.24.132.174
**supbervisor**
172.24.132.143
172.24.132.142

## 主机名与 IP 的映射 ##
编辑各个机子的 `/etc/hosts` 文件，将主机名称与 `IP` 的对应关系加上，例如：
```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.24.132.142  CLTQ-132-142
```

## 配置 jdk ##
1. 用文本编辑器打开/etc/profile
2. 在profile文件末尾加入：
```
export JAVA_HOME=/usr/local/lifecycle/jdk1.8.0_91
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```
3. `source /etc/profile`  使修改生效

注意点：
1. 你要将 `/usr/local/lifecycle/jdk1.8.0_91` 改为本机 `jdk`安装目录
2. linux下用冒号 `:` 来分隔路径
3. CLASSPATH中当前目录 `.` 不能丢,把当前目录丢掉也是常见的错误。
4. export是把这三个变量导出为全局变量。
5. 大小写必须严格区分。

## 安装 zookeeper 集群 ##
`zookeeper-3.4.9.tar.gz` 解压到：`/usr/local/`
```
sudo tar -C /usr/local -xzf zookeeper-3.4.9.tar.gz
```
### 配置 zookeeper 的环境变量(可选) ###
配置 `zookeeper` 的环境变量，这一步是可选的，配置之后能够全局使用 `zookeeper` 相关的命令，没有配置则需要到 `zookeeper` 的安装目录下执行命令
添加如下配置到 `/etc/profile` 文件的最后，并通过命令 `source /etc/profile` 命令使修改后的配置生效
```
#ZOOKEEPER
ZOOKEEPER=/usr/local/zookeeper-3.4.9
PATH=$PATH:$ZOOKEEPER/bin
```

### 修改 zookeeper 的配置文件 ###
首先将 `/usr/local/zookeeper-3.4.6/conf/zoo_sample.cfg` 文件复制一份，并更名为 `zoo.cfg`。如果不需要配置集群，则不修改修改 `zoo.cfg` 文件。要配置集群，则需要将 `zookeeper` 集群信息通过 `server` 配置。
```
tickTime=2000  
initLimit=10  
syncLimit=5  
dataDir=/usr/local/zookeeper-3.4.6/data
dataLogDir=/usr/local/zookeeper-3.4.6/log
clientPort=2181  
server.1=172.24.132.173:2888:3888
server.2=172.24.132.143:2888:3888
server.3=172.24.132.142:2888:3888
```
`server.A=B：C：D`：其中 `A` 是一个数字，表示这个是第几号服务器；`B` 是这个服务器的 `ip` 地址；`C` 表示的是这个服务器与集群中的 `Leader` 服务器交换信息的端口；`D` 表示的是万一集群中的 `Leader` 服务器挂了，需要一个端口来重新进行选举，选出一个新的 `Leader`，而这个端口就是用来执行选举时服务器相互通信的端口。如果是伪集群的配置方式，由于 `B` 都是一样，所以不同的 `zookeeper` 实例通信端口号不能一样，所以要给它们分配不同的端口号。

根据 `dataDir` 和 `dataLogDir` 变量创建相应的目录，建议优先创建，因为有可能使用的 `linux` 账户权限不足，`zookeeper` 无法自动创建这几个目录。

### 创建 myid 文件 ###
在配置文件 `zoo.cfg` 中 `dataDir` 所指路径 `/usr/local/zookeeper-3.4.6/data` 下，新建 `myid` 文件，并写入 `zoo.cfg` 文件的 `server.A` 中 `A` 的数值，在不同机器上的该文件中填写相应的值。本次部署中，`172.24.132.142 的 myid` 文件应该写入数值 `3`；`172.24.132.143 的 myid` 文件应该写入数值 `2`；`172.24.132.173 的 myid` 文件应该写入数值 `1`

### 启动 zookeeper ###
执行命令 `zkServer.sh start` 将会启动 `zookeeper`。而执行命令 `zkServer.sh stop` 将会停止 `zookeeper`。

通过 `jps` 命令，可以看到 `zookeeper` 的进程名：`QuorumPeerMain`。以及执行命令 `zkServer.sh status` 查看 `zookeeper` 集群状态，如下所示：
```
#172.24.132.142
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.9/bin/../conf/zoo.cfg  
Mode: follower  

#172.24.132.143
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.9/bin/../conf/zoo.cfg
Mode: leader  

#172.24.132.173  
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.9/bin/../conf/zoo.cfg  
Mode: follower  
```

### 可以忽略的异常信息 ###
`zookeeper` 集群逐台启动的过程中，查阅 `zookeeper.out`，会有如下异常：
```
2017-06-12 19:58:04,289 [myid:3] - WARN  [WorkerSender[myid=3]:QuorumCnxManager@400] - Cannot open channel to 1 at election address CLTQ-132-173/172.24.132.173:3888
java.net.ConnectException: Connection refused
        at java.net.PlainSocketImpl.socketConnect(Native Method)
        at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:350)
        at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:206)
        at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:188)
        at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
        at java.net.Socket.connect(Socket.java:589)
        at org.apache.zookeeper.server.quorum.QuorumCnxManager.connectOne(QuorumCnxManager.java:381)
        at org.apache.zookeeper.server.quorum.QuorumCnxManager.toSend(QuorumCnxManager.java:354)
        at org.apache.zookeeper.server.quorum.FastLeaderElection$Messenger$WorkerSender.process(FastLeaderElection.java:452)
        at org.apache.zookeeper.server.quorum.FastLeaderElection$Messenger$WorkerSender.run(FastLeaderElection.java:433)
        at java.lang.Thread.run(Thread.java:745)
2017-06-12 19:58:04,289 [myid:3] - INFO  [WorkerSender[myid=3]:QuorumPeer$QuorumServer@149] - Resolved hostname: CLTQ-132-173 to address: CLTQ-132-173/172.24.132.173
```
上述异常可以忽略，因为集群环境中某些子节点还没有启动 `zookeeper`

### 验证 zookeeper 集群 ###
#### AUOK ####
先安装 `nc`：
```
yum install -y nc
```
返回 `imok` 则表明机子的状态是正常的
```
echo ruok | nc 172.24.132.143 2181
imok
```

#### zookeeper 客户端 ####
```
bin/zkCli.sh -server 172.24.132.143:2181
```
如果需要一次性连接 `zookeeper` 集群的多台机子，则可以使用如下语法：
```
bin/zkCli.sh -server 172.24.132.142:2181,172.24.132.173:2181,172.24.132.143:2181
```
输出如下结果：
```
[root@CLTQ-132-142 zookeeper-3.4.9]# bin/zkCli.sh -server 172.24.132.143:2181
Connecting to 172.24.132.143:2181
2017-06-08 16:25:07,196 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.9-1757313, built on 08/23/2016 06:50 GMT
2017-06-08 16:25:07,199 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=CLTQ-132-142
2017-06-08 16:25:07,199 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.8.0_91
2017-06-08 16:25:07,200 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
2017-06-08 16:25:07,200 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/usr/local/lifecycle/jdk1.8.0_91/jre
2017-06-08 16:25:07,200 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/usr/local/zookeeper-3.4.9/bin/../build/classes:/usr/local/zookeeper-3.4.9/bin/../build/lib/*.jar:/usr/local/zookeeper-3.4.9/bin/../lib/slf4j-log4j12-1.6.1.jar:/usr/local/zookeeper-3.4.9/bin/../lib/slf4j-api-1.6.1.jar:/usr/local/zookeeper-3.4.9/bin/../lib/netty-3.10.5.Final.jar:/usr/local/zookeeper-3.4.9/bin/../lib/log4j-1.2.16.jar:/usr/local/zookeeper-3.4.9/bin/../lib/jline-0.9.94.jar:/usr/local/zookeeper-3.4.9/bin/../zookeeper-3.4.9.jar:/usr/local/zookeeper-3.4.9/bin/../src/java/lib/*.jar:/usr/local/zookeeper-3.4.9/bin/../conf:.:/usr/local/lifecycle/jdk1.8.0_91/lib/dt.jar:/usr/local/lifecycle/jdk1.8.0_91/lib/tools.jar
2017-06-08 16:25:07,200 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
2017-06-08 16:25:07,200 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/tmp
2017-06-08 16:25:07,200 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
2017-06-08 16:25:07,201 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Linux
2017-06-08 16:25:07,201 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=amd64
2017-06-08 16:25:07,201 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=2.6.32-431.el6.x86_64
2017-06-08 16:25:07,201 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=root
2017-06-08 16:25:07,201 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/root
2017-06-08 16:25:07,201 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/usr/local/zookeeper-3.4.9
2017-06-08 16:25:07,202 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=172.24.132.143:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@799f7e29
Welcome to ZooKeeper!
2017-06-08 16:25:07,219 [myid:] - INFO  [main-SendThread(172.24.132.143:2181):ClientCnxn$SendThread@1032] - Opening socket connection to server 172.24.132.143/172.24.132.143:2181. Will not attempt to authenticate using SASL (unknown error)
JLine support is enabled
2017-06-08 16:25:07,266 [myid:] - INFO  [main-SendThread(172.24.132.143:2181):ClientCnxn$SendThread@876] - Socket connection established to 172.24.132.143/172.24.132.143:2181, initiating session
[zk: 172.24.132.143:2181(CONNECTING) 0] 2017-06-08 16:25:07,286 [myid:] - INFO  [main-SendThread(172.24.132.143:2181):ClientCnxn$SendThread@1299] - Session establishment complete on server 172.24.132.143/172.24.132.143:2181, sessionid = 0x25c86c7c3bf0000, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
```
创建文件夹：
```
[zk: 172.24.132.143:2181(CONNECTED) 4] create /c1project c1project
Created /c1project
[zk: 172.24.132.143:2181(CONNECTED) 5] get /c1project c1project
c1project
cZxid = 0x100000002
ctime = Thu Jun 08 16:30:47 CST 2017
mZxid = 0x100000002
mtime = Thu Jun 08 16:30:47 CST 2017
pZxid = 0x100000002
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 9
numChildren = 0
```
在集群的其他机子上能够查询到该文件，则说明 `zookeeper` 集群是创建成功了的。

## storm 集群部署 ##
`apache-storm-1.0.0.tar.gz` 解压到： `/usr/local/`：
```
sudo tar -C /usr/local -xzf apache-storm-1.1.0.tar.gz
```
### 修改配置 ###
添加如下几个主要参数就可以
```
storm.zookeeper.servers:
    - "172.24.132.142"
    - "172.24.132.143"
    - "172.24.132.173"

storm.zookeeper.port: 2181
storm.local.dir: "/usr/local/apache-storm-1.0.0/data"

nimbus.seeds: ["172.24.132.174"]

supervisor.slots.ports:
    - 6700
    - 6701
    - 6702
    - 6703
```
- `storm.zookeeper.servers`：`zookeeper` 集群的 `IP`
- `storm.zookeeper.port`：`zookeeper` 集群的端口，如果不是默认端口 `2181` 则需要设置
- `nimbus.seeds`：可以作为 `nimbus` 的机子

### 拷贝配置完成的软件 ###
将配置修改完成的软件拷贝到其他机子上：
```
scp -r apache-storm-1.0.0 172.24.132.142:/usr/local/
```
### 启动 nimbus 和 supervisor ###
`nimbus` 与 `supervisor` 可以部署在同一台机子，但是建议分开，避免相互影响。
**172.24.132.174 后台运行 `nimbus`**
```
bin/storm nimbus >/dev/null 2>&1 &
```
**172.24.132.142 172.24.132.143后台运行 `supervisor`**
```
bin/storm supervisor >/dev/null 2>&1 &
```
**172.24.132.174 后台运行 `storm ui`**
`storm ui` 得要在 `nimbus` 机子上运行，不能够在 `supervisor` 机子上运行
```
bin/storm ui >/dev/null 2>&1 &
```

### 测试 storm 集群 ###
使用 `storm` 自带测试例子测试，从 `github` 下载工程到本地：
```
git clone https://github.com/apache/storm.git
```
拉取对应版本的分支，本次部署使用分支 `1.0.x-branch`：
```
git checkout 1.0.x-branch
```
进入目录 `storm/examples/storm-starter`，打包：
```
mvn package
```
打包很有可能失败，往往是各种的 `jar` 包无法下载，可以借助 IDE，将需要的依赖下载到本地。本次部署一直无法下载的 `kafka-avro-serializer 1.0`，可以通过如下连接直接下载，[kafka-avro-serializer 1.0](http://packages.confluent.io/maven/io/confluent/kafka-avro-serializer/1.0/kafka-avro-serializer-1.0.jar)，下载完成的 `jar` 包得要放到 `.m2\repository\io\confluent\kafka-avro-serializer\1.0` 下。
生成目录：`storm\examples\storm-starter\target`，其下的 `storm-starter-1.0.2.jar` 可以作为测试 `jar`
执行命令：
```
bin/storm jar /usr/local/apache-storm-1.0.0/storm-starter-1.0.2.jar org.apache.storm.starter.ExclamationTopology et
```
`et` 是 `topology` 的名字，任意，但是不能够省略。
```
[root@CLTQ-132-174 apache-storm-1.0.0]# bin/storm jar /usr/local/apache-storm-1.0.0/storm-starter-1.0.2.jar org.apache.storm.starter.ExclamationTopology et
Running: /usr/local/lifecycle/jdk1.8.0_91/bin/java -client -Ddaemon.name= -Dstorm.options= -Dstorm.home=/usr/local/apache-storm-1.0.0 -Dstorm.log.dir=/usr/local/apache-storm-1.0.0/logs -Djava.library.path=/usr/local/lib:/opt/local/lib:/usr/lib -Dstorm.conf.file= -cp /usr/local/apache-storm-1.0.0/storm-starter-1.0.2.jar:/usr/local/apache-storm-1.0.0/lib/servlet-api-2.5.jar:/usr/local/apache-storm-1.0.0/lib/clojure-1.7.0.jar:/usr/local/apache-storm-1.0.0/lib/slf4j-api-1.7.7.jar:/usr/local/apache-storm-1.0.0/lib/asm-5.0.3.jar:/usr/local/apache-storm-1.0.0/lib/kryo-3.0.3.jar:/usr/local/apache-storm-1.0.0/lib/log4j-core-2.1.jar:/usr/local/apache-storm-1.0.0/lib/log4j-over-slf4j-1.6.6.jar:/usr/local/apache-storm-1.0.0/lib/storm-rename-hack-1.0.0.jar:/usr/local/apache-storm-1.0.0/lib/objenesis-2.1.jar:/usr/local/apache-storm-1.0.0/lib/reflectasm-1.10.1.jar:/usr/local/apache-storm-1.0.0/lib/storm-core-1.0.0.jar:/usr/local/apache-storm-1.0.0/lib/minlog-1.3.0.jar:/usr/local/apache-storm-1.0.0/lib/log4j-slf4j-impl-2.1.jar:/usr/local/apache-storm-1.0.0/lib/log4j-api-2.1.jar:/usr/local/apache-storm-1.0.0/lib/disruptor-3.3.2.jar:/usr/local/apache-storm-1.0.0/storm-starter-1.0.2.jar:/usr/local/apache-storm-1.0.0/conf:/usr/local/apache-storm-1.0.0/bin -Dstorm.jar=/usr/local/apache-storm-1.0.0/storm-starter-1.0.2.jar org.apache.storm.starter.ExclamationTopology et
531  [main] INFO  o.a.s.StormSubmitter - Generated ZooKeeper secret payload for MD5-digest: -7107387084739244238:-5556405776831141841
583  [main] INFO  o.a.s.s.a.AuthUtils - Got AutoCreds []
624  [main] INFO  o.a.s.StormSubmitter - Uploading topology jar /usr/local/apache-storm-1.0.0/storm-starter-1.0.2.jar to assigned location: /usr/local/apache-storm-1.0.0/data/nimbus/inbox/stormjar-1ee74fb2-7356-4e0c-8f97-f0c71ff4c884.jar
Start uploading file '/usr/local/apache-storm-1.0.0/storm-starter-1.0.2.jar' to '/usr/local/apache-storm-1.0.0/data/nimbus/inbox/stormjar-1ee74fb2-7356-4e0c-8f97-f0c71ff4c884.jar' (73376663 bytes)
[==================================================] 73376663 / 73376663
File '/usr/local/apache-storm-1.0.0/storm-starter-1.0.2.jar' uploaded to '/usr/local/apache-storm-1.0.0/data/nimbus/inbox/stormjar-1ee74fb2-7356-4e0c-8f97-f0c71ff4c884.jar' (73376663 bytes)
1029 [main] INFO  o.a.s.StormSubmitter - Successfully uploaded topology jar to assigned location: /usr/local/apache-storm-1.0.0/data/nimbus/inbox/stormjar-1ee74fb2-7356-4e0c-8f97-f0c71ff4c884.jar
1029 [main] INFO  o.a.s.StormSubmitter - Submitting topology et in distributed mode with conf {"storm.zookeeper.topology.auth.scheme":"digest","storm.zookeeper.topology.auth.payload":"-7107387084739244238:-5556405776831141841","topology.workers":3,"topology.debug":true}
1471 [main] INFO  o.a.s.StormSubmitter - Finished submitting topology: et
```

## 错误 ##
### Failed to Sync Supervisor ###
```
16373 [Thread-10] ERROR o.a.s.d.s.ReadClusterState - Failed to Sync Supervisor
java.lang.RuntimeException: java.lang.InterruptedException
        at org.apache.storm.utils.Utils.wrapInRuntime(Utils.java:1531) ~[storm-core-1.1.0.jar:1.1.0]
        at org.apache.storm.zookeeper.zookeeper.getChildren(zookeeper.java:265) ~[storm-core-1.1.0.jar:1.1.0]
        at org.apache.storm.cluster.ZKStateStorage.get_children(ZKStateStorage.java:174) ~[storm-core-1.1.0.jar:1.1.0]
        at org.apache.storm.cluster.StormClusterStateImpl.assignments(StormClusterStateImpl.java:153) ~[storm-core-1.1.0.jar:1.1.0]
        at org.apache.storm.daemon.supervisor.ReadClusterState.run(ReadClusterState.java:126) [storm-core-1.1.0.jar:1.1.0]
        at org.apache.storm.event.EventManagerImp$1.run(EventManagerImp.java:54) [storm-core-1.1.0.jar:1.1.0]
Caused by: java.lang.InterruptedException
        at java.lang.Object.wait(Native Method) ~[?:1.8.0_91]
        at java.lang.Object.wait(Object.java:502) ~[?:1.8.0_91]
        at org.apache.storm.shade.org.apache.zookeeper.ClientCnxn.submitRequest(ClientCnxn.java:1342) ~[storm-core-1.1.0.jar:1.1.0]
        at org.apache.storm.shade.org.apache.zookeeper.ZooKeeper.getChildren(ZooKeeper.java:1588) ~[storm-core-1.1.0.jar:1.1.0]
        at org.apache.storm.shade.org.apache.zookeeper.ZooKeeper.getChildren(ZooKeeper.java:1625) ~[storm-core-1.1.0.jar:1.1.0]
        at org.apache.storm.shade.org.apache.curator.framework.imps.GetChildrenBuilderImpl$3.call(GetChildrenBuilderImpl.java:226) ~[storm-core-1.1.0.jar:1.1.0]
        at org.apache.storm.shade.org.apache.curator.framework.imps.GetChildrenBuilderImpl$3.call(GetChildrenBuilderImpl.java:219) ~[storm-core-1.1.0.jar:1.1.0]
        at org.apache.storm.shade.org.apache.curator.RetryLoop.callWithRetry(RetryLoop.java:109) ~[storm-core-1.1.0.jar:1.1.0]
        at org.apache.storm.shade.org.apache.curator.framework.imps.GetChildrenBuilderImpl.pathInForeground(GetChildrenBuilderImpl.java:216) ~[storm-core-1.1.0.jar:1.1.0]
        at org.apache.storm.shade.org.apache.curator.framework.imps.GetChildrenBuilderImpl.forPath(GetChildrenBuilderImpl.java:207) ~[storm-core-1.1.0.jar:1.1.0]
        at org.apache.storm.shade.org.apache.curator.framework.imps.GetChildrenBuilderImpl.forPath(GetChildrenBuilderImpl.java:40) ~[storm-core-1.1.0.jar:1.1.0]
        at org.apache.storm.zookeeper.zookeeper.getChildren(zookeeper.java:260) ~[storm-core-1.1.0.jar:1.1.0]
        ... 4 more
```
部署的 `storm` 服务器版本为 `1.1.0`，而提交给 `storm` 跑的 `jar` 包为：`storm-starter-1.0.2.jar`。版本不对应导致上面的问题。应该是 `storm-core-1.1.0.jar` 的代码与 `storm-core-1.0.2.jar` 相差较大，或者修改了通信的协议导致。将 `storm` 服务器版本修改为 `1.0.0` 即可解决问题
### [有多个 supervisor 但 storm ui 上只显示一个](http://blog.csdn.net/cuilanbo/article/details/43984121) ###
具体现象就是启动了多个 `supervisor`，单在 `ui` 上只显示一个（也有可能是多个 `supervisor` 中的某几个看上去被“合并”了），`kill` 掉其中任意一个 `supervisor`，另一个就出现。
例如本例中有两个 `supervisor`，`172.24.132.143` 和 `172.24.132.142`，但是通过接口请求，每次都只会显示其中的一个，但是现实的机子是交替出现的：
```
curl -X GET \
  http://172.24.132.174:8080/api/v1/supervisor/summary

{
    "supervisors": [
        {
            "totalMem": 3072,
            "host": "CLTQ-132-142",
            "id": "26cdf80b-394e-47e8-a82a-ea78f82e7c22",
            "uptime": "14h 16m 17s",
            "totalCpu": 400,
            "usedCpu": 0,
            "logLink": "http://CLTQ-132-142:8000/daemonlog?file=supervisor.log",
            "usedMem": 2496,
            "slotsUsed": 3,
            "version": "1.0.0",
            "slotsTotal": 4,
            "uptimeSeconds": 51377
        }
    ],
    "schedulerDisplayResource": false,
    "logviewerPort": 8000
}
```
解决方案：`storm.yaml` 文件中有配置 `storm.local.dir: "/usr/local/apache-storm-1.0.0/data"`，`local.dir` 所指目录，重启即可解决问题。原因是由于部署时通过 `linux scp` 命令直接分发软件到其他机子，残留了 `local.dir` 的东西，而 `storm` 是根据 `local.dir` 中的某一个或一些文件计算出一个 `supervisor id` 的。删除 `local.dir` 后，会重新生成 `id`。
### Could not find or load main class org.apache.storm.starter.ExclamationTopology ###
```
bin/storm jar /usr/local/apache-storm-1.0.0/storm-starter-1.0.2.jar org.apache.storm.starter.ExclamationTopology et
```
第一确保 `storm-starter-1.0.2.jar` 的路径是正确的；第二保证 `packagename.ExclamationTopology`，包名`packagename` 与 类名 `ExclamationTopology` 是正确的


## 参考文档 ##
1. [Documentation](http://storm.apache.org/releases/1.0.0/index.html)
2. [Setting up a Storm Cluster](http://storm.apache.org/releases/1.0.0/Setting-up-a-Storm-cluster.html)
3. [storm ui显示supervisor个数与实际不符的解决](http://blog.csdn.net/cuilanbo/article/details/43984121)
4. [Storm安装教程_CentOS6.4/Storm0.9.6](http://www.powerxing.com/install-storm/)
5. [Storm实战 (1) storm1.0.0集群安装](http://aperise.iteye.com/blog/2295227)
6. [Storm配置项详解](http://xstarcd.github.io/wiki/Cloud/storm_config_detail.html)
