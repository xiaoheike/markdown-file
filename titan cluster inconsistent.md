# titan cluster inconsistent #
## 起因 ##
在本地压测环境部署了有两台机子的集群，往 titan 集群中插入数据，发现有字段是丢失的，比如 `search_path_string`。由于缺少了这个字段，基于 es 分词检索的压测无法进行。缺少该字段之后分词检索不起作用了，所以被我发现了。
## 寻找原因 ##
跟开发该部分代码的小伙伴沟通之后，他表示这部分代码已经很久没有修改，我通过 git 查看，确实是没有什么修改的。之后我们往其他的测试环境发送请求，发现数据是完整插入的，所以觉得是我打的war 包有问题。由于目前我们的分支有十几个，难道是我打包的时候讲分支搞错了？所以我麻烦小伙伴打包了一个新的war包给我，使用该 war 之后，测试插入了一条资源，确实是没有问题的。我觉得奇怪的地方在于，打 war 包的分支是 localpressure 分支，该分支在打 war 包之前刚刚完成住分支的合并，也就是说各个分支的代码应该是没有什么分别的，就算是我打错了分支，也不会有什么问题才是。
如果说是我的 IDE 有问题，那么这也很难让人相信，因为我在打包的时候都会做一些标记，就比如：“esp-lifecycle-4f287b4-94--localpressure”，其中 4f287b4 代表 git 提交版本号，每次提交 git 都会生成一个，localpressure 代表打包使用的 git 分支，94 代表发布的机子。我很难相信是我打包的代码有问题，二则 IDE 出现问题这概率也太小了，是在有点想不通。
事实胜于雄辩，竟然事实证明 war 有问题也就无所谓了。新的 war 包经过压测，发现性能差了好多，300 并发，原本 0.42s，而更新 war 包之后响应时间变成了 1.08s。相差太大了。所以我觉得是代码有一些差别，由于硬件方面的部署都没有任何的修改，和小伙伴沟通之后，知道有 `TitanResultParse2` 这个类，这个类是解析 titan 返回值的核心类。我将 最新分支打包的 war 包与小伙伴给我的 war 包做对比，发现小伙伴给我的 war 里边竟然没有 `TitanResultParse2` 类，我瞬间懂了，小伙伴估计给了我错误的 war 包，是优化之前的。
既然小伙伴提供的 war 包不靠谱，那么还是只能够靠自己了，所以我自己使用 IDE 在最新 localpressure 分支上打了 war 包，并部署到 94 机子的 tomcat 上，启动。我创建了一条资源，发现还是不行，难道真的是我的 IDE 有问题？
我找到 gremlin-server.log 发现其中有如下的异常信息：
```
432558734 2016-08-16 11:17:37 [gremlin-server-exec-177] WARN  org.apache.tinkerpop.gremlin.server.op.AbstractEvalOpProcessor  - Exception processing a script on request [RequestMessage{, requestId=6c17b043-feff-443b-9dbe-6a3036b215f5, op='eval', processor='', args={gremlin=g.V().has(primaryCategory,'identifier',identifier).next().addEdge('has_coverage',g.V(coverageNodeId).next(),'identifier',edgeIdentifier, 'target_type', target_type, 'strategy', strategy, 'target', target).id(), bindings={identifier=3bbc3c1b-8d7f-4b2f-b5cb-c30fb86228cf, coverageNodeId=-1, primaryCategory=assets, target_type=Org, edgeIdentifier=6c4846c8-b332-4444-85f5-b2aa19692dce, strategy=OWNER, target=nd}, batchSize=20}}].
org.apache.tinkerpop.gremlin.process.traversal.util.FastNoSuchElementException
432559436 2016-08-16 11:17:37 [gremlin-server-exec-19] WARN  org.apache.tinkerpop.gremlin.server.op.AbstractEvalOpProcessor  - Exception processing a script on request [RequestMessage{, requestId=58f7008c-e7e2-4156-82f5-74d2a42833b4, op='eval', processor='', args={gremlin=g.V().has(primaryCategory,'identifier',identifier).next().addEdge('has_coverage',g.V(coverageNodeId).next(),'identifier',edgeIdentifier, 'target_type', target_type, 'strategy', strategy, 'target', target).id(), bindings={identifier=789d70f7-4d32-4541-bd48-246291e88841, coverageNodeId=-1, primaryCategory=assets, target_type=Org, edgeIdentifier=d9aad0b9-4ad1-43f3-aa96-be55a530de6b, strategy=OWNER, target=nd}, batchSize=20}}].
org.apache.tinkerpop.gremlin.process.traversal.util.FastNoSuchElementException
432561671 2016-08-16 11:17:39 [gremlin-server-exec-72] WARN  org.apache.tinkerpop.gremlin.server.op.AbstractEvalOpProcessor  - Exception processing a script on request [RequestMessage{, requestId=a2a001d6-ac28-4055-b02e-1a4da636ed7f, op='eval', processor='', args={gremlin=g.V().has(primaryCategory,'identifier',identifier).next().addEdge('has_coverage',g.V(coverageNodeId).next(),'identifier',edgeIdentifier, 'target_type', target_type, 'strategy', strategy, 'target', target).id(), bindings={identifier=2eb491bb-2440-4495-a17b-e065e4e4aacc, coverageNodeId=-1, primaryCategory=assets, target_type=Org, edgeIdentifier=11f2dd3b-4065-4686-8a1f-1b11e721f36a, strategy=OWNER, target=nd}, batchSize=20}}].
org.apache.tinkerpop.gremlin.process.traversal.util.FastNoSuchElementException
432562453 2016-08-16 11:17:40 [gremlin-server-exec-38] WARN  org.apache.tinkerpop.gremlin.server.op.AbstractEvalOpProcessor  - Exception processing a script on request [RequestMessage{, requestId=6051f058-75df-44c9-9f5a-bf87bd5f5808, op='eval', processor='', args={gremlin=g.V().has(primaryCategory,'identifier',identifier).next().addEdge('has_coverage',g.V(coverageNodeId).next(),'identifier',edgeIdentifier, 'target_type', target_type, 'strategy', strategy, 'target', target).id(), bindings={identifier=649044bc-f67a-43e7-b063-85417b0fbcf1, coverageNodeId=-1, primaryCategory=assets, target_type=Org, edgeIdentifier=de500aaa-d37e-492e-81f7-fc69c66e7d04, strategy=OWNER, target=nd}, batchSize=20}}].
org.apache.tinkerpop.gremlin.process.traversal.util.FastNoSuchElementException
```
有大量 `FastNoSuchElementException` 异常信息，这个异常表明有节点无法找到。我突然想到一个可能性，本地压测的 titan 是使用 42,89 两台机子部署成集群，数据存储使用 cassandra，检索使用 elasticsearch，而 sdp 提供的 cassandra 有 3 个顶点。我之前使用 titan 之间的数据同步确实是需要一些时间，具体时间没有注意，但是一定是在 10s 以上。titan 是使用轮询的方式将请求分发给不同的机子，也就是说，第一次请求到达 42 机子，第二次请求就会到达 89 机子。假设有一个新增资源的请求到达 42 机子，那么我立即在 89 机子检索该新增资源是无法查询到的。
现在的问题就在于 titan 资源之间同步是需要时间的，而我们的程序实现中，插入顶点与边是在不同的事务中完成的，也就是说，创建顶点和创建边的脚本会本分发到不同的机器上执行，这样完全有可能出现创建边时顶点无法找到的情况，从而导致边创建失败。这就合理解释了为什么会丢失一些字段，例如：`search_path_string`。
当然这些都是我自己的假设，所以需要验证。验证方式很简单，还是使用现在感觉有问题的 war 包，将 42 机子的 titan 关闭，这样所有的创建资源请求就都会到 89 机子上，就不会出现找不到节点的情况。实验结果证明了我的猜测，确实是 titan 集群数据同步有问题。
## 解决问题方案 ##
#### 方案一 ###
因为 titan 是使用 cassandra 作为数据存储层，所以导致这个问题的原因应该是 cassandra 的一致性问题。titan 中有两个参数是用于解决数据一致性问题的，不管有没有采用，都需要做一个实验。
做了两个实验，第一个，新增参数配置：
```
storage.cassandra.write-consistency-level = ALL
storage.cassandra.read-consistency-level = QUORUM
```
第二个实验，新增参数配置：
```
storage.cassandra.write-consistency-level = ALL
storage.cassandra.read-consistency-level = ALL
```
两个实验都失败了， 还是有字段会丢失。及时字段没有丢失，我也觉得这种做法不可靠，因为对性能的影响太大了。这就好比 mysql 中为了解决 脏读，不可重复读，幻读而将事务的等级设置为序列化。
至于为什么已经将一致性的等级提供到最高级还是无法解决问题，这个我现在还不知道，因为这个设计到 cassandra 的处理机制，而我对 cassandra 一窍不通。
### 方案二 ###
其实我们可以创建一个 client（应用与 titan 通信的方式），这个 client 至于 titan 中的一台机子通信，比如 42 机子，那么保存数据时就不会出现请求被分发给不同机子的请求了，那样资源就可以被创建成功了。优化版本就可以对不同机子创建不同的 client，咋们也做一个轮询就好。
## 收获 ##
- 事情多而杂时必须要做好记录与备份，要不然等出问题了你自己都不知道使用了那个东西。就比如每次部署 war 包都需要备份，并做备注，要不然自己真心会糊涂的。
- 出现问题先不要急，可以猜想，但是要找出真凭实据，需要和开发相关代码的小伙伴沟通
- 别人提供的东西可能就是有问题的，哈哈，这个很难避免，毕竟我们是需要团队合作的，我们得要建立在信任对方的基础上进行开发与合作，当别人提供东西给你时，尽量做相关的确认吧。
