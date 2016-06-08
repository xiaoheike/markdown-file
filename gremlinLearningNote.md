## 注意 ##
部分语法在gremlin-server端可以运行，但是在gremlin-cosole端不能够运行。
## add vertex ##
```
Vertex marko = graph.addVertex(T.label, "person", T.id, 1, "name", "marko", "age", 29);
Vertex vadas = graph.addVertex(T.label, "person", T.id, 2, "name", "vadas", "age", 27);
gremlin = graph.addVertex(label,'software','name','gremlin')
```
各个属性以及属性与值之间是通过逗号分隔的。存储方式是无结构的，属性值可以有任意多个
T.label：gremlin中自定义的一个参数，标签，类似于mysql中的数据库
label：作用和T.label相同
T.id：gremlin中自定义的一个参数，id，每个边和顶点都必须有唯一id，并且边和顶点的id是共用的
返回值是新增节点
## add edge ##
```
marko.addEdge("knows", vadas, T.id, 7, "weight", 0.5f);
```
添加边marko-->vadas
## Vertex Properties ##


## Lambda Steps ##
- map(Function<Traverser<S>, E>) map the traverser to some object of type E for the next step to process.
- flatMap(Function<Traverser<S>, Iterator<E>>) map the traverser to an iterator of E objects that are streamed to the next step.
- filter(Predicate<Traverser<S>>) map the traverser to either true or false, where false will not pass the traverser to the next step.
- sideEffect(Consumer<Traverser<S>>) perform some operation on the traverser and pass it to the next step.
- branch(Function<Traverser<S>,M>) split the traverser to all the traversals indexed by the M token.

Traverser<S> 的值可能是如下几个：
1. 当前遍历到的对象S -- Traverser.get()
2. 当前被遍历的路径 -- Traverser.path()。获得历史路径对象的快捷方式 -- Traverser.path(String)==Traverser.path().get(String)
3. 遍历已经经历当前循环的次数 -- Traverser.loops()
4. 这个循环需要遍历的对象数量，例如顶点总数 -- Traverser.bulk()
5. 与当前遍历相关的本地数据结构 -- Traverser.sack()
6. 当前循环的副作用 -- Traverser.sideEffects()。获得指定循环的副作用快捷方式 -- Traverser.sideEffects(String) == Traverser.sideEffects().get(String)
----------
```
gremlin> g.V(1).out().values('name') //(1)
==>lop
==>vadas
==>josh
gremlin> g.V(1).out().map {it.get().value('name')} //(2)
==>lop
==>vadas
==>josh
```
- (1) 从顶点1到其他相邻节点并输出name属性
- (2) 与(1)是相同操作，不过使用lambda 语法实现
----------
```
gremlin> g.V().filter {it.get().label() == 'person'} //(1)
==>v[1]
==>v[2]
==>v[4]
==>v[6]
gremlin> g.V().hasLabel('person') //(2)
==>v[1]
==>v[2]
==>v[4]
==>v[6]
```
- (1) 一种过滤器，只允许顶点来传递，如果它有一个年龄属性
- (2) 与(1)相同，filter更多是实现比较，比如gt，lt等等。filter方法有需要其他的方法实现，比如hasLabel('person')
----------
```
gremlin> g.V().hasLabel('person').sideEffect(System.out.&println) //(1)
v[1]
==>v[1]
v[2]
==>v[2]
v[4]
==>v[4]
v[6]
==>v[6]    
```
- (1) 并不知道sideEffect() 作用是什么。
----------
```
gremlin> g.V().branch(values('name')).
               option('marko', values('age')).
               option(none, values('name')) //(1)
==>29
==>vadas
==>lop
==>josh
==>ripple
==>peter
gremlin> g.V().choose(has('name','marko'),
                      values('age'),
                      values('name')) //(2)
==>29
==>vadas
==>lop
==>josh
==>ripple
==>peter
```
- (1) 类似于java的switch语法，节点的name属性如果为'marko'则输出它的'age'值，其他的全部输出'name'值
- (2) 与(1)的功能相同
## AddEdge Step ##
![Alt text][http://tinkerpop.apache.org/docs/3.2.1-SNAPSHOT/images/addedge-step.png]
```
gremlin> g.V(1).as('a').out('created').in('created').where(neq('a')).
           addE('co-developer').from('a').property('year',2009) //(1)
==>e[12][1-co-developer->4]
==>e[13][1-co-developer->6]
gremlin> g.V(3,4,5).aggregate('x').has('name','josh').as('a').
           select('x').unfold().hasLabel('software').addE('createdBy').to('a') //(2)
==>e[14][3-createdBy->4]
==>e[15][5-createdBy->4]
gremlin> g.V().as('a').out('created').addE('createdBy').to('a').property('acl','public') //(3)
==>e[16][3-createdBy->1]
==>e[17][5-createdBy->4]
==>e[18][3-createdBy->4]
==>e[19][3-createdBy->6]
gremlin> g.V(1).as('a').out('knows').
           addE('livesNear').from('a').property('year',2009).
           inV().inE('livesNear').values('year') //(4)
==>2009
==>2009
gremlin> g.V().match(
                 __.as('a').out('knows').as('b'),
                 __.as('a').out('created').as('c'),
                 __.as('b').out('created').as('c')).
               addE('friendlyCollaborator').from('a').to('b').
                 property(id,13).property('project',select('c').values('name')) //(5)
==>e[13][1-friendlyCollaborator->4]
gremlin> g.E(13).valueMap()
==>[project:lop]
```
1. (1) 在marko和他的合作者之间创建 co-developer 关系
2. (2) 
## AddVertex Step ##
```
gremlin> g.addV('person').property('name','stephen') //(1)
==>v[12]
gremlin> g.V().values('name')
==>marko
==>vadas
==>lop
==>josh
==>ripple
==>peter
==>stephen
gremlin> g.V().outE('knows').addV().property('name','nothing')
==>v[14]
==>v[16]
gremlin> g.V().has('name','nothing')
==>v[16]
==>v[14]
gremlin> g.V().has('name','nothing').bothE()
```
- (1) 添加一个顶点：label：person，属性name：stephen

## AddProperty Step ##

```

```








