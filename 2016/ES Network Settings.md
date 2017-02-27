# 网络设置 #
Elasticsearch 缺省情况下是绑定 localhost。对于本地开发服务是足够的（如果你在相同机子上启动多个节点，它还可以形成一个集群），但是你需要配置[基本的网络设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html#common-network-settings)，为了能够在实际的多服务器生产集群中运行。
***WARNING***：**注意网络配置，永远不要暴露未受保护的节点到公网上**
## 常用的网络配置 ##
**network.hos**
        节点将绑定到一个主机名或者 ip 地址并且会将该这个节点通知集群中的其他节点。接受 ip 地址，主机名，指定值或者包含这些值的数组
        默认值：_local_
**discovery.zen.ping.unicast.hosts**
        为了加入集群，一个节点至少需要知道集群中其他节点的主机名或者 ip 地址。这个设置提供初始其他节点列表，当前节点将尝试联系。接收 ip 地址或者主机名。
        默认值：["127.0.0.1", "[::1]"]
**http.port**
        HTTP 请求通信端口。接收单值或者一个范围。如果指定一个范围，该节点将会绑定范围的第一个可用顶点。
        默认值：9200-9300
**transport.tcp.port**
        节点间通信端口。接收单值或者一个范围。如果指定一个范围，该节点将会绑定范围的第一个可用顶点。
        默认值：9300-9400
## network.host 的特殊值 ##
以下特殊值将可以传递给 **network.host**：
- _[networkInterface]_      网络接口的地址，例如 _en0_。
- _local_       系统中的回路地址，例如 127.0.0.1。
- _site_        系统中任何的本地站点地址，例如 192.168.0.1。
- _global_      系统中的任何全局作用域地 8.8.8.8。
### IPv4 vs IPv6 ###
默认情况下这些特殊值都可以在 IPv4 和IPv6 中使用，但是你可以使用 :ipv4，:ipv6 字符限制使用。例如，_en0:ipv4_ 将绑定 en0 接口的 IPv4 地址。
***Tip***：**在云上使用，更多特别设定可用，当你在 AWS 云或者 Google Compute Engine 云上使用时**
## 高级网络配置 ##
在[常用的网络配置](#常用的网络配置)中解释的 network.host 是快捷方式，同时设置绑定地址和发布地址。在高级使用情况下，例如在一个代理服务器中运行，你可能需要设置如下不同的值：
**network.bind_host**
        这将指定用于监听请求的网络接口。一个节点可以绑定多个接口，例如有两块网卡，一个本地站点地址，一个本地地址。
        默认值：network.host
**network.publish_host**
        发布地址，一个单一地址，用于通知集群中的其他节点，以便其他的节点能够和它通信。当前，一个 elasticsearch 节点可能被绑定到多个地址，但是仅仅有一个发布地址。如果没有指定，这个默认值将为 network.host 配置中的最好地址，以 IPv4/Ipv6 堆栈性能，之后以稳定性排序。
上述两个设置可以向 network.host 那样被设置--他们都接受 IP 地址，主机名和特定值
## 高级 TCP 设置 ##
任何使用 TCP（像 HTTP 和 Transport 模块）共享如下设置：
- network.tcp.no_delay      开启或关闭 TCP 无延迟设置。默认值为 true。
- network.tcp.keep_alive      开启或关闭 TCP 长连接，默认值为 true。
- network.tcp.reuse_address     一个地址是否可以被重用。在非 windows 机子上默认值为 true。
- network.tcp.send_buffer_size      TCP 发送缓冲区大小（以[size unit](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#size-units)指定）。没有默认值。
- network.tcp.receive_buffer_size   TCP 接收缓冲区大小（以[size unit](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#size-units)指定）。没有默认值。
## Transport 和 HTTP 协议 ##
一个Elasticsearch节点暴露两个网络协议配置继承上面的设置，但可独立地进一步配置两个网络协议：
**TCP Transport**
        用于集群中节点之间的通信。
**HTTP**
        暴露基于 HTTP JSON 请求接口，被所有客户端使用，比局限于 Java 客户端。

翻译自：https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html

2016年7月22日





