## jackson的自动检测机制 ##
jackson允许使用任意的构造方法或工厂方法来构造实例
使用 @JsonAutoDetect（作用在类上）来开启/禁止自动检测
- fieldVisibility:字段的可见级别
- ANY:任何级别的字段都可以自动识别
- NONE:所有字段都不可以自动识别
- NON_PRIVATE:非private修饰的字段可以自动识别
- PROTECTED_AND_PUBLIC:被protected和public修饰的字段可以被自动识别
- PUBLIC_ONLY:只有被public修饰的字段才可以被自动识别
- DEFAULT:同PUBLIC_ONLY

jackson默认的字段属性发现规则如下：
**所有被public修饰的字段->所有被public修饰的getter->所有被public修饰的setter**

举例：
```java
public static class TestPOJO{  
    TestPOJO(){}  

    TestPOJO(String name){  
        this.name = name;  
    }  
    private String name;  

    @Override  
    public String toString() {  
        return "TestPOJO{" +  
              "name='" + name + '\'' +  
              '}';  
    }  
}  
```

这个类我们只有一个private的name属性，并且没有提供对应的get,set方法，如果按照默认的属性发现规则我们将***无法序列化和反序列化***name 字段(如果没有get,set方法，只有被public修饰的属性才会被发现)，你可以通过修改@JsonAutoDetect的 fieldVisibility来调整自动发现级别，为了使name被自动发现，我们需要将级别调整为ANY
```java
@JsonAutoDetect(fieldVisibility = JsonAutoDetect.Visibility.ANY)  
```

同理，除了fieldVisibility可以设置外，还可以设置getterVisibility、setterVisibility、isGetterVisibility、creatorVisibility级别，不再多讲

除了上面的方式，你还可以有一些其他方式可以配置methods,fields和creators(构造器和静态方法)的自动检测，例如：
你可以配置MapperFeature来启动/禁止一些特别类型(getters,setters,fields,creators)的自动检测
比如下面的MapperFeature配置：
- SORT_PROPERTIES_ALPHABETICALLY：按字母顺序排序属性

```java
ObjectMapper objectMapper = new ObjectMapper();  
objectMapper.configure(MapperFeature.SORT_PROPERTIES_ALPHABETICALLY,true);  
```

## 配置SerializationFeature ##
一些我们比较常用的SerializationFeature配置：
- SerializationFeature.WRAP_ROOT_VALUE：是否环绕根元素，默认false，如果为true，则默认以类名作为根元素，你也可以通过@JsonRootName来自定义根元素名称

```java
objectMapper.configure(SerializationFeature.WRAP_ROOT_VALUE,true);  
```

举例：
```java
@JsonRootName("myPojo")  
public static class TestPOJO{  
    private String name;  

    public String getName() {  
        return name;  
    }  

    public void setName(String name) {  
        this.name = name;  
    }  
}
```

该类在序列化成json后类似如下:`{"myPojo":{"name":"aaaa"}}`

- SerializationFeature.INDENT_OUTPUT：是否缩放排列输出，默认false，有些场合为了便于排版阅读则需要对输出做缩放排列

```java
objectMapper.configure(SerializationFeature.INDENT_OUTPUT,true);  
```

举例：
如果一个类中有a、b、c、d四个可检测到的属性，那么序列化后的json输出类似下面：
```json
{
  "a" : "aaa",
  "b" : "bbb",
  "c" : "ccc",
  "d" : "ddd"
}
```

- SerializationFeature.WRITE_DATES_AS_TIMESTAMPS：序列化日期时以timestamps输出，默认true

```java
objectMapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS,true);  
```

比如如果一个类中有private Date date;这种日期属性，序列化后为：`{"date" : 1413800730456}`，若不为true，则为 `{"date" : "2014-10-20T10:26:06.604+0000"}`

- SerializationFeature.WRITE_ENUMS_USING_TO_STRING：序列化枚举是以toString()来输出，默认false，即默认以name()来输出

```java
objectMapper.configure(SerializationFeature.WRITE_ENUMS_USING_TO_STRING,true);  
```

- SerializationFeature.WRITE_ENUMS_USING_INDEX：序列化枚举是以ordinal()来输出，默认false

```java
objectMapper.configure(SerializationFeature.WRITE_ENUMS_USING_INDEX,true);  
```

举例：
```java
@Test  
public void enumTest() throws Exception {  
    TestPOJO testPOJO = new TestPOJO();  
    testPOJO.setName("myName");  
    testPOJO.setMyEnum(TestEnum.ENUM01);  
    ObjectMapper objectMapper = new ObjectMapper();  
    objectMapper.configure(SerializationFeature.WRITE_ENUMS_USING_TO_STRING,false);  
    objectMapper.configure(SerializationFeature.WRITE_ENUMS_USING_INDEX,false);  
    String jsonStr1 = objectMapper.writeValueAsString(testPOJO);  
    Assert.assertEquals("{\"myEnum\":\"ENUM01\",\"name\":\"myName\"}",jsonStr1);  

    ObjectMapper objectMapper2 = new ObjectMapper();  
    objectMapper2.configure(SerializationFeature.WRITE_ENUMS_USING_TO_STRING,true);  
    String jsonStr2 = objectMapper2.writeValueAsString(testPOJO);  
    Assert.assertEquals("{\"myEnum\":\"enum_01\",\"name\":\"myName\"}",jsonStr2);  

    ObjectMapper objectMapper3 = new ObjectMapper();  
    objectMapper3.configure(SerializationFeature.WRITE_ENUMS_USING_INDEX,true);  
    String jsonStr3 = objectMapper3.writeValueAsString(testPOJO);  
    Assert.assertEquals("{\"myEnum\":0,\"name\":\"myName\"}",jsonStr3);  
}
public static class TestPOJO{  
    TestPOJO(){}  
    private TestEnum myEnum;  
    private String name;  

    //getters、setters省略  
}  
public static enum TestEnum{  
    ENUM01("enum_01"),ENUM02("enum_01"),ENUM03("enum_01");  

    private String title;  

    TestEnum(String title) {  
        this.title = title;  
    }  

    @Override  
    public String toString() {  
        return title;  
    }  
}
```

- SerializationFeature.WRITE_SINGLE_ELEM_ARRAYS_UNWRAPPED：序列化单元素数组时不以数组来输出，默认false

```java
objectMapper.configure(SerializationFeature.WRITE_ENUMS_USING_TO_STRING,true);  
```

举例：
```java
@Test  
public void singleElemArraysUnwrap() throws Exception {  
    TestPOJO testPOJO = new TestPOJO();  
    testPOJO.setName("myName");  
    List<Integer> counts = new ArrayList<>();  
    counts.add(1);  
    testPOJO.setCounts(counts);  
    ObjectMapper objectMapper = new ObjectMapper();  
    objectMapper.configure(SerializationFeature.WRITE_SINGLE_ELEM_ARRAYS_UNWRAPPED,false);  
    String jsonStr1 = objectMapper.writeValueAsString(testPOJO);  
    Assert.assertEquals("{\"name\":\"myName\",\"counts\":[1]}",jsonStr1);  

    ObjectMapper objectMapper2 = new ObjectMapper();  
    objectMapper2.configure(SerializationFeature.WRITE_SINGLE_ELEM_ARRAYS_UNWRAPPED,true);  
    String jsonStr2 = objectMapper2.writeValueAsString(testPOJO);  
    Assert.assertEquals("{\"name\":\"myName\",\"counts\":1}",jsonStr2);  
}  

public static class TestPOJO{  
    private String name;  
    private List<Integer> counts;  

    //getters、setters省略  
}
```

- SerializationFeature.ORDER_MAP_ENTRIES_BY_KEYS：序列化Map时对key进行排序操作，默认false

```java
objectMapper.configure(SerializationFeature.ORDER_MAP_ENTRIES_BY_KEYS,true);  
```

