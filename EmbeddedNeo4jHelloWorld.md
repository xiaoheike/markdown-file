# Eclipse下maven使用嵌入式（Embedded）Neo4j创建Hello World项目 #
新建一个maven工程，这里不赘述如何新建maven工程。
## 添加Neo4j jar到你的工程 ##

有两种方式：
- 上网站[官网下载jar包](http://neo4j.com/download/ "官网下载Neo4j jar包")，根据自己的系统下载不同的压缩包，详细过程不描述，请自行搜索其他博客
- 通过maven获得jar包，本文将详细介绍这个方法

## pom.xml文件下添加dependency ##
	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	  <modelVersion>4.0.0</modelVersion>
	
	  <groupId>nd.esp.com</groupId>
	  <artifactId>MyNeo4j</artifactId>
	  <version>0.0.1-SNAPSHOT</version>
	  <packaging>jar</packaging>
	
	  <name>MyNeo4j</name>
	  <url>http://maven.apache.org</url>

	  <properties>
	    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	  </properties>

	<!-- Embedded Neo4j依赖，目前最新版本是2.3.3-->
	  <dependencies>
	    <dependency>
	     <groupId>org.neo4j</groupId>
	     <artifactId>neo4j-slf4j</artifactId>
	     <version>2.3.3</version>
	    </dependency>
	  </dependencies>
	</project>

## Hello World 程序 ##

代码可以在github上看到：[https://github.com/neo4j/neo4j/blob/2.3.3/community/embedded-examples/src/main/java/org/neo4j/examples/EmbeddedNeo4j.java](https://github.com/neo4j/neo4j/blob/2.3.3/community/embedded-examples/src/main/java/org/neo4j/examples/EmbeddedNeo4j.java "Embedded Neo4j Hello World")


	/*
	 * Licensed to Neo Technology under one or more contributor
	 * license agreements. See the NOTICE file distributed with
	 * this work for additional information regarding copyright
	 * ownership. Neo Technology licenses this file to you under
	 * the Apache License, Version 2.0 (the "License"); you may
	 * not use this file except in compliance with the License.
	 * You may obtain a copy of the License at
	 *
	 * http://www.apache.org/licenses/LICENSE-2.0
	 *
	 * Unless required by applicable law or agreed to in writing,
	 * software distributed under the License is distributed on an
	 * "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
	 * KIND, either express or implied. See the License for the
	 * specific language governing permissions and limitations
	 * under the License.
	 */
	package org.neo4j.examples;
	
	import java.io.File;
	import java.io.IOException;
	
	import org.neo4j.graphdb.Direction;
	import org.neo4j.graphdb.GraphDatabaseService;
	import org.neo4j.graphdb.Node;
	import org.neo4j.graphdb.Relationship;
	import org.neo4j.graphdb.RelationshipType;
	import org.neo4j.graphdb.Transaction;
	import org.neo4j.graphdb.factory.GraphDatabaseFactory;
	import org.neo4j.io.fs.FileUtils;
	
	public class EmbeddedNeo4j
	{
		// Embedded Neo4j会在本地产生一个文件夹(类似于Mysql的数据库)
	    private static final String DB_PATH = "target/neo4j-hello-db";
	
	    public String greeting;
	
	    // START SNIPPET: vars
	    GraphDatabaseService graphDb;
	    Node firstNode;
	    Node secondNode;
	    Relationship relationship;
	    // END SNIPPET: vars
	
	    // START SNIPPET: createReltype
	    private static enum RelTypes implements RelationshipType
	    {
	        KNOWS
	    }
	    // END SNIPPET: createReltype
	
	    public static void main( final String[] args ) throws IOException
	    {
	        EmbeddedNeo4j hello = new EmbeddedNeo4j();
	        hello.createDb();
			// 删除数据
	        hello.removeData();
	        hello.shutDown();
	    }
	
	    void createDb() throws IOException
	    {
	        FileUtils.deleteRecursively( new File( DB_PATH ) );
	
	        // START SNIPPET: startDb
	        graphDb = new GraphDatabaseFactory().newEmbeddedDatabase( DB_PATH );
	        registerShutdownHook( graphDb );
	        // END SNIPPET: startDb
	
	        // START SNIPPET: transaction
			// Embedded Neo4j基本上所有的操作都需要在事务内执行
	        try ( Transaction tx = graphDb.beginTx() )
	        {
	            // Database operations go here
	            // END SNIPPET: transaction
	            // START SNIPPET: addData
	            firstNode = graphDb.createNode();
	            firstNode.setProperty( "message", "Hello, " );
	            secondNode = graphDb.createNode();
	            secondNode.setProperty( "message", "World!" );
	
	            relationship = firstNode.createRelationshipTo( secondNode, RelTypes.KNOWS );
	            relationship.setProperty( "message", "brave Neo4j " );
	            // END SNIPPET: addData
	
	            // START SNIPPET: readData
	            System.out.print( firstNode.getProperty( "message" ) );
	            System.out.print( relationship.getProperty( "message" ) );
	            System.out.print( secondNode.getProperty( "message" ) );
	            // END SNIPPET: readData
	
	            greeting = ( (String) firstNode.getProperty( "message" ) )
	                       + ( (String) relationship.getProperty( "message" ) )
	                       + ( (String) secondNode.getProperty( "message" ) );
	
	            // START SNIPPET: transaction
	            tx.success();
	        }
	        // END SNIPPET: transaction
	    }
		// 移除新建的数据
	    void removeData()
	    {
	        try ( Transaction tx = graphDb.beginTx() )
	        {
	            // START SNIPPET: removingData
	            // let's remove the data
	            firstNode.getSingleRelationship( RelTypes.KNOWS, Direction.OUTGOING ).delete();
	            firstNode.delete();
	            secondNode.delete();
	            // END SNIPPET: removingData
	
	            tx.success();
	        }
	    }
		// 关闭Neo4j 数据库
	    void shutDown()
	    {
	        System.out.println();
	        System.out.println( "Shutting down database ..." );
	        // START SNIPPET: shutdownServer
	        graphDb.shutdown();
	        // END SNIPPET: shutdownServer
	    }
	
	    // 为Neo4j 实例注册一个关闭的hook，当VM被强制退出时，Neo4j 实例能够正常关闭
	    private static void registerShutdownHook( final GraphDatabaseService graphDb )
	    {
	        // Registers a shutdown hook for the Neo4j instance so that it
	        // shuts down nicely when the VM exits (even if you "Ctrl-C" the
	        // running application).
	        Runtime.getRuntime().addShutdownHook( new Thread()
	        {
	            @Override
	            public void run()
	            {
	                graphDb.shutdown();
	            }
	        } );
	    }
	}

### 在事务中操作 ###
所有的操作都必须在一个事务中完成。这是官方一个刻意的设计，因为他们坚信事务划分是企业型数据库重要的一部分。所以，你可以使用如下的方式开启事务：

	try ( Transaction tx = graphDb.beginTx() ){
	    // Database operations go here
	    tx.success();
	}

try(){}这种语法是在jdk1.7之后支持的，这种方式能够让vm支持自动释放使用结束的资源。在这里可以不需要自己调用语句:

	finally{
		tx.close();
	}
	
如果你使用低于jdk1.7以后的版本，可以修改为如下的代码：

	Transaction tx = graphDb.beginTx()	
	try{
	    // Database operations go here
	    tx.success();
	}finally{
		tx.close();
	}


### 创建一个简单的图 ###

使用一下代码创建两个节点和一个关系：

	firstNode = graphDb.createNode();
	firstNode.setProperty( "message", "Hello, " );
	secondNode = graphDb.createNode();
	secondNode.setProperty( "message", "World!" );
	
	relationship = firstNode.createRelationshipTo( secondNode, RelTypes.KNOWS );
	relationship.setProperty( "message", "brave Neo4j " );

创建之后的图数据如下：
![HelloWorld输出结果](http://i.imgur.com/PqDKcix.png)


本文翻译自官网使用手册：[http://neo4j.com/docs/stable/tutorials-java-embedded-hello-world.html](http://neo4j.com/docs/stable/tutorials-java-embedded-hello-world.html "官网文章")

教程结束，感谢阅读。
欢迎转载，但请注明本文链接，谢谢。
2016/4/5 20:09:25 