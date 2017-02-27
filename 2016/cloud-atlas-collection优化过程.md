## cloud_atlas_collection功能 ##
这个应用是实现数据采集功能，只有一个接口，采集用户的信息。用户的信息被分装成json格式通过http POST请求传给应用。应用将数据写入mysql和hdfs。
json格式：
```JSON
{
  "data": [{
    "bussiness_type": "custom_event_log",
    "properties": {
      "create_time": "2016-04-07T16:50:50.336+0800",
      "session_id": "ae45821d-5413-4921-84d2-ef22b07ef954",
      "device_id": "cyjgoodbad",
      "function_id": 21,
      "app_ver": "2016-04-07 16:33:33",
      "account_id":"9876543210"
    }
  },
  {
    "bussiness_type": "login",
    "properties": {
      "create_time": "2016-04-07T16:50:50.336+0800",
      "session_id": "ae45821d-5413-4921-84d2-ef22b07ef954",
      "device_id": "cyjaaaaaaaaaa",
      "function_id": 21,
      "app_ver": "2016-04-07 16:33:33",
      "account_id":"000000000"
    }
  }]
}
```

## 多线程并发，写入flume数据丢失 ##
由于明东哥老婆生产了，陪产假去了，项目组里边只有我与明东哥的技术栈是相似的，所有这个工程就由我接受了。接手后QA性能测试发现单线程300个http请求能够正常将数据写入flume以及mysql没有数据丢失。但是2u并发，每个并发300个请求，就会有数据丢失。

我的第一感觉：有全局变量，多线程并发导致数据被淹没丢失。

cloud_atlas_collection工程使用spring 相关内容。类使用注解实现，由于spring默认采用单例模式，一个类只有一个实例，并由spring管理。跟进代码发现在`FlumeAdapterServiceImpl`类中使用内部类`private FlumeRpcClientFacade clientFacade`。`FlumeAdapterServiceImpl.save()`方法被注解`@Async("collectFlumeExecutor")`定义为异步操作。而`clientFacade`变量在save()方法中 new 实例，数据写入flume后被销毁。所以`clientFacade`变量成为一个全局变量会导致并发问题。

有如下解决方案：
1. 给变量`clientFacade`加同步锁
2. 想方设法将`clientFacade`变为局部变量

### 给变量clientFacade加同步锁 ###

首先采用对变量`clientFacade`加锁，保证给最小的代码添加同步锁。加锁之后可以支撑1000u，思考时间3s 2小时压测。但是无法支撑1500u压测，会出现`异步任务队列溢出，之后的请求被拒绝`。错误信息如下图所示。
【图片】
这个原因是由于spring async设置队列太小导致，消耗速度远远比不上请求速度，导致请求堆积在队列中。
经过多次测试为了满足3000u，思考时间3s测试，修改队列大小到100w。

## flume写入超时 ##
3000u，思考时间10s测试，队列大小100w，压测12小时。出现部分数据写入超时，错误信息如下图所示。
【图片】
引起这个问题有两种可能：
1. flume配置`connect-timeout=20000  request-timeout=20000`，队列堆积，任务请求超时
2. 多线程同步锁竞争消耗太多资源，任务执行速度过慢

鉴于mysql的连接超时时间往往是30000ms，所以修改flume超时设置`connect-timeout=30000  request-timeout=30000`。修改clientFacade为局部变量。

### clientFacade 变为局部变量 ###
由于`clientFacade`的功能在于将数据写入flume，可以直接在`save()`方法中new 实例，完成任务之后在销毁。在异步环境下，使用这种方式会导致内存增加，因为`clientFacade`会跟随任务存放到内存中。

经过调整可以支撑3000u，思考时间10s，12小时测试。

## mysql连接超时 ##
3000u，思考时间5s，12小时测试，会出现如下图所示的错误。
【图片】

按理写flume是不应该与mysql有关联的，但是东哥为了实现无结构表结构，所以在写入数据库时都需要查询一次表结构。每次都要查询表结构明显是不合理的。所以之后的优化是第一次需要查询表结构，之后将表结构缓存到内存中。经过上述修改之后，测试还是会出现伤上述的问题。按理只写入flume是不可能会出现上述问题的啊。所以我做了如下的调试方案尝试找出还有使用数据库的代码。
- 让程序成功执行一次
- 停止数据库服务
- 发送http请求
- 单步调试，看信息是否写入flume

奇怪的是，断开数据库连接报错并没有指明具体是那一段程序出错。并且任务并没有头放入异步队列执行，修改策略
- 让程序成功执行一次
- 任务被投放入队列
- 停止数据库服务
- 发送http请求
- 单步调试，看信息是否写入flume

单步调试，发现写入flume代码并没有向数据库发送连接请求。各种不理解，经过2小时的排查，灵光一闪，觉得会不会是如下代码一起的呢？
```JAVA
@Override
@Async("collectFlumeExecutor")
@Transactional
public String save(CollectRequestBean bean) {
	// 省略代码
}
```

`@Transactional `并不起眼，但是就是它引起的。想想也是合理的，事务在代码开始时肯定会执行`beginTransaction()`方法，代码逻辑执行结束之后才会执行`commit()`方法。事务的连接肯定是需要取得数据库连接的。自己凭空瞎想是没用的，还是得要实际验证。本地验证之后，果然是这个问题。交给QA做3000u，思考时间5s，压测时长12小时，过关。少了这个事务，处理速度肯定会上升，效率也会提高。

## 总结 ##
接手别人的代码往往是比较难的，因为每个人的编程习惯并不一样。东哥的代码很随意，有面向过程的感觉。阅读东哥的代码也是有难度的，需要花费时间很精力。很多神奇的问题往往都是由于细节没有处理清楚，`@Transaction`注解是东哥复制代码时忘记去除了，而之后需要花费更多的时间去修正。复制拷贝需要用心才是。
