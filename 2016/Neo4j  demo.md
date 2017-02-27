## 需求 ##
新的需求下来，需要我使用一周的时间做出一个Neo4j的demo。需求的内容大概是这样子：我给我一个excel文件，文件中每行有10个标签（可以任务是一个名词，比如翠鸟，恐龙），这10个标签有一定的关联，比如都属于动物。excel文件中一共有10万行这样的数据。使用Neo4j数据库建立他们之间的关系。
### 需求分析 ###

1. 如果使用关系型数据库是不好表示各个标签之间的联系的，查询效率也低。
2. 行与行之间的标签有可能是相同的，那么创建节点就要保证唯一性
3. 相同节点之间有可能会创建相同的联系，需要保证关系的唯一性


## 了解Neo4j Cypher Query Language ##
通过谷歌找到了一个网站：[http://www.tutorialspoint.com/neo4j/index.htm](http://www.tutorialspoint.com/neo4j/index.htm "Neo4j tutorial")，上面有最简单的CQL语句使用。我在本地安装Neo4j server，创建一个Hello World数据库，跟着该网站的教程，了解了CQL的基本语法。CQL和SQL有一些小区别。


## 官网例子 ##
找到Neo4j 官网使用手册：[http://neo4j.com/docs/stable/preface.html](http://neo4j.com/docs/stable/preface.html)。跟着里边的教程，熟悉如何在工程中使用Embedded Neo4j。官网手册内容很丰富，我使用到的知识点里边都覆盖了，所以我要做的事情就是测试它给的代码，看输入输出是否符合需求。


## 唯一节点 ##
官方使用手册关于创建唯一节点有专门说明：[How to create unique nodes](http://neo4j.com/docs/stable/tutorials-java-embedded-unique-nodes.html)
该文章对Legacy index 和schema index创建唯一节点都列举出了方案。我最后采用"Merge to create a unique node"这个方案。
CQL语句：

	Merge(n:Tag{name = "A"}) return n 

相当于SQL语句：

	replace into Tag(name) values("A")

如果节点不存在则插入数据库，存在则不做操作。
采用创建唯一约束也是可以的，只是想到需要处理异常，觉得有些麻烦。


## 唯一联系 ##
使用手册中有提到相关内容：[Create unique relationships](http://neo4j.com/docs/stable/query-create-unique.html#_create_unique_relationships)

    MATCH (lft { name: 'A' }),(rgt)
    WHERE rgt.name IN ['B', 'C']
    CREATE UNIQUE (lft)-[r:KNOWS]->(rgt)
    RETURN r

如果不存在节点A到节点B，C的KNOWS联系，则创建联系，存在则不作操作。
因为需要在联系上添加weight属性，所以如果采用唯一索引的方式也不方便。所以之后采用方案：

- 检索该联系是否存在
- 如果存在则weight+1，不存在则创建联系设置weight=1

## 打包成jar ##
用三天的时间，完成了所有的需求，由于对方并没有给我数据源，所以我只能够模拟小数据的测试，简单过一下功能点。
使用的数据源：

	String[] tagArr1 = new String[] { "A", "B", "C", "D", "K", "M" };
    String[] tagArr3 = new String[] { "D", "O", "P", "Q" };
    String[] tagArr4 = new String[] { "Q", "1", "2", "3", "4" };
    String[] tagArr5 = new String[] { "1", "2", "5", "6" };

使用例子类：UndirectedGraphForManyTagArrExample,运行结果：

![Neo4j demo 运行结果](http://i.imgur.com/5LM0LAk.png)

通过neo4j server可以看到可视化数据情况：

![运行结果](http://i.imgur.com/hXiFhvb.png)

也就是将每行的字母当成一个标签，做成一个无向图。

源代码放置到github上：[https://github.com/xiaoheike/Neo4jUndirectedCompleteGraph](https://github.com/xiaoheike/Neo4jUndirectedCompleteGraph "Neo4j demo")