举例：
```java
@Test  
public void orderMapBykey() throws Exception {  
    TestPOJO testPOJO = new TestPOJO();  
    testPOJO.setName("myName");  
    Map<String,Integer> counts = new HashMap<>();  
    counts.put("a",1);  
    counts.put("d",4);  
    counts.put("c",3);  
    counts.put("b",2);  
    counts.put("e",5);  
    testPOJO.setCounts(counts);  
    ObjectMapper objectMapper = new ObjectMapper();  
    objectMapper.configure(SerializationFeature.ORDER_MAP_ENTRIES_BY_KEYS,false);  
    String jsonStr1 = objectMapper.writeValueAsString(testPOJO);  
    Assert.assertEquals("{\"name\":\"myName\",\"counts\":{\"d\":4,\"e\":5,\"b\":2,\"c\":3,\"a\":1}}",jsonStr1);  

    ObjectMapper objectMapper2 = new ObjectMapper();  
    objectMapper2.configure(SerializationFeature.ORDER_MAP_ENTRIES_BY_KEYS,true);  
    String jsonStr2 = objectMapper2.writeValueAsString(testPOJO);  
    Assert.assertEquals("{\"name\":\"myName\",\"counts\":{\"a\":1,\"b\":2,\"c\":3,\"d\":4,\"e\":5}}",jsonStr2);  
}  

public static class TestPOJO{  
    private String name;  
    private Map<String,Integer> counts;  

    //getters、setters省略  
}  
```

- SerializationFeature.WRITE_CHAR_ARRAYS_AS_JSON_ARRAYS：序列化char[]时以json数组输出，默认false

```java
objectMapper.configure(SerializationFeature.WRITE_CHAR_ARRAYS_AS_JSON_ARRAYS,true);  
```

举例：
```java
@Test  
public void charArraysAsJsonArrays() throws Exception {  
    TestPOJO testPOJO = new TestPOJO();  
    testPOJO.setName("myName");  
    char[] counts = new char[]{'a','b','c','d'};  
    testPOJO.setCounts(counts);  
    ObjectMapper objectMapper = new ObjectMapper();  
    objectMapper.configure(SerializationFeature.WRITE_CHAR_ARRAYS_AS_JSON_ARRAYS,false);  
    String jsonStr1 = objectMapper.writeValueAsString(testPOJO);  
    Assert.assertEquals("{\"name\":\"myName\",\"counts\":\"abcd\"}",jsonStr1);  

    ObjectMapper objectMapper2 = new ObjectMapper();  
    objectMapper2.configure(SerializationFeature.WRITE_CHAR_ARRAYS_AS_JSON_ARRAYS,true);  
    String jsonStr2 = objectMapper2.writeValueAsString(testPOJO);  
    Assert.assertEquals("{\"name\":\"myName\",\"counts\":[\"a\",\"b\",\"c\",\"d\"]}",jsonStr2);  
}  

public static class TestPOJO{  
    private String name;  
    private char[] counts;  

    //getters、setters省略  
}
```

- SerializationFeature.WRITE_BIGDECIMAL_AS_PLAIN：序列化BigDecimal时之间输出原始数字还是科学计数，默认false，即是否以toPlainString()科学计数方式来输出

```java
objectMapper.configure(SerializationFeature.WRITE_CHAR_ARRAYS_AS_JSON_ARRAYS,true);  
```

举例：
```java
@Test  
public void bigDecimalAsPlain() throws Exception {  
    TestPOJO testPOJO = new TestPOJO();  
    testPOJO.setName("myName");  
    testPOJO.setCount(new BigDecimal("1e20"));  

    ObjectMapper objectMapper = new ObjectMapper();  
    objectMapper.configure(SerializationFeature.WRITE_BIGDECIMAL_AS_PLAIN,false);  
    String jsonStr1 = objectMapper.writeValueAsString(testPOJO);  
    Assert.assertEquals("{\"name\":\"myName\",\"count\":1E+20}",jsonStr1);  

    ObjectMapper objectMapper2 = new ObjectMapper();  
    objectMapper2.configure(SerializationFeature.WRITE_BIGDECIMAL_AS_PLAIN,true);  
    String jsonStr2 = objectMapper2.writeValueAsString(testPOJO);  
    Assert.assertEquals("{\"name\":\"myName\",\"count\":100000000000000000000}",jsonStr2);  
}  
```

