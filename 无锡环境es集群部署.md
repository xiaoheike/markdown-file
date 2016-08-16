## 起因 ##
titan 应用需要发布预生产和生产，所以需要部署相关的服务。这个任务由我完成。由于公司将机器的权限全部回收给技术部了，我们只能够通过和技术部的同学沟通，我有编写部署文档，但是作用还是有限的，因为小伙伴很多时候是不熟悉你使用的软件的，所以你需要和对方沟通。无锡的机器的结构比较复杂，比如有三个外网ip，一个内网ip，而平时我们都是使用只有外网ip的机器，所以部署的时候难免会有一些坑。还有就是技术部的小伙伴并不是为你一个人服务的，而是为大家服务的，沟通很费时间，并且你不能够保证对方的操作是按照你的要求进行的。

## 部署 elasticsearch ##
我们有两台机子，使用外网ip 221.228.81.165 和221.228.81.166，内网ip 10.33.6.165 与10.33.6.165。要将这两个机子配成集群。很奇怪的事情，在本地运行的很好的配置，到了无锡环境的机器就是无法启动，报错。
使用的elasticsearch 配置文件：
```XML
cluster.name: es1.5.1-titan-wx
#index.number_of_shards: 5
#index.number_of_replicas: 1
# 本机ip地址
network.bind_host: 10.33.6.165
transport.tcp.port: 9388
http.port: 9288
script.disable_dynamic: false
path.data: /data/elasticsearch/data
path.work: /data/elasticsearch/work
path.logs: /data/elasticsearch/logs

index:
  analysis:
    analyzer:
      ik:
          alias: [ik_analyzer]
          type: org.elasticsearch.index.analysis.IkAnalyzerProvider
      ik_max_word:
          type: ik
          use_smart: false
      ik_smart:
          type: ik
          use_smart: true

index.analysis.analyzer.default.type : "ik"
```
配置了 ik，绑定ip 为 10.33.6.165
```XML
[2016-07-20 00:01:17,246][WARN ][cluster.service          ] [Mad Thinker’s Awesome Android] failed to reconnect to node [Mad Thinker’s Awesome Android][JuWIGiO5R6KAdbaPBW9-Vg][SXWX-081165][inet[/120.195.212.165:9388]]
org.elasticsearch.transport.ConnectTransportException: [Mad Thinker’s Awesome Android][inet[/120.195.212.165:9388]] connect_timeout[30s]
	at org.elasticsearch.transport.netty.NettyTransport.connectToChannels(NettyTransport.java:797)
	at org.elasticsearch.transport.netty.NettyTransport.connectToNode(NettyTransport.java:731)
	at org.elasticsearch.transport.netty.NettyTransport.connectToNode(NettyTransport.java:704)
	at org.elasticsearch.transport.TransportService.connectToNode(TransportService.java:216)
	at org.elasticsearch.cluster.service.InternalClusterService$ReconnectToNodes.run(InternalClusterService.java:562)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)
Caused by: java.net.ConnectException: Connection refused: /120.195.212.165:9388
	at sun.nio.ch.SocketChannelImpl.checkConnect(Native Method)
	at sun.nio.ch.SocketChannelImpl.finishConnect(SocketChannelImpl.java:717)
	at org.elasticsearch.common.netty.channel.socket.nio.NioClientBoss.connect(NioClientBoss.java:152)
	at org.elasticsearch.common.netty.channel.socket.nio.NioClientBoss.processSelectedKeys(NioClientBoss.java:105)
	at org.elasticsearch.common.netty.channel.socket.nio.NioClientBoss.process(NioClientBoss.java:79)
	at org.elasticsearch.common.netty.channel.socket.nio.AbstractNioSelector.run(AbstractNioSelector.java:337)
	at org.elasticsearch.common.netty.channel.socket.nio.NioClientBoss.run(NioClientBoss.java:42)
	at org.elasticsearch.common.netty.util.ThreadRenamingRunnable.run(ThreadRenamingRunnable.java:108)
	at org.elasticsearch.common.netty.util.internal.DeadLockProofWorker$1.run(DeadLockProofWorker.java:42)
	... 3 more
```
有点觉得莫名其妙，因为ip 120.195.212.165 是该机子的一个外网ip，压根不知道为什么冒出来。
## 原因 ##
这个是由于无锡环境的网络有经过专门的处理，elasticsearch 的集群化是通过集群名称实现，所以它应该是通过广播的形式在网路中找到集群，而无锡环境对于这个做了限制，因为elasticsearch 是无法找到自己的归属而集群化。
在上述配置中添加如下配置解决问题：
```XML
discovery.zen.ping.multicast.enabled: false
discovery.zen.ping.unicast.hosts: ["10.33.6.165:9388"]
```
上述配置告诉titan 关闭组播方式二直接通过单点的方式寻找集群信息。

确实应用上了生产环境之后都经过会严格很多，不会出现的问题往往会不断出现。


