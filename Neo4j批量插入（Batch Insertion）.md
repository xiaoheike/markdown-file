# Neo4j批量插入(Batch Insertion)
## 注意事项 
Neo4j针对初次数据导入提供了批量插入(Batch Insertion)方法，为了提高性能，批量插入会跳过事务以及其他检查。当你有一个很大的数据集需要被装载时，批量插入是很有用的。
批量插入包含在Neo4j的内核组件，它是所有的Neo4j分布和版本的一部分。
使用批量插入时需要注意一下几点：


1. 批量插入功能用于数据的初始导入，但是你可以将它使用于一个已经存在的数据库，前提是事先需要关闭该数据库(简单尝试后似乎并不能够，没有深入研究)
2. 批量插入不是线程安全的。
3. 批量插入是非事务。
4. 批量插入数据时，该操作不强制约束插入的数据（请查看本文“对第4点的实验”章节）
5. 批量插入**关闭(调用shutdown方法)**之后才会创建所有的索引以及约束
6. 如果批量插入不再最后被**关闭(调用shutdown方法)**，那么数据将会被破坏

以上内容翻译自：>[http://neo4j.com/docs/stable/batchinsert.html](http://neo4j.com/docs/stable/batchinsert.html)

## 对第4点的实验

**测试代码**

	public static void main(String[] args) {
        BatchInserter inserter = null;
        try
        {	
			String dir = "D:/neo4j/batchinserter-example";
			// 如果文件夹存在则删除
            File tempStoreDir = new File(dir).getAbsoluteFile();
            FileUtils.deleteRecursively(tempStoreDir);

            inserter = BatchInserters.inserter(new File(dir));

            Label personLabel = DynamicLabel.label("Person");
            // 标签Person下的name属性添加唯一约束
            inserter.createDeferredConstraint(personLabel).assertPropertyIsUnique("name").create();

            Map<String, Object> properties = new HashMap<>();

            properties.put("name", "Mattias");
            long mattiasNode = inserter.createNode(properties, personLabel);
			// 插入相同node(用于判断唯一约束是否起作用)
            inserter.createNode(properties, personLabel);

            properties.put("name", "Chris");
            long chrisNode = inserter.createNode(properties, personLabel);

            RelationshipType knows = DynamicRelationshipType.withName("KNOWS");
            Map<String, Object> map = new HashMap<String, Object>();
            map.put("weight", 1L);
            inserter.createRelationship(mattiasNode, chrisNode, knows, map);

        } catch (ConstraintViolationException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (inserter != null)
            {
                inserter.shutdown();
            }
        }
    }

**运行结果**

		Exception in thread "main" java.lang.RuntimeException: PreexistingIndexEntryConflictException{propertyValue=Mattias, addedNodeId=1, existingNodeId=0}
			at org.neo4j.unsafe.batchinsert.BatchInserterImpl.shutdown(BatchInserterImpl.java:1040)
			at nd.esp.com.MyNeo4j.BatchInsertDocTest.main(BatchInsertDocTest.java:126)
		Caused by: PreexistingIndexEntryConflictException{propertyValue=Mattias, addedNodeId=1, existingNodeId=0}
			at org.neo4j.kernel.api.impl.index.DeferredConstraintVerificationUniqueLuceneIndexPopulator$DuplicateCheckingCollector.doCollect(DeferredConstraintVerificationUniqueLuceneIndexPopulator.java:335)
			at org.neo4j.kernel.api.impl.index.DeferredConstraintVerificationUniqueLuceneIndexPopulator$DuplicateCheckingCollector.collect(DeferredConstraintVerificationUniqueLuceneIndexPopulator.java:296)
			at org.apache.lucene.search.TermScorer.score(TermScorer.java:75)
			at org.apache.lucene.search.TermScorer.score(TermScorer.java:67)
			at org.apache.lucene.search.IndexSearcher.search(IndexSearcher.java:581)
			at org.apache.lucene.search.IndexSearcher.search(IndexSearcher.java:383)
			at org.neo4j.kernel.api.impl.index.DeferredConstraintVerificationUniqueLuceneIndexPopulator.verifyDeferredConstraints(DeferredConstraintVerificationUniqueLuceneIndexPopulator.java:136)
			at org.neo4j.unsafe.batchinsert.BatchInserterImpl.repopulateAllIndexes(BatchInserterImpl.java:581)
			at org.neo4j.unsafe.batchinsert.BatchInserterImpl.shutdown(BatchInserterImpl.java:1036)
			... 1 more

关键点在：`PreexistingIndexEntryConflictException{propertyValue=Mattias, addedNodeId=1, existingNodeId=0}`，插入已经存在的node，而又设置了唯一约束，批量插入操作异常。

所以第四点的意思是指：如果设置了约束，比如唯一约束，那么需要保证批量插入数据的唯一性；如果数据本身没有唯一性，那么批量插入将无法正常运作。

教程结束，感谢阅读。

欢迎转载，但请注明本文链接，谢谢。

4/1/2016 5:07:33 PM 