配置 [DeserializationFeature](http://fasterxml.github.io/jackson-databind/javadoc/2.0.2/com/fasterxml/jackson/databind/DeserializationFeature.html)

需要注意的是对于第二种通过配置SerializationConfig和DeserializationConfig方式只能启动/禁止自动检测，无法修改我们所需的可见级别
有时候对每个实例进行可见级别的注解可能会非常麻烦，这时候我们需要配置一个全局的可见级别，通过 objectMapper.setVisibilityChecker()来实现，默认的VisibilityChecker实现类为 VisibilityChecker.Std，这样可以满足实现复杂场景下的基础配置。

也有一些实用简单的可见级别配置，比如：

```java
ObjectMapper objectMapper = new ObjectMapper();  
objectMapper.setVisibility(PropertyAccessor.FIELD, JsonAutoDetect.Visibility.ANY) // auto-detect all member fields  
 .setVisibility(PropertyAccessor.GETTER, JsonAutoDetect.Visibility.NONE) // but only public getters  
 .setVisibility(PropertyAccessor.IS_GETTER, JsonAutoDetect.Visibility.NONE) // and none of "is-setters"  
;
```

你也可以通过下面方式来禁止所有的自动检测功能
```java
ObjectMapper objectMapper = new ObjectMapper();  
objectMapper.setVisibilityChecker(objectMapper.getVisibilityChecker().with(JsonAutoDetect.Visibility.NONE));  
```

## jackson的常用注解 ##
1、@JsonAutoDetect
看上面自动检测，不再重复

2、@JsonIgnore
作用在字段或方法上，用来完全忽略被注解的字段和方法对应的属性，即便这个字段或方法可以被自动检测到或者还有其他的注解

举例
```java
@Test  
public void jsonIgnoreTest() throws Exception {  
    TestPOJO testPOJO = new TestPOJO();  
    testPOJO.setId(111);  
    testPOJO.setName("myName");  
    testPOJO.setCount(22);  

    ObjectMapper objectMapper = new ObjectMapper();  
    String jsonStr = objectMapper.writeValueAsString(testPOJO);  
    Assert.assertEquals("{\"id\":111}",jsonStr);  

    String jsonStr2 = "{\"id\":111,\"name\":\"myName\",\"count\":22}";  
    TestPOJO testPOJO2 = objectMapper.readValue(jsonStr2, TestPOJO.class);  
    Assert.assertEquals(111,testPOJO2.getId());  
    Assert.assertNull(testPOJO2.getName());  
    Assert.assertEquals(0,testPOJO2.getCount());  
}  

public static class TestPOJO{  
    private int id;  
    @JsonIgnore  
    private String name;  
    private int count;  

    public int getId() {  
        return id;  
    }  

    public void setId(int id) {  
        this.id = id;  
    }  

    public String getName() {  
        return name;  
    }  

    public void setName(String name) {  
        this.name = name;  
    }  

    public int getCount() {  
        return count;  
    }  
    @JsonIgnore  
    public void setCount(int count) {  
        this.count = count;  
    }  
}
```

当@JsonIgnore不管注解在getters上还是setters上都会忽略对应的属性

3、@JsonProperty
作用在字段或方法上，用来对属性的序列化/反序列化，可以用来避免遗漏属性，同时提供对属性名称重命名，比如在很多场景下Java对象的属性是按照规范的驼峰书写，但是实际展示的却是类似C-style或C++/Microsolft style

举例
```java
@Test  
public void jsonPropertyTest() throws Exception {  
    TestPOJO testPOJO = new TestPOJO();  
    testPOJO.wahaha(111);  
    testPOJO.setFirstName("myName");  

    ObjectMapper objectMapper = new ObjectMapper();  
    String jsonStr = objectMapper.writeValueAsString(testPOJO);  
    Assert.assertEquals("{\"id\":111,\"first_name\":\"myName\"}",jsonStr);  

    String jsonStr2 = "{\"id\":111,\"first_name\":\"myName\"}";  
    TestPOJO testPOJO2 = objectMapper.readValue(jsonStr2, TestPOJO.class);  
    Assert.assertEquals(111, testPOJO2.wahaha());  
    Assert.assertEquals("myName", testPOJO2.getFirstName());  
}  

public static class TestPOJO{  
    @JsonProperty//注意这里必须得有该注解，因为没有提供对应的getId和setId函数，而是其他的getter和setter，防止遗漏该属性  
    private int id;  
    @JsonProperty("first_name")  
    private String firstName;  

    public int wahaha() {  
        return id;  
    }  

    public void wahaha(int id) {  
        this.id = id;  
    }  

    public String getFirstName() {  
        return firstName;  
    }  

    public void setFirstName(String firstName) {  
        this.firstName = firstName;  
    }  
}  
```

4、@JsonIgnoreProperties
作用在类上，用来说明有些属性在序列化/反序列化时需要忽略掉，可以将它看做是@JsonIgnore的批量操作，但它的功能比 @JsonIgnore要强，比如一个类是代理类，我们无法将将@JsonIgnore标记在属性或方法上，此时便可用 @JsonIgnoreProperties标注在类声明上，它还有一个重要的功能是作用在反序列化时解析字段时过滤一些未知的属性，否则通常情况下解析 到我们定义的类不认识的属性便会抛出异常。

可以注明是想要忽略的属性列表如@JsonIgnoreProperties({"name","age","title"})，

也可以注明过滤掉未知的属性如@JsonIgnoreProperties(ignoreUnknown=true)

举例：
```java
@Test(expected = UnrecognizedPropertyException.class)  
public void JsonIgnoreProperties() throws Exception {  
    TestPOJO testPOJO = new TestPOJO();  
    testPOJO.setId(111);  
    testPOJO.setName("myName");  
    testPOJO.setAge(22);  
    ObjectMapper objectMapper = new ObjectMapper();  
    String jsonStr = objectMapper.writeValueAsString(testPOJO);  
    Assert.assertEquals("{\"id\":111}",jsonStr);//name和age被忽略掉了  

    String jsonStr2 = "{\"id\":111,\"name\":\"myName\",\"age\":22,\"title\":\"myTitle\"}";  
    TestPOJO testPOJO2 = objectMapper.readValue(jsonStr2, TestPOJO.class);  
    Assert.assertEquals(111, testPOJO2.getId());  
    Assert.assertNull(testPOJO2.getName());  
    Assert.assertEquals(0,testPOJO2.getAge());  
    String jsonStr3 = "{\"id\":111,\"name\":\"myName\",\"count\":33}";//这里有个未知的count属性，反序列化会报错  
    objectMapper.readValue(jsonStr3, TestPOJO.class);  
}  

@JsonIgnoreProperties({"name","age","title"})  
public static class TestPOJO{  
    private int id;  
    private String name;  
    private int age;  

    //getters、setters省略  
}  
```
如果将上面的
```java
@JsonIgnoreProperties({"name","age","title"})  
```

更换为
```java
@JsonIgnoreProperties(ignoreUnknown=true)  
```
那么测试用例中在反序列化未知的count属性时便不会抛出异常了

5、@JsonUnwrapped
作用在属性字段或方法上，用来将子JSON对象的属性添加到封闭的JSON对象，说起来比较难懂，看个例子就很清楚了，不多解释

举例
```java
@Test  
public void jsonUnwrapped() throws Exception {  
    TestPOJO testPOJO = new TestPOJO();  
    testPOJO.setId(111);  
    TestName testName = new TestName();  
    testName.setFirstName("张");  
    testName.setSecondName("三");  
    testPOJO.setName(testName);  
    ObjectMapper objectMapper = new ObjectMapper();  
    String jsonStr = objectMapper.writeValueAsString(testPOJO);  
    //如果没有@JsonUnwrapped，序列化后将为{"id":111,"name":{"firstName":"张","secondName":"三"}}  
    //因为在name属性上加了@JsonUnwrapped，所以name的子属性firstName和secondName将不会包含在name中。  
    Assert.assertEquals("{\"id\":111,\"firstName\":\"张\",\"secondName\":\"三\"}",jsonStr);  
    String jsonStr2 = "{\"id\":111,\"firstName\":\"张\",\"secondName\":\"三\"}";  
    TestPOJO testPOJO2 = objectMapper.readValue(jsonStr2,TestPOJO.class);  
    Assert.assertEquals(111,testPOJO2.getId());  
    Assert.assertEquals("张",testPOJO2.getName().getFirstName());  
    Assert.assertEquals("三",testPOJO2.getName().getSecondName());  
}  

public static class TestPOJO{  
    private int id;  
    @JsonUnwrapped  
    private TestName name;  

    //getters、setters省略  
}                     
public static class TestName{  
    private String firstName;  
    private String secondName;  

    //getters、setters省略  
}  
```

在2.0+版本中@JsonUnwrapped添加了prefix和suffix属性，用来对字段添加前后缀，这在有关属性分组上比较有用，在上面的测试用例中，如果我们将TestPOJO的name属性上的@JsonUnwrapped添加前后缀配置，即
```java
@JsonUnwrapped(prefix = "name_",suffix = "_test")  
```

那么TestPOJO序列化后将为{"id":111,"name_firstName_test":"张","name_secondName_test":"三"}，反序列化时也要加上前后缀才会被解析为POJO
6、@JsonIdentityInfo
2.0+版本新注解，作用于类或属性上，被用来在序列化/反序列化时为该对象或字段添加一个对象识别码，通常是用来解决循环嵌套的问题，比如数据库 中的多对多关系，通过配置属性generator来确定识别码生成的方式，有简单的，配置属性property来确定识别码的名称，识别码名称没有限制。
对象识别码可以是虚拟的，即存在在JSON中，但不是POJO的一部分，这种情况下我们可以如此使用注解

```java
@JsonIdentityInfo(generator = ObjectIdGenerators.IntSequenceGenerator.class,property = "@id")  
```

对象识别码也可以是真实存在的，即以对象的属性为识别码，通常这种情况下我们一般以id属性为识别码，可以这么使用注解
```java
@JsonIdentityInfo(generator = ObjectIdGenerators.PropertyGenerator.class,property = "id")  
```

举例
```java
@Test  
public void jsonIdentityInfo() throws Exception {  
    Parent parent = new Parent();  
    parent.setName("jack");  
    Child child = new Child();  
    child.setName("mike");  
    Child[] children = new Child[]{child};  
    parent.setChildren(children);  
    child.setParent(parent);  
    ObjectMapper objectMapper = new ObjectMapper();  
    String jsonStr = objectMapper.writeValueAsString(parent);  
    Assert.assertEquals("{\"@id\":1,\"name\":\"jack\",\"children\":[{\"name\":\"mike\",\"parent\":1}]}",jsonStr);  
}  

@JsonIdentityInfo(generator = ObjectIdGenerators.IntSequenceGenerator.class,property = "@id")  
public static class Parent{  
    private String name;  
    private Child[] children;  

    //getters、setters省略  
}  

public static class Child{  
    private String name;  
    private Parent parent;  

    //getters、setters省略  
}  
```

这里需要提醒一下的是在1.6版本中提供了@JsonManagedReference和@JsonBackReference来解决循环嵌套问题，因为属于过时注解这里就不解释了，有兴趣的可以自己看

7、@JsonNaming
jackson 2.1+版本的注解，作用于类或方法，注意这个注解是在jackson-databind包中而不是在jackson-annotations包里，它可以让你定制属性命名策略，作用和前面提到的@JsonProperty的重命名属性名称相同。比如
你有一个JSON串{"in_reply_to_user_id":"abc123"}，需要反序列化为POJO，POJO一般情况下则需要如此写

```java
public static class TestPOJO{  

    private String in_reply_to_user_id;  

    public String getIn_reply_to_user_id() {  
        return in_reply_to_user_id;  
    }  

    public void setIn_reply_to_user_id(String in_reply_to_user_id) {  
        this.in_reply_to_user_id = in_reply_to_user_id;  
    }  
}
```

但这显然不符合JAVA的编码规范，你可以用@JsonProperty，比如：
```java
public static class TestPOJO{  

    @JsonProperty("in_reply_to_user_id")  
    private String inReplyToUserId;  

    public String getInReplyToUserId() {  
        return inReplyToUserId;  
    }  

    public void setInReplyToUserId(String inReplyToUserId) {  
        this.inReplyToUserId = inReplyToUserId;  
    }  
}
```

这样就符合规范了，可是如果POJO里有很多属性，给每个属性都要加上@JsonProperty是多么繁重的工作，这里就需要用到@JsonNaming了，它不仅能制定统一的命名规则，还能任意按自己想要的方式定制
举例

```java
@Test  
public void jsonNaming() throws Exception{  
    String jsonStr = "{\"in_reply_to_user_id\":\"abc123\"}";  
    ObjectMapper objectMapper = new ObjectMapper();  
    TestPOJO testPOJO = objectMapper.readValue(jsonStr,TestPOJO.class);  
    Assert.assertEquals("abc123",testPOJO.getInReplyToUserId());  

    TestPOJO testPOJO2 = new TestPOJO();  
    testPOJO2.setInReplyToUserId("abc123");  
    String jsonStr2 = objectMapper.writeValueAsString(testPOJO2);  
    Assert.assertEquals(jsonStr,jsonStr2);  
}  

@JsonNaming(PropertyNamingStrategy.LowerCaseWithUnderscoresStrategy.class)  
public static class TestPOJO{  

    private String inReplyToUserId;  

    public String getInReplyToUserId() {  
        return inReplyToUserId;  
    }  

    public void setInReplyToUserId(String inReplyToUserId) {  
        this.inReplyToUserId = inReplyToUserId;  
    }  
}  
```

@JsonNaming 使用了jackson已经实现的PropertyNamingStrategy.LowerCaseWithUnderscoresStrategy，它 可以将大写转换为小写并添加下划线。你可以自定义，必须继承类PropertyNamingStrategy，建议继承 PropertyNamingStrategyBase，我们自己实现一个类似LowerCaseWithUnderscoresStrategy的策 略，只是将下划线改为破折号
举例

```java
@Test  
public void jsonNaming() throws Exception{  
    String jsonStr = "{\"in-reply-to-user-id\":\"abc123\"}";  
    ObjectMapper objectMapper = new ObjectMapper();  
    TestPOJO testPOJO = objectMapper.readValue(jsonStr,TestPOJO.class);  
    Assert.assertEquals("abc123", testPOJO.getInReplyToUserId());  

    TestPOJO testPOJO2 = new TestPOJO();  
    testPOJO2.setInReplyToUserId("abc123");  
    String jsonStr2 = objectMapper.writeValueAsString(testPOJO2);  
    Assert.assertEquals(jsonStr, jsonStr2);  
}  

@JsonNaming(MyPropertyNamingStrategy.class)  
public static class TestPOJO{  

    private String inReplyToUserId;  

    public String getInReplyToUserId() {  
        return inReplyToUserId;  
    }  

    public void setInReplyToUserId(String inReplyToUserId) {  
        this.inReplyToUserId = inReplyToUserId;  
    }  
}  

public static class MyPropertyNamingStrategy
                                         extends PropertyNamingStrategy.PropertyNamingStrategyBase {  
    @Override  
    public String translate(String input) {  
        if (input == null) return input; // garbage in, garbage out  
        int length = input.length();  
        StringBuilder result = new StringBuilder(length * 2);  
        int resultLength = 0;  
        boolean wasPrevTranslated = false;  
        for (int i = 0; i < length; i++)  
        {  
            char c = input.charAt(i);  
            if (i > 0 || c != '-') // skip first starting underscore  
            {  
                if (Character.isUpperCase(c))  
                {  
                    if (!wasPrevTranslated && resultLength > 0 && result.charAt(resultLength - 1) != '-')  
                    {  
                        result.append('-');  
                        resultLength++;  
                    }  
                    c = Character.toLowerCase(c);  
                    wasPrevTranslated = true;  
                }  
                else  
                {  
                    wasPrevTranslated = false;  
                }  
                result.append(c);  
                resultLength++;  
            }  
        }  
        return resultLength > 0 ? result.toString() : input;  
    }  
}  
```

如果你想让自己定制的策略对所有解析都实现，除了对每个具体的实体类对应的位置加上@JsonNaming外你还可以如下做全局配置
```java
ObjectMapper objectMapper = new ObjectMapper();  
objectMapper.setPropertyNamingStrategy(new MyPropertyNamingStrategy());  
```

多态类型处理
jackson允许配置多态类型处理，当进行反序列话时，JSON数据匹配的对象可能有多个子类型，为了正确的读取对象的类型，我们需要添加一些类型信息。可以通过下面几个注解来实现：

@JsonTypeInfo
作用于类/接口，被用来开启多态类型处理，对基类/接口和子类/实现类都有效

```java
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME,include = JsonTypeInfo.As.PROPERTY,property = "name")  
```
这个注解有一些属性，
use:定义使用哪一种类型识别码，它有下面几个可选值：
1、JsonTypeInfo.Id.CLASS：使用完全限定类名做识别
2、JsonTypeInfo.Id.MINIMAL_CLASS：若基类和子类在同一包类，使用类名(忽略包名)作为识别码
3、JsonTypeInfo.Id.NAME：一个合乎逻辑的指定名称
4、JsonTypeInfo.Id.CUSTOM：自定义识别码，由@JsonTypeIdResolver对应，稍后解释
5、JsonTypeInfo.Id.NONE：不使用识别码

include(可选):指定识别码是如何被包含进去的，它有下面几个可选值：
1、JsonTypeInfo.As.PROPERTY：作为数据的兄弟属性
2、JsonTypeInfo.As.EXISTING_PROPERTY：作为POJO中已经存在的属性
3、JsonTypeInfo.As.EXTERNAL_PROPERTY：作为扩展属性
4、JsonTypeInfo.As.WRAPPER_OBJECT：作为一个包装的对象
5、JsonTypeInfo.As.WRAPPER_ARRAY：作为一个包装的数组

property(可选):制定识别码的属性名称
此属性只有当use为JsonTypeInfo.Id.CLASS（若不指定property则默认为@class）、 JsonTypeInfo.Id.MINIMAL_CLASS(若不指定property则默认为@c)、JsonTypeInfo.Id.NAME(若 不指定property默认为@type)，include为JsonTypeInfo.As.PROPERTY、 JsonTypeInfo.As.EXISTING_PROPERTY、JsonTypeInfo.As.EXTERNAL_PROPERTY时才有效

defaultImpl(可选)：如果类型识别码不存在或者无效，可以使用该属性来制定反序列化时使用的默认类型
visible(可选，默认为false)：是否可见

属性定义了类型标识符的值是否会通过JSON流成为反序列化器的一部分，默认为fale,也就是说,jackson会从JSON内容中处理和删除类型标识符再传递给JsonDeserializer。

@JsonSubTypes
作用于类/接口，用来列出给定类的子类，只有当子类类型无法被检测到时才会使用它
一般是配合@JsonTypeInfo在基类上使用，比如：

```java
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME,include = JsonTypeInfo.As.PROPERTY,property = "typeName")  
@JsonSubTypes({@JsonSubTypes.Type(value=Sub1.class,name = "sub1")
,@JsonSubTypes.Type(value=Sub2.class,name = "sub2")})  
```

@JsonSubTypes 的值是一个@JsonSubTypes.Type[]数组，里面枚举了多态类型(value对应类)和类型的标识符值(name对应 @JsonTypeInfo中的property标识名称的值，此为可选值，若不制定需由@JsonTypeName在子类上制定)
@JsonTypeName

作用于子类，用来为多态子类指定类型标识符的值

比如：

```java
@JsonTypeName(value = "sub1")  
```

value属性作用同上面@JsonSubTypes里的name作用
@JsonTypeResolver和@JsonTypeIdResoler

作用于类，可以自定义多态的类型标识符，这个平时很少用到，主要是现有的一般就已经满足绝大多数的需求了，如果你需要比较特别的类型标识符，建议使用这2个注解，自己定制基于TypeResolverBuilder和TypeIdResolver的类即可

我们看几个jackson处理多态的例子
```java
@Test  
public void jsonTypeInfo() throws Exception{  
    Sub1 sub1 = new Sub1();  
    sub1.setId(1);  
    sub1.setName("sub1Name");  
    Sub2 sub2 = new Sub2();  
    sub2.setId(2);  
    sub2.setAge(33);  
    ObjectMapper objectMapper = new ObjectMapper();  
    TestPOJO testPOJO = new TestPOJO();  
    testPOJO.setMyIns(new MyIn[]{sub1, sub2});  
    String jsonStr = objectMapper.writeValueAsString(testPOJO);  
    Assert.assertEquals("{\"myIns\":[{\"id\":1,\"name\":\"sub1Name\"},{\"id\":2,\"age\":33}]}", jsonStr);  
    System.out.println(jsonStr);  
}  

public static abstract class MyIn{  
    private int id;  

    //getters、setters省略  
}  

public static class Sub1 extends MyIn{  
    private String name;  

    //getters、setters省略  
}  

public static class Sub2 extends MyIn{  
    private int age;  

    //getters、setters省略  
}  
```
这是序列化时最简单的一种多态处理方式，因为没有使用任何多态处理注解，即默认使用的识别码类型为JsonTypeInfo.Id.NONE，而 jackson没有自动搜索功能，所以只能序列化而不能反序列化，上面序列化测试的结果为`{"myIns": [{"id":1,"name":"sub1Name"},{"id":2,"age":33}]}`，我们可以看到JSON串中是没有对应的多态类型识别码的。
下面我们在基类MyIn上加上多态处理相关注解，首先我们在基类MyIn上添加@JsonTypeInfo(use = JsonTypeInfo.Id.CLASS)即

```java
@JsonTypeInfo(use = JsonTypeInfo.Id.CLASS)  
public static abstract class MyIn{  
    private int id;  

    //getters、setters省略  
}  
```

执行上面的序列化测试代码结果将会是
`{"myIns":[{"@class":"cn.yangyong.fodder.util.JacksonUtilsTest$Sub1","id":1,"name":"sub1Name"},{"@class":"cn.yangyong.fodder.util.JacksonUtilsTest$Sub2","id":2,"age":33}]}`

我们可以看到多了相应的多态类型识别码，识别码名称为默认的@class（因为没有指定名称），识别码的值为JsonTypeInfo.Id.CLASS即子类完全限定名

我们再添加上property属性@JsonTypeInfo(use = JsonTypeInfo.Id.CLASS,property = "typeName")即
```java
@JsonTypeInfo(use = JsonTypeInfo.Id.CLASS,property = "typeName")  
public static abstract class MyIn{  
    private int id;  

    //getters、setters省略  
}
```

再次执行上面的序列化测试代码结果将会是
`{"myIns":[{"typeName":"cn.yangyong.fodder.util.JacksonUtilsTest$Sub1","id":1,"name":"sub1Name"},{"typeName":"cn.yangyong.fodder.util.JacksonUtilsTest$Sub2","id":2,"age":33}]}`

这次多态类型识别码的名称已经变成了我们指定的typeName而不是默认的@class了

上面的例子都是默认选择的include为JsonTypeInfo.As.PROPERTY，下面我们更改include方式，看看有什么变化，将include设置为JsonTypeInfo.As.WRAPPER_OBJECT即

```java
@JsonTypeInfo(use = JsonTypeInfo.Id.CLASS,include = JsonTypeInfo.As.WRAPPER_OBJECT,property = "typeName")  
public static abstract class MyIn{  
    private int id;  

    //getters、setters省略  
}
```

再次执行序列化测试，结果为
`{"myIns":[{"cn.yangyong.fodder.util.JacksonUtilsTest$Sub1":{"id":1,"name":"sub1Name"}},{"cn.yangyong.fodder.util.JacksonUtilsTest$Sub2":{"id":2,"age":33}}]}`

我们看到类型识别码不再成为兄弟属性包含进去了而是为父属性将其他属性包含进去，此时我们指定的property=“typeName”已经无用了

再次修改use属性指定为JsonTypeInfo.Id.MINIMAL_CLASS，即@JsonTypeInfo(use = JsonTypeInfo.Id.MINIMAL_CLASS,include = JsonTypeInfo.As.PROPERTY,property = "typeName")

```java
@JsonTypeInfo(use = JsonTypeInfo.Id.MINIMAL_CLASS,include = JsonTypeInfo.As.PROPERTY,property = "typeName")  
public static abstract class MyIn{  
    private int id;  

    //getters、setters省略  
}
```

测试序列化结果为
`{"myIns":[{"typeName":".JacksonUtilsTest$Sub1","id":1,"name":"sub1Name"},{"typeName":".JacksonUtilsTest$Sub2","id":2,"age":33}]}`

发现已经没有同包的package名称，识别码的值更加简短了

测试反序列化

```java
@Test  
public void jsonTypeInfo() throws Exception{  
    ObjectMapper objectMapper = new ObjectMapper();  
    String jsonStr2 = "{\"myIns\":[{\"typeName\":\".JacksonUtilsTest$Sub1\",\"id\":1,\"name\":\"sub1Name\"},{\"typeName\":\".JacksonUtilsTest$Sub2\",\"id\":2,\"age\":33}]}";  
    TestPOJO testPOJO2 = objectMapper.readValue(jsonStr2,TestPOJO.class);  
    MyIn[] myIns = testPOJO2.getMyIns();  
    for (MyIn myIn : myIns) {  
        System.out.println(myIn.getClass().getSimpleName());  
    }  
}
```

结果将会显示为Sub1和Sub2说明是可以实现多态的反序列化的
可能我们在反序列化时觉得如此传递识别码很不友好，最好可以自定义识别码的值，可以选择use = JsonTypeInfo.Id.NAME和@JsonSubTypes配合即

```java
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME,include = JsonTypeInfo.As.PROPERTY,property = "typeName")  
@JsonSubTypes({@JsonSubTypes.Type(value=Sub1.class,name="sub1")
,@JsonSubTypes.Type(value=Sub2.class,name="sub2")})  
public static abstract class MyIn{  
    private int id;  
    //getters、setters省略  
}
```

执行序列化结果为
`{"myIns":[{"typeName":"sub1","id":1,"name":"sub1Name"},{"typeName":"sub2","id":2,"age":33}]}`

使用这个结果反序列化也可以得到我们想要的结果，或者在子类上添加@JsonTypeName(value = "sub1")和@JsonTypeName(value = "sub2")以便取代@JsonSubTypes里的name

如果想不使用@JsonSubTypes来实现反序列化，我们可以在ObjectMapper上注册子类实现，即

```java
ObjectMapper objectMapper = new ObjectMapper();  
objectMapper.registerSubtypes(new NamedType(Sub1.class,"sub1"));  
objectMapper.registerSubtypes(new NamedType(Sub2.class,"sub2"));  
```

## 用于序列化和反序列化的注解类 ##


1、@JsonSerialize和@JsonDeserialize

作用于方法和字段上，通过 using(JsonSerializer)和using(JsonDeserializer)来指定序列化和反序列化的实现，通常我们在需要自定义序列化和反序列化时会用到，比如下面的例子中的日期转换

```java
@Test  
public void jsonSerializeAndDeSerialize() throws Exception {  
    TestPOJO testPOJO = new TestPOJO();  
    testPOJO.setName("myName");  
    testPOJO.setBirthday(new Date());  
    ObjectMapper objectMapper = new ObjectMapper();  
    String jsonStr = objectMapper.writeValueAsString(testPOJO);  
    System.out.println(jsonStr);  

    String jsonStr2 = "{\"name\":\"myName\",\"birthday\":\"2014-11-11 19:01:58\"}";  
    TestPOJO testPOJO2 = objectMapper.readValue(jsonStr2,TestPOJO.class);  
    System.out.println(testPOJO2.toString());  
}  

public static class TestPOJO{  
    private String name;  
    @JsonSerialize(using = MyDateSerializer.class)  
    @JsonDeserialize(using = MyDateDeserializer.class)  
    private Date birthday;  

    //getters、setters省略  

    @Override  
    public String toString() {  
        return "TestPOJO{" +  
            "name='" + name + '\'' +  
            ", birthday=" + birthday +  
            '}';  
    }  
}  

private static class MyDateSerializer extends JsonSerializer<Date>{  
    @Override  
    public void serialize(Date value, JsonGenerator jgen, SerializerProvider provider)
                                                                throws IOException, JsonProcessingException {  
        DateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");  
        String dateStr = dateFormat.format(value);  
        jgen.writeString(dateStr);  
    }  
}  

private static class MyDateDeserializer extends JsonDeserializer<Date>{  
    @Override  
    public Date deserialize(JsonParser jp, DeserializationContext ctxt)
throws IOException, JsonProcessingException {  
        String value = jp.getValueAsString();  
        DateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");  
        try {  
            return dateFormat.parse(value);  
        } catch (ParseException e) {  
            e.printStackTrace();  
        }  
        return null;  
    }  
}
```

上面的例子中自定义了日期的序列化和反序列化方式，可以将Date和指定日期格式字符串之间相互转换。

也可以通过使用as(JsonSerializer)和as(JsonDeserializer)来实现多态类型转换，上面我们有提到多态类型处理时可以 使用@JsonTypeInfo实现，还有一种比较简便的方式就是使用@JsonSerialize和@JsonDeserialize指定as的子类类 型，注意这里必须指定为子类类型才可以实现替换运行时的类型

```java
@Test  
public void jsonSerializeAndDeSerialize() throws Exception {  
    TestPOJO testPOJO = new TestPOJO();  
    testPOJO.setName("myName");  
    Sub1 sub1 = new Sub1();  
    sub1.setId(1);  
    sub1.setName("sub1Name");  
    Sub2 sub2 = new Sub2();  
    sub2.setId(2);  
    sub2.setAge(22);  
    testPOJO.setSub1(sub1);  
    testPOJO.setSub2(sub2);  
    ObjectMapper objectMapper = new ObjectMapper();  
    String jsonStr = objectMapper.writeValueAsString(testPOJO);  
    System.out.println(jsonStr);  

    String jsonStr2 = "{\"name\":\"myName\",\"sub1\":{\"id\":1,\"name\":\"sub1Name\"},\"sub2\":{\"id\":2,\"age\":22}}";  
    TestPOJO testPOJO2 = objectMapper.readValue(jsonStr2,TestPOJO.class);  
    System.out.println(testPOJO2.toString());  
}  

public static class TestPOJO{  
    private String name;  
    @JsonSerialize(as = Sub1.class)  
    @JsonDeserialize(as = Sub1.class)  
    private MyIn sub1;  
    @JsonSerialize(as = Sub2.class)  
    @JsonDeserialize(as = Sub2.class)  
    private MyIn sub2;  

    //getters、setters省略  

    @Override  
    public String toString() {  
        return "TestPOJO{" +  
            "name='" + name + '\'' +  
            ", sub1=" + sub1 +  
            ", sub2=" + sub2 +  
            '}';  
    }  
}  

public static class MyIn{  
    private int id;  

    //getters、setters省略  
}  

public static class Sub1 extends MyIn{  
    private String name;  

    //getters、setters省略  

    @Override  
    public String toString() {  
        return "Sub1{" +  
            "id=" + getId()  +  
            "name='" + name + '\'' +  
            '}';  
    }  
}  
public static class Sub2 extends MyIn{  
    private int age;  
    //getters、setters省略  

    @Override  
    public String toString() {  
        return "Sub1{" +  
            "id=" + getId() +  
            "age='" + age +  
            '}';  
    }  
}
```

上面例子中通过as来指定了需要替换实际运行时类型的子类，实际上上面例子中序列化时是可以不使用@JsonSerialize(as = Sub1.class)的，因为jackson可以自动将POJO转换为对应的JSON，而反序列化时由于无法自动检索匹配类型必须要指定 @JsonDeserialize(as = Sub1.class)方可实现
最后@JsonSerialize可以配置include属性来指定序列化时被注解的属性被包含的方式，默认总是被包含进来，但是可以过滤掉空的属性或有默认值的属性，举个简单的过滤空属性的例子如下

```java
@Test  
public void jsonSerializeAndDeSerialize() throws Exception {  
    TestPOJO testPOJO = new TestPOJO();  
    testPOJO.setName("");  
    ObjectMapper objectMapper = new ObjectMapper();  
    String jsonStr = objectMapper.writeValueAsString(testPOJO);  
    Assert.assertEquals("{}",jsonStr);  
}  

public static class TestPOJO{  
    @JsonSerialize(include = JsonSerialize.Inclusion.NON_EMPTY)  
    private String name;  

    //getters、setters省略  
}  
```

2、@JsonPropertyOrder
作用在类上，被用来指明当序列化时需要对属性做排序，它有2个属性
一个是alphabetic：布尔类型，表示是否采用字母拼音顺序排序，默认是为false，即不排序

```java
@Test  
public void jsonPropertyOrder() throws Exception {  
    TestPOJO testPOJO = new TestPOJO();  
    testPOJO.setA("1");  
    testPOJO.setB("2");  
    testPOJO.setC("3");  
    testPOJO.setD("4");  
    ObjectMapper objectMapper = new ObjectMapper();  
    String jsonStr = objectMapper.writeValueAsString(testPOJO);  
    Assert.assertEquals("{\"a\":\"1\",\"c\":\"3\",\"d\":\"4\",\"b\":\"2\"}",jsonStr);  
}  

public static class TestPOJO{  
    private String a;  
    private String c;  
    private String d;  
    private String b;  

    //getters、setters省略  
}
```

我们先看一个默认的排序方式，序列化单元测试结果依次为{"a":"1","c":"3","d":"4","b":"2"}，即是没有经过排序操作的，在 TestPOJO上加上@jsonPropertyOrder(alphabetic = true)再执行测试结果将会为{"a":"1","b":"2","c":"3","d":"4"}
还有一个属性是value：数组类型，表示将优先其他属性排序的属性名称

```java
@Test  
public void jsonPropertyOrder() throws Exception {  
    TestPOJO testPOJO = new TestPOJO();  
    testPOJO.setA("1");  
    testPOJO.setB("2");  
    testPOJO.setC("3");  
    testPOJO.setD("4");  
    ObjectMapper objectMapper = new ObjectMapper();  
    String jsonStr = objectMapper.writeValueAsString(testPOJO);  
    System.out.println(jsonStr);  
    Assert.assertEquals("{\"c\":\"3\",\"b\":\"2\",\"a\":\"1\",\"d\":\"4\"}",jsonStr);  
}  

@JsonPropertyOrder(alphabetic = true,value = {"c","b"})  
public static class TestPOJO{  
    private String a;  
    private String c;  
    private String d;  
    private String b;  

    //getters、setters省略  
}
```

上面例子可以看到value指定了c和b属性优先排序，所以序列化后为{"c":"3","b":"2","a":"1","d":"4"}
还记得本文上面最开始配置MapperFeature时也有属性排序么，对，就是

```java
objectMapper.configure(MapperFeature.SORT_PROPERTIES_ALPHABETICALLY,true);  
```

只不过@JsonPropertyOrder颗粒度要更细一点，可以决定哪些属性优先排序

3、@JsonView
视图模板，作用于方法和属性上，用来指定哪些属性可以被包含在JSON视图中，在前面我们知道已经有@JsonIgnore和 @JsonIgnoreProperties可以排除过滤掉不需要序列化的属性，可是如果一个POJO中有上百个属性，比如订单类、商品详情类这种属性超 多，而我们可能只需要概要简单信息即序列化时只想输出其中几个或10几个属性，此时使用@JsonIgnore和 @JsonIgnoreProperties就显得非常繁琐，而使用@JsonView便会非常方便，只许在你想要输出的属性(或对应的getter)上 添加@JsonView即可，举例：

```java
@Test  
public void jsonView() throws Exception {  
    TestPOJO testPOJO = new TestPOJO();  
    testPOJO.setA("1");  
    testPOJO.setB("2");  
    testPOJO.setC("3");  
    testPOJO.setD("4");  
    ObjectMapper objectMapper = new ObjectMapper();  
    objectMapper.configure(MapperFeature.DEFAULT_VIEW_INCLUSION, false);  
    String jsonStr = objectMapper.writerWithView(FilterView.OutputA.class).writeValueAsString(testPOJO);  
    Assert.assertEquals("{\"a\":\"1\",\"c\":\"3\"}",jsonStr);  
    String jsonStr2 = objectMapper.writerWithView(FilterView.OutputB.class).writeValueAsString(testPOJO);  
    Assert.assertEquals("{\"d\":\"4\",\"b\":\"2\"}",jsonStr2);  
}  

public static class TestPOJO{  
    @JsonView(FilterView.OutputA.class)  
    private String a;  
    @JsonView(FilterView.OutputA.class)  
    private String c;  
    @JsonView(FilterView.OutputB.class)  
    private String d;  
    @JsonView(FilterView.OutputB.class)  
    private String b;  
    //getters、setters忽略  
}  


private static class FilterView {  
    static class OutputA {}  
    static class OutputB {}  
}
```

上面的测试用例中，我们在序列化之前先设置了 objectMapper.configure(MapperFeature.DEFAULT_VIEW_INCLUSION, false)，看javadoc说这是一个双向开关，开启将输出没有JsonView注解的属性，false关闭将输出有JsonView注解的属性，可惜我在测试中开启开关后有JsonView注解的属性任然输出了，大家可以研究下。序列化时使用了 objectMapper.writerWithView(FilterView.OutputA.class).writeValueAsString(testPOJO)， 即使用哪个视图来输出。在上面的例子中又2种视图，我们在序列化的时候可以选择想要的视图来输出，这在一些地方比较好用，比如安卓、苹果、桌面等不同的客户端可能会输出不同的属性。在1.6版本中这个@JsonView注解同时也会强制性自动发现，也就是说不管属性的可见性以及是否设置了自动发现这些属性 都将会自动被发现，在上例中TestPOJO中的getters、setters可以不需要也能输出我们想要的结果。

4、@JsonFilter

Json属性过滤器，作用于类，作用同上面的@JsonView，都是过滤掉不想要的属性，输出自己想要的属性。和@FilterView不同的是 @JsonFilter可以动态的过滤属性，比如我不想输出以system开头的所有属性等待，应该说@JsonFilter更高级一点，举个简单的例子

```java
@Test  
public void jsonFilter() throws Exception {  
    TestPOJO testPOJO = new TestPOJO();  
    testPOJO.setA("1");  
    testPOJO.setB("2");  
    testPOJO.setC("3");  
    testPOJO.setD("4");  
    ObjectMapper objectMapper = new ObjectMapper();  
    FilterProvider filters = new SimpleFilterProvider()
                                         .addFilter("myFilter",SimpleBeanPropertyFilter.filterOutAllExcept("a"));  
    objectMapper.setFilters(filters);  
    String jsonStr = objectMapper.writeValueAsString(testPOJO);  
    Assert.assertEquals("{\"a\":\"1\"}",jsonStr);  
}  

@JsonFilter("myFilter")  
public static class TestPOJO{  
    private String a;  
    private String c;  
    private String d;  
    private String b;  

    //getters、setters省略  
}
```

上面例子中在我们想要序列化的POJO上加上了@JsonFilter，表示该类将使用名为myFilter的过滤器。在测试中定义了一个名为 myFilter的SimpleFilterProvider，这个过滤器将会过滤掉所有除a属性以外的属性。这只是最简单的输出指定元素的例子，你可以 自己实现FilterProvider来满足你的过滤需求。
有时候我们可能需要根据现有的POJO来过滤属性，而这种情况下通常不会让你修改已有的代码在POJO上加注解，这种情况下我们就可以结合@JsonFilter和MixInAnnotations来实现过滤属性，如下例所示，不再多做解释

```java
@Test  
public void jsonFilter() throws Exception {  
    TestPOJO testPOJO = new TestPOJO();  
    testPOJO.setA("1");  
    testPOJO.setB("2");  
    testPOJO.setC("3");  
    testPOJO.setD("4");  
    ObjectMapper objectMapper = new ObjectMapper();  
    FilterProvider filters = new SimpleFilterProvider()
                                              .addFilter("myFilter",SimpleBeanPropertyFilter.filterOutAllExcept("a"));  
    objectMapper.setFilters(filters);  
    objectMapper.addMixInAnnotations(TestPOJO.class,MyFilterMixIn.class);  
    String jsonStr = objectMapper.writeValueAsString(testPOJO);  
    Assert.assertEquals("{\"a\":\"1\"}",jsonStr);  
}  

public static class TestPOJO{  
    private String a;  
    private String c;  
    private String d;  
    private String b;  
    //getters、setters省略  
}  

@JsonFilter("myFilter")  
private static interface MyFilterMixIn{  
}  
```

5、@JsonIgnoreType
作用于类，表示被注解该类型的属性将不会被序列化和反序列化，也跟上面几个一样属于过滤属性功能的注解，举例：
```java
@Test  
public void jsonFilter() throws Exception {  
    TestPOJO testPOJO = new TestPOJO();  
    testPOJO.setName("myName");  
    Sub1 sub1 = new Sub1();  
    sub1.setId(1);  
    sub1.setName("sub1");  
    Sub2 sub2 = new Sub2();  
    sub2.setId(2);  
    sub2.setAge(22);  
    testPOJO.setMyIn(sub1);  
    testPOJO.setSub1(sub1);  
    testPOJO.setSub2(sub2);  
    ObjectMapper objectMapper = new ObjectMapper();  
    String jsonStr = objectMapper.writeValueAsString(testPOJO);  
    System.out.println(jsonStr);  
}  

public static class TestPOJO{  
    private Sub1 sub1;  
    private Sub2 sub2;  
    private MyIn myIn;  
    private String name;  
    //getters、setters省略  
}  

public static class MyIn{  
    private int id;  
    //getters、setters省略  
}  

@JsonIgnoreType  
public static class Sub1 extends MyIn{  
    private String name;  
    //getters、setters省略  
}  

@JsonIgnoreType  
public static class Sub2 extends MyIn{  
    private int age;  
    //getters、setters省略  
}
```

上面例子中我们在类Sub1和Sub2上都加上了@JsonIgnoreType，那么需要序列化和反序列时POJO中所有Sub1和Sub2类型的属性都 将会被忽略，上面测试结果为{"myIn":{"id":1,"name":"sub1"},"name":"myName"}，只输出了name和 myIn属性。需要注意的是@JsonIgnoreType是可以继承的，即如果在基类上添加了该注解，那么子类也相当于加了该注解。在上例中，如果只在 基类MyIn上添加@JsonIgnoreType那么序列化TestPOJO时将会过滤掉MyIn、Sub1、Sub2。输出结果为 {"name":"myName"}

6、@JsonAnySetter
作用于方法，在反序列化时用来处理遇到未知的属性的时候调用，在本文前面我们知道可以通过注解 @JsonIgnoreProperties(ignoreUnknown=true)来过滤未知的属性，但是如果需要这些未知的属性该如何是好?那么 @JsonAnySetter就可以派上用场了，它通常会和map属性配合使用用来保存未知的属性，举例：

```java
@Test  
public void jsonAnySetter() throws Exception {  
    ObjectMapper objectMapper = new ObjectMapper();  
    String jsonStr = "{\"name\":\"myName\",\"code\":\"12345\",\"age\":12}";  
    TestPOJO testPOJO = objectMapper.readValue(jsonStr,TestPOJO.class);  
    Assert.assertEquals("myName",testPOJO.getName());  
    Assert.assertEquals("12345",testPOJO.getOther().get("code"));  
    Assert.assertEquals(12,testPOJO.getOther().get("age"));  
}  

public static class TestPOJO{  
    private String name;  

    private Map other = new HashMap();  

    @JsonAnySetter  
    public void set(String name,Object value) {  
        other.put(name,value);  
    }  

    //getters、setters省略  
}
```

测试用例中我们在set方法上标注了@JsonAnySetter，每当遇到未知的属性时都会调用该方法

7、@JsonCreator

作用于方法，通常用来标注构造方法或静态工厂方法上，使用该方法来构建实例，默认的是使用无参的构造方法，通常是和@JsonProperty或@JacksonInject配合使用，举例

```java
@Test  
public void jsonCreator() throws Exception {  
    ObjectMapper objectMapper = new ObjectMapper();  
    String jsonStr = "{\"full_name\":\"myName\",\"age\":12}";  
    TestPOJO testPOJO = objectMapper.readValue(jsonStr,TestPOJO.class);  
    Assert.assertEquals("myName",testPOJO.getName());  
    Assert.assertEquals(12, testPOJO.getAge());  
}  

public static class TestPOJO{  
    private String name;  
    private int age;  

    @JsonCreator  
    public TestPOJO(@JsonProperty("full_name") String name,@JsonProperty("age") int age){  
        this.name = name;  
        this.age = age;  
    }  
    public String getName() {  
        return name;  
    }  
    public int getAge() {  
        return age;  
    }  
}
```
上面示例中是在构造方法上标注了@JsonCreator，同样你也可以标注在静态工厂方法上，比如：
```java
@Test  
public void jsonCreator() throws Exception {  
    ObjectMapper objectMapper = new ObjectMapper();  
    String jsonStr = "{\"name\":\"myName\",\"birthday\":1416299461556}";  
    TestPOJO testPOJO = objectMapper.readValue(jsonStr,TestPOJO.class);  
    Assert.assertEquals("myName",testPOJO.getName());  
    System.out.println(testPOJO.getBirthday());  
}  

public static class TestPOJO{  
    private String name;  
    private Date birthday;  

    private TestPOJO(String name,Date birthday){  
        this.name = name;  
        this.birthday = birthday;  
    }  

    @JsonCreator  
    public static TestPOJO getInstance(@JsonProperty("name") String name
                                                                           ,@JsonProperty("birthday") long timestamp){  
        Date date = new Date(timestamp);  
        return new TestPOJO(name,date);  
    }  

    public String getName() {  
        return name;  
    }  

    public Date getBirthday() {  
        return birthday;  
    }  
}
```
这个实例中，TestPOJO的构造方法是私有的，外面无法new出来该对象，只能通过工厂方法getInstance来构造实例，此时@JsonCreator就标注在工厂方法上。
除了这2种方式外，还有一种构造方式成为授权式构造器，也是我们平常比较常用到的，这个构造器只有一个参数，且不能使用@JsonProperty。举例：

```java
@Test  
public void jsonCreator() throws Exception {  
    ObjectMapper objectMapper = new ObjectMapper();  
    String jsonStr = "{\"full_name\":\"myName\",\"age\":12}";  
    TestPOJO testPOJO = objectMapper.readValue(jsonStr,TestPOJO.class);  
    Assert.assertEquals("myName",testPOJO.getName());  
    Assert.assertEquals(12,testPOJO.getAge());  
}  

public static class TestPOJO{  
    private String name;  
    private int age;  
    @JsonCreator  
    public TestPOJO(Map map){  
        this.name = (String)map.get("full_name");  
        this.age = (Integer)map.get("age");  
    }  

    public String getName() {  
        return name;  
    }  

    public int getAge() {  
        return age;  
    }  
}  
```

8、@JacksonInject
作用于属性、方法、构造参数上，被用来反序列化时标记已经被注入的属性，举例：
```java
@Test  
public void jacksonInject() throws Exception {  
    ObjectMapper objectMapper = new ObjectMapper();  
    String jsonStr = "{\"age\":12}";  
    InjectableValues inject = new InjectableValues.Std().addValue("name","myName");  
    TestPOJO testPOJO = objectMapper.reader(TestPOJO.class).with(inject).readValue(jsonStr);  
    Assert.assertEquals("myName", testPOJO.getName());  
    Assert.assertEquals(12,testPOJO.getAge());  
}  

public static class TestPOJO{  
    @JacksonInject("name")  
    private String name;  
    private int age;  

    //getters、setters省略  
}
```

上面例子中我们在反序列化前通过InjectableValues来进行注入我们想要的属性

9、@JsonPOJOBuilder

作用于类，用来标注如何定制构建对象，使用的是builder模式来构建，比如Value v = new ValueBuilder().withX(3).withY(4).build();这种就是builder模式来构建对象，通常会喝 @JsonDeserialize.builder来配合使用，我们举个例子：

```java
@Test  
public void jacksonInject() throws Exception {  
    ObjectMapper objectMapper = new ObjectMapper();  
    String jsonStr = "{\"name\":\"myName\",\"age\":12}";  
    TestPOJO testPOJO = objectMapper.readValue(jsonStr,TestPOJO.class);  
    Assert.assertEquals("myName", testPOJO.getName());  
    Assert.assertEquals(12,testPOJO.getAge());  
}  

@JsonDeserialize(builder=TestPOJOBuilder.class)  
public static class TestPOJO{  
    private String name;  
    private int age;  

    public TestPOJO(String name, int age) {  
        this.name = name;  
        this.age = age;  
    }  

    public String getName() {  
        return name;  
    }  

    public int getAge() {  
        return age;  
    }  
}  

@JsonPOJOBuilder(buildMethodName = "create",withPrefix = "with")  
public static class TestPOJOBuilder{  
    private String name;  
    private int age;  

    public TestPOJOBuilder withName(String name) {  
        this.name = name;  
        return this;  
    }  

    public TestPOJOBuilder withAge(int age) {  
        this.age = age;  
        return this;  
    }  

    public TestPOJO create() {  
        return new TestPOJO(name,age);  
    }  
}
```

在 TestPOJOBuilder上有@JsonPOJOBuilder注解，表示所有的参数传递方法都是以with开头，最终构建好的对象是通过 create方法来获得，而在TestPOJO上使用了@JsonDeserializer，告诉我们在反序列化的时候我们使用的是 TestPOJOBuilder来构建此对象的

还有一些过期不推荐使用的注解，我们一笔带过，主要知道他们是跟哪些其他注解功能一样即可

@JsonGetter
作用于方法，1.0版本开始的注解，已经过期，不推荐使用，改用@JsonProperty

@JsonUseSerializer
作用于类和方法，1.5版本开始被移除了，改用@JsonSerialize

@JsonSetter
作用于方法，1.0版本开始的注解，已过期，不推荐使用，改用@JsonProperty

@JsonClass
作用于方法和类，1.9版本开始被移除了，改为@JsonDeserialize.as

@JsonContentClass
作用于方法，1.9版本开始被移除了，改为@JsonDeserialize.contentAs

@JsonKeyClass
作用于方法和类，1.9版本开始被移除了，改为@JsonDeserialize.keyAs

@JsonUseDeserializer
作用于方法和类，1.5版本开始被移除了，改为@JsonDeserialize
