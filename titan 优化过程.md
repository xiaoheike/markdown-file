## 起因 ##
titan 的功能大致都完成了，所以压测看看titan 的情况，未雨绸缪。压测并发为200，没有思考时间，连续压15分钟，也就是差不多 180000 个请求，然而根据 QA 反应，一分钟就挂了。

## 解决过程 ##
- 首先我的第一个想法是，挂了也正常，因为毕竟是使用默认配置，gremlin-server 的内存才 512M，gremlin-server-worker 才 1 个线程，gremlin-server-exec 才 8 个线程。使用默认配置无法通过测试，说明还有很大的提升空间，所以不用担心。
- 第二个想法是：果然新技术难以驾驭啊，网络上资料怎么少，google 都救不了我啊，优化还是有难度的。
- 第三个想法是：默认配置撑不起 200 并发，有点脆弱啊。

先看日志才是，server 端日志，gremlin-server.log，没有输出错误信息。这说明有两种情况：
1. 请求没有到达 gremlin-server 时已经出错，也就是错误发生在 server 端
2. gremlin-server 并非将所有的信息都输出到 gremlin-server.log 中

转念一想就知道第一种的可能非常大。所以果断拿出 tomcat 日志，果然所有错误如出一辙：
```XML
2016-06-28 18:12:54  95808720 [http-bio-8080-exec-53] ERROR com.nd.gaea.rest.exceptions.FriendlyWafRestErrorResolver:57   -
java.lang.NullPointerException
        at nd.esp.service.lifecycle.services.titan.TitanSearchServiceImpl.search(TitanSearchServiceImpl.java:99)
        at nd.esp.service.lifecycle.educommon.services.impl.NDResourceServiceImpl.resourceQueryByTitan(NDResourceServiceImpl.java:256)
        at sun.reflect.GeneratedMethodAccessor876.invoke(Unknown Source)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:317)
        at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:201)
        at com.sun.proxy.$Proxy146.resourceQueryByTitan(Unknown Source)
        at nd.esp.service.lifecycle.educommon.controllers.NDResourceController.requestQuering(NDResourceController.java:578)
        at nd.esp.service.lifecycle.educommon.controllers.NDResourceController.requestQueringByTitan(NDResourceController.java:346)
        at nd.esp.service.lifecycle.educommon.controllers.NDResourceController$$FastClassBySpringCGLIB$$a47413c5.invoke(<generated>)
        at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:204)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:711)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:157)
        at org.springframework.aop.framework.adapter.MethodBeforeAdviceInterceptor.invoke(MethodBeforeAdviceInterceptor.java:52)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
        at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:92)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
        at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:644)
        at nd.esp.service.lifecycle.educommon.controllers.NDResourceController$$EnhancerBySpringCGLIB$$e0c1ded6.requestQueringByTitan(<generated>)
        at sun.reflect.GeneratedMethodAccessor875.invoke(Unknown Source)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.springframework.web.method.support.InvocableHandlerMethod.invoke(InvocableHandlerMethod.java:215)
        at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:132)
        at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:104)
        at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandleMethod(RequestMappingHandlerAdapter.java:749)
        at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:689)
        at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:83)
        at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:938)
        at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:870)
        at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:961)
        at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:852)
        at javax.servlet.http.HttpServlet.service(HttpServlet.java:624)
        at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:837)
        at javax.servlet.http.HttpServlet.service(HttpServlet.java:731)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:303)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
        at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
        at org.springframework.security.web.FilterChainProxy.doFilterInternal(FilterChainProxy.java:186)
        at org.springframework.security.web.FilterChainProxy.doFilter(FilterChainProxy.java:160)
        at org.springframework.web.filter.DelegatingFilterProxy.invokeDelegate(DelegatingFilterProxy.java:344)
        at org.springframework.web.filter.DelegatingFilterProxy.doFilter(DelegatingFilterProxy.java:261)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
        at com.nd.gaea.rest.filter.WafHttpMethodOverrideFilter.doFilterInternal(WafHttpMethodOverrideFilter.java:91)
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:108)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)
at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:711)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:157)
        at org.springframework.aop.framework.adapter.MethodBeforeAdviceInterceptor.invoke(MethodBeforeAdviceInterceptor.java:52)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
        at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:92)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
        at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:644)
        at nd.esp.service.lifecycle.educommon.controllers.NDResourceController$$EnhancerBySpringCGLIB$$e0c1ded6.requestQueringByTitan(<generated>)
        at sun.reflect.GeneratedMethodAccessor875.invoke(Unknown Source)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.springframework.web.method.support.InvocableHandlerMethod.invoke(InvocableHandlerMethod.java:215)
        at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:132)
        at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:104)
        at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandleMethod(RequestMappingHandlerAdapter.java:749)
        at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:689)
        at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:83)
        at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:938)
        at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:870)
        at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:961)
        at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:852)
        at javax.servlet.http.HttpServlet.service(HttpServlet.java:624)
        at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:837)
        at javax.servlet.http.HttpServlet.service(HttpServlet.java:731)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:303)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
        at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
        at org.springframework.security.web.FilterChainProxy.doFilterInternal(FilterChainProxy.java:186)
        at org.springframework.security.web.FilterChainProxy.doFilter(FilterChainProxy.java:160)
        at org.springframework.web.filter.DelegatingFilterProxy.invokeDelegate(DelegatingFilterProxy.java:344)
        at org.springframework.web.filter.DelegatingFilterProxy.doFilter(DelegatingFilterProxy.java:261)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)params:{cg_taxonpath={eq=[K12/$ON030000/*/$E004001]}, lc_create_time={gt=[2016-01-14 00:00:00]}, title={like=[cms]}, coverages={in=[Debug/qa/*, Org/nd/*/ONLINE]}}
        at com.nd.gaea.rest.filter.WafHttpMethodOverrideFilter.doFilterInternal(WafHttpMethodOverrideFilter.java:91)
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:108)
at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
        at com.nd.gaea.rest.filter.WafCorsFilter.doFilterInternal(WafCorsFilter.java:58)null
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:108)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)gt
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
        at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:88)
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:108)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
        at com.nd.gaea.rest.filter.ExceptionFilter.doFilterInternal(ExceptionFilter.java:52)
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:108)
        at org.springframework.web.filter.DelegatingFilterProxy.invokeDelegate(DelegatingFilterProxy.java:344)
        at org.springframework.web.filter.DelegatingFilterProxy.doFilter(DelegatingFilterProxy.java:261)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
        at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:220)
        at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:122)
        at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:505)
        at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:169)
        at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:103)
        at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:116)
        at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:423){title0=.*cms.*, has_coverage1=has_coverage, target_type0=Debug, target_type1=Org, has_coverage0=has_coverage, coverage0=coverage, coverage1=coverage, target1=nd, assets0=assets, has_categories_path0=has_categories_path, target0=qa, lc_enable0=true, categories_path0=categories_path, lc_status0=ONLINE, cg_taxonpath0=K12/\$ON030000/.*/\$E004001, lc_create_time0=1452700800000}
        at org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1079)
        at org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:625)
        at org.apache.tomcat.util.net.JIoEndpoint$SocketProcessor.run(JIoEndpoint.java:316)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:745)
Caused by: java.lang.RuntimeException: java.util.concurrent.TimeoutException
        at org.apache.tinkerpop.gremlin.driver.Client.submitAsync(Client.java:183)
        at org.apache.tinkerpop.gremlin.driver.Client.submitAsync(Client.java:163)
        at org.apache.tinkerpop.gremlin.driver.Client.submit(Client.java:133)
        ... 103 more
```
`NullPointExecption` 这个错误太常见了，主要的代码并不是我写的，但是我有如下几个猜测：
1. 有共享变量或者全局变量的创建与销毁，导致并发出错
2. 由于 titan 请求是异步的，是否数据没有全部回到客户端已经被被使用
3. 数据有问题，在使用时报错了

定位到核心方法：
```JAVA
public ListViewModel<ResourceModel> search(String resType,
                                               List<String> includes,
                                               Map<String, Map<String, List<String>>> params,
                                               Map<String, String> orderMap, int from, int size, boolean reverse, String words) {
        System.out.println("params:" + params);
        TitanExpression titanExpression = new TitanExpression();

        Map<String, Object> scriptParamMap = new HashMap<String, Object>();

        dealWithOrderAndRange(titanExpression, orderMap, from, size);
        dealWithRelation(titanExpression, params.get("relation"), reverse);
        params.remove("relation");
        // for now only deal with code
        dealWithTaxoncode(titanExpression,
                params.get(ES_SearchField.cg_taxoncode.toString()));
        params.remove(ES_SearchField.cg_taxoncode.toString());

        dealWithTaxonpath(titanExpression,
                params.get(ES_SearchField.cg_taxonpath.toString()));
        params.remove(ES_SearchField.cg_taxonpath.toString());

        dealWithCoverage(titanExpression,
                params.get(ES_SearchField.coverages.toString()));
        params.remove(ES_SearchField.coverages.toString());

        TitanQueryVertexWithWords resourceQueryVertex = new TitanQueryVertexWithWords();
        resourceQueryVertex.setWords(words);
        resourceQueryVertex.setVertexLabel(resType);
        Map<String, Map<Titan_OP, List<Object>>> resourceVertexPropertyMap = new HashMap<String, Map<Titan_OP, List<Object>>>();
        resourceQueryVertex.setPropertiesMap(resourceVertexPropertyMap);
        resourceVertexPropertyMap
                .put(ES_SearchField.lc_enable.toString(),
                        generateFieldCondtion(
                                ES_SearchField.lc_enable.toString(), true));
        dealWithResource(resourceQueryVertex, params);
        titanExpression.addCondition(resourceQueryVertex);

        ListViewModel<ResourceModel> viewModels = new ListViewModel<ResourceModel>();
        List<ResourceModel> items = new ArrayList<ResourceModel>();

        //for count and result
        String scriptForResultAndCount = titanExpression.generateScriptForResultAndCount(scriptParamMap);
        System.out.println(scriptForResultAndCount);
        System.out.println(scriptParamMap);
        long beginTime = System.currentTimeMillis();
        ResultSet resultSet = titanResourceRepository.search(scriptForResultAndCount, scriptParamMap);
        Iterator<Result> iterator = resultSet.iterator();
        List<String> otherLines = new ArrayList<>();
        String taxOnPath = null;
        String mainResult = null;
        int count = 0;
        while (iterator.hasNext()) {
            String line = iterator.next().getString();
            System.out.println(line);
            if (count > 0 && (line.contains(ES_SearchField.lc_create_time.toString()) || line.startsWith(TitanKeyWords.TOTALCOUNT.toString()))) {
                items.add(getItem(resType, mainResult, otherLines, taxOnPath));
                otherLines.clear();
                taxOnPath = null;
            }

            if (line.startsWith(TitanKeyWords.TOTALCOUNT.toString())) {
                viewModels.setTotal(Long.parseLong(line.split(":")[1].trim()));
            } else if (line.contains(ES_SearchField.cg_taxonpath.toString())) {
                line = line.split("=")[1];
                int length = line.length();
                if (length > 2) {
                    taxOnPath = line.substring(1, length - 2);
                }
            } else if (line.contains(ES_SearchField.lc_create_time.toString())) {
                mainResult = line;
            } else {
                otherLines.add(line);
            }
            count++;
        }

        System.out.println("titan get values consume times: " + (System.currentTimeMillis() - beginTime));

        viewModels.setItems(items);
        return viewModels;

    }
```
由于压测是使用同一条语句不断发送请求，可以确定返回的数据格式是没有问题的。看代码会没有涉及到实例的创建与销毁，没有共享变量，主要使用局部变量，无法理解。代码是使用迭代器获得数据，虽然请求是异步的，但是在迭代器之后肯定是能够获得数据的。所以之前的猜想都不对，这就无法理解了。所以果断重现错误，先将应用 debug 模式启动，我本地通过 `JMeter` 压测，先让程序出现异常，之后单步跟踪。`gremlin-driver` 也就是客户端是通过socket 连接，可能是因为压力太大，客户端和服务器的连接超时了。所以执行语句返回的 `ResultSet` 为 `null`，虽然代码中有捕获异常，但是并没有调用 `throw` 抛出该异常，所以后面的代码继续执行，当通过 `ResultSet` 取数据时就会出现 `NullPointException`。
## 压测 ##
## 第一次 ##
所以整个过程就是因为 `gremlin` 连接超时导致的，我在想应该是有哪些配置没有注意到，所以通过 `google` 更换各种关键词，发现原来在连接的时候可以通过 `Cluster.Builder` 类设置线程池等参数。查看 `Cluster.Bulider` 源码，虽然注释并不是很多，但是一些基本的配置还是能够看懂的，之后是修改如下配置。
* 客户端
```JAVA
clusterBuilder.minConnectionPoolSize(200);
clusterBuilder.maxConnectionPoolSize(800);
clusterBuilder.nioPoolSize(50);
clusterBuilder.workerPoolSize(100);
```
* 服务端
修改 titan 的内存大小为16G：
```XML
gremlin-server.yaml
    threadPoolWorker: 48
        gremlinPool: 192
gremlin-server.sh
    if [ "$JAVA_OPTIONS" = "" ] ; then
    JAVA_OPTIONS="-Xmx16384m -Xms16384m -javaagent:$LIB/jamm-0.3.0.jar"
    fi
```
实际上说明中有指明，`workerPoolSize` 不应该超过内核线程数的两倍，但是这个小项目的 leader 觉得需要调大，那就只能够这样子设定了。我是觉得既然对方这样子设定肯定是有原因的，因为对方很了解自己做的东西，二则一开始不经过测试就盲目修改是没有意义的，也是没有标准的。
这样修改之后已经能够支撑 200 并发，15分钟的测试了，但是执行的速度并不快，我通过 `JVisualVM` 监控 `tomcat` 已经 `gremlin-server`，从线程数上看，实际上瓶颈在 `gremlin-server`，默认配置的性能比较低，没有充分利用 IO 和内存。
1. `gremlin-server` 线程情况：
![gremlin-server](http://i.imgur.com/DS6yLvX.jpg)
2. `tomcat` 也就是 `gremlin-driver` 的线程使用情况：
![gremlin-driver](http://i.imgur.com/wvDuG0Z.jpg)

很明显，服务端的进程都处于忙碌状态，而客户端则很多处于驻留状态，等待服务端数据返回。
QA 测试报告：
![QA 第一次测试报告](http://wiki.sdp.nd/images/a/ae/%E7%AC%AC%E4%B8%80%E6%AC%A1%E5%8E%8B%E6%B5%8B.jpg)

### 第二次压测 ###
* 客户端
```XML
clusterBuilder.minConnectionPoolSize(200);
clusterBuilder.maxConnectionPoolSize(800);
clusterBuilder.nioPoolSize(200);
clusterBuilder.workerPoolSize(1000);
``` 
* 服务端
```XML
gremlin-server.yaml
    threadPoolWorker: 100
        gremlinPool: 1000
gremlin-server.sh
    if [ "$JAVA_OPTIONS" = "" ] ; then
    JAVA_OPTIONS="-Xmx16384m -Xms16384m -javaagent:$LIB/jamm-0.3.0.jar"
    fi
```
实际上就是将服务端的线程池大小增大了。我是觉得没有必要的，因为官方文档已经说明了 `threadPoolWorker` 的值不应该大于cpu 核数的两倍，`gremlinPool` 大小有你的脚本的执行快慢决定，如果脚本能够在100ms 以内被解析完成则它的值应该为 `threadPoolWorker` 的 2 倍，否则则为 4 倍。
时间证明，加并不是线程数多效率就高。QA的测试报告：
![QA 第二次测试报告](http://wiki.sdp.nd/images/thumb/e/e7/%E7%AC%AC%E4%BA%8C%E6%AC%A1%E5%8E%8B%E6%B5%8B.jpg/600px-%E7%AC%AC%E4%BA%8C%E6%AC%A1%E5%8E%8B%E6%B5%8B.jpg)

### 第三次测试 ###
* 客户端
```XML
clusterBuilder.minConnectionPoolSize(200);
clusterBuilder.maxConnectionPoolSize(800);
clusterBuilder.nioPoolSize(200);
clusterBuilder.workerPoolSize(1000);
``` 
* 服务端
```XML
gremlin-server.yaml
    threadPoolWorker: 48
        gremlinPool: 96
gremlin-server.sh
    if [ "$JAVA_OPTIONS" = "" ] ; then
    JAVA_OPTIONS="-Xmx16384m -Xms16384m -javaagent:$LIB/jamm-0.3.0.jar"
    fi
```
这次修改是将线程池的数量更改为按照官方建议的做。QA 测试结果和***第一次测试***的结果是一样的。
![第三次测试报告](http://wiki.sdp.nd/images/4/41/%E7%AC%AC%E4%B8%89%E6%AC%A1%E5%8E%8B%E6%B5%8B.jpg)







