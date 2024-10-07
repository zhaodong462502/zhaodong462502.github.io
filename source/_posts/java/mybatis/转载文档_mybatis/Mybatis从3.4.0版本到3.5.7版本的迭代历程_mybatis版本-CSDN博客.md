## [Mybatis](https://so.csdn.net/so/search?q=Mybatis&spm=1001.2101.3001.7020)从3.4.0版本到3.5.7版本的迭代历程

___

### 一、3.4.0

#### 1、主要的功能增强

1.  在SqlSession中新增Cursor List方法。
2.  BatchExecutor和ReuseExecutor操作支持事务超时处理，主要是修复通过spring注解@Transactional指定超时未使用到sql的情况
3.  更好的支持泛型
4.  支持Java8新的Date和Time API(JSR-310)

```java
<T> Cursor<T> selectcursor(String statement); <T> Cursor<T> selectcursor(string statement, object parameter); <T> Cursor<T> selectcursor(string statement, Object parameter, RowBounds rowBounds);
```

#### 2、selectCursor example

```xml
<select id="getEmployeeNestedCursor"resultMap="results" resultOrdered="true"> select * from employees order by id </select>
```

```java
// Get a list which can have millions of rows Cursor<Employee> employees=sq1Session.selectCursor("getEmployeeNestedCursor"); // Get an iterator on employees Iterator<Employee> iter = employees.iterator(); List<Employee> smallChunk = new ArrayList<>(10); while(iter.hasNext()){ // Fetch next 10 employees for(int i = 0; i<10 && iter.hasNext(); i++){ smallChunk.add(iter.next()); } doSomethingWithAlreadyFetchedEmployees(smallChunk); smallChunk.clear(); }
```

#### 3、不兼容的更改

1.  Transaction接口新增getTimeout()方法，如有实现该接口需特别注意。
2.  注解@options()参数flushCache改用枚举。
3.  因为增加对事务超时的支持，所以StatementHandler#prepare(Connection)更改为StatementHander#prepare(Connection，Integer)，后一个参数取事务超时时间。
4.  添加了SqlSession#selectCursor、Executor#queryCursor、ResultSetHander#handleCursorResultsSets和StatementHandler#queryCursor接口，如有实现需特别注意。

```java
public enum FlushCachePolicy { /** <code>false</code> for select statement; * <code>true</code> for insert/update/delete statement. * */ DEFAULT, /** Flushes cache regardless of the statement type. */ TRUE, /** Does not flush cache regardless of the statement type. */ FALSE }
```

### 二、3.4.1

#### 1、主要的功能增强

1.  使用Java 8 -parameters选项编译时，允许通过其声明的名称引用参数，处理代码详见ParamNameResolver。  
    ~ 可以在mapper类上省略@Param注解指定参数名（前提xml中的名称要与参数名一致）  
    ~ 默认开启
2.  根据JSR310 对日期和时间新的API定义，增加对month和year的类型处理器，详见MonthTypeHandler和YearTypeHandler。
3.  @Select 返回的数据类型支持对象数组，详见MapperAnnotationBuilder#getReturnType
    
    ```java
    @Select("select * from users") User[] getUsers();
    ```
    
4.  logImpl配置参数支持自定义xml配置，XMLConfigBuilder。

#### 2、修复的错误

1.  修复当在xml映射文件中指定了columnPrefix时，循环引用的resultMap也会填充祖父对象对应的值。

```xml
<resultMap id="user" type="User"> <id property="id" column="id"/> <result property="name" column="name"/> <association property="superior" resultMap="user" columnPrefix="superior_"/> </resultMap> <select id="find" parameterType="long" resultMap="user"> select 1 as id, 'a' as name, 2 as superior_id, 'b' as superior_name from dual </select>
```

```java
User user = dao.find(1L);
```

2.  修复在 IBM WebSphere Application Server 8.5.5.9 上启动时抛出RuntimeException。
3.  修复Cursor 无法作为@Select语句的返回类型  
    PS：3.4.0增加了selectCursor游标处理查询结果的方法，但是无法支持@Select
4.  修复与 Kylin JDBC 驱动程序一起使用时抛出 NullPointerException。
5.  修复RowBounds无法作为返回类型为 Cursor 的 select 语句的参数。

```java
Cursor<User> usersCursor = sqlSession.selectCursor( "getAllUsers", null, new RowBounds(1, 3));
```

6.  Select 语句@Param不能用作关联的嵌套选择语句。

### 三、3.4.2

#### 1、主要的功能增强

1.  增加returnInstanceForEmptyRow配置，以控制当查询结果中所有列都为null时返回空对象，而不是null。
2.  支持mapper接口中的默认方法。
3.  TypeHandler支持递归搜索。
4.  @CacheNamespace注解增加properties属性。

```java
Property[] properties() default {};
```

5.  允许在占位符上指定默认值。  
    PS：默认情况下该功能禁用，可使用org.apache.ibatis.parsing.PropertyParser.enable-default-value=true配置来开启。
6.  新增JSR310 1.0.2定义的类型处理器。

#### 2、修复的错误

1.  aggressiveLazyLoading 的默认值更改为false。
2.  使用keyProperty指定的字段找不到时抛出异常。

```java
@Options(useGeneratedKeys = true, keyProperty = "id")
```

### 四、3.4.3

#### 1、主要的功能增强

1.  增加为枚举的公共接口注册类型处理程序。  
    [https://github.com/mybatis/mybatis-3/pull/947](https://github.com/mybatis/mybatis-3/pull/947)
2.  在MapperAnnotationBuilder中Jdbc3KeyGenerator 和 NoKeyGenerator 分别共享自己的同一个实例，详见MapperAnnotationBuilder#parseStatement。
3.  可以通过SQL Builder构建update … join 语句。  
    ~ 以下代码3.4.3以前不生效

```java
new SQL() .UPDATE("table_1 a") .INNER_JOIN("table_2 b USING (ID)") .SET("a.value = b.value");
```

#### 2、修复的错误

1.  修复找不到祖父级mapper接口上的方法。  
    ~ 如下代码中，super\_super\_a接口中的方法在3.4.3以前的版本中是无法被遍历到的

```java
public interface a extends super_a public interface super_a extends super_super_a
```

2.  修复当LanguageDriver为null时，将实例对象放在“LANGUAGE\_DRIVER\_MAP”中，而不是“驱动程序”对象中的错误，详见LanguageDriverRegistry。
3.  修复布尔值不能同时映射到get和is。
4.  解决解析嵌套结果过多内存分配的问题。
5.  修复接口默认方法只支持public的情况。  
    ~ 以下代码3.4.3以前是不能支持的，因为Mapper是包级别的访问权限

```java
@Mapper interface Mapper extends SupperMapper{ @Select("select * from login_user") Cursor<Map<String, Object>> select(); @Select("select id from login_user") Integer[] selectIds(); default List<Map<String, Object>> defaultGetUser(){ return selectList(); } }
```

7.  修复resultMap和as别名之间不应该出现的自动映射。  
    ~ [https://github.com/mybatis/mybatis-3/pull/1100](https://github.com/mybatis/mybatis-3/pull/1100)  
    ~ 对于 ≤ 3.4.2，即使在结果映射中明确定义了字段映射，就算[SQL语句](https://so.csdn.net/so/search?q=SQL%E8%AF%AD%E5%8F%A5&spm=1001.2101.3001.7020)中字段与映射中定义的字段不匹配，也会对该字段进行自动映射。  
    ~ 对于 ≥ 3.4.3，自动映射不适用于在结果映射中显式映射的属性。
    
    以下代码中phone在版本3.4.3以后中返回null
    

```xml
<resultMap id="user" type="User"> <id property="id" column="id"/> <result property="name" column="name"/> <result property="phone" column="phone_number"/> </resultMap> <select id="getUser" resultMap="user"> select id, name, phone_number as phoneNumber from user where id = 1 </select>
```

### 五、3.4.4

#### 1、主要的功能增强

1.  无功能更新，仅仅修复了3.4.3版本中Meven Central 上的 JAR 中的一个错误。

### 六、3.4.5

#### 1、主要的功能增强

1.  允许为enum自定义默认的TypeHandler，详见EnumOrdinalTypeHandler。
2.  sql provider 方法上支持 ProviderContext用于构建通用mapper。
3.  将 JSR-310（Java 日期和时间 API）的类型处理程序合并到master。
4.  允许在sql provider中替换ProviderSqlSource 上的属性配置。

```xml
<properties> <property name="Constants.LOGICAL_DELETE_OFF" value="false"/> </properties>
```

```java
public String buildSql(@Param("name") final String name){ return new SQL(){{ SELECT("*"); FROM("test"); if(name != null){ WHERE("name = #{name}"); } WHERE("logical_delete = #{Constants.LOGICAL_DELETE_OFF}"); }}.toString(); }
```

#### 2、修复的错误

1.  修复在两个select之间执行insert或者update时，并非所有的select结果都得到处理。

```xml
<select id="multi" resultMap="userResult,groupsResult"> select * from mbtest.order_detail; insert into mbtest.order_detail (order_id, line_number) values (1, 132); select * from mbtest.order_header; </select>
```

2.  延迟加载不能覆盖用户设置的属性值。  
    ~ [https://github.com/mybatis/mybatis-3/issues/988](https://github.com/mybatis/mybatis-3/issues/988)
3.  修复当使用size为属性名时用integer映射的错误。  
    以下代码在版本3.4.5之前无法正常运行

```java
@Insert("INSERT INTO test(number, size) VALUES(#{number}, #{size})") public void insert(@Param("number") long number, @Param("size") long size);
```

4.  修复PostgreSQL中useGeneratedKeys=true，当 keyProperty 指定的‘id’没有setter则抛出 ExecutorException。
5.  修复使用foreach标签污染全局上下文。  
    以下代码其中item=status，会污染 and status =#{status}中的status

```xml
<select> select * from tb <where> <if test="status != null"> and status = #{status} </if> <if test="statusList != null"> and status in <foreach collection="statusList" item="status", open="(" separator="," close=")"> #{status} </foreach> </if> </where> </select>
```

### 七、3.4.6

#### 1、主要的功能增强

1.  将自定义 ResultHandler应用于cursor类型的返回。  
    ~ [https://github.com/mybatis/mybatis-3/issues/493](https://github.com/mybatis/mybatis-3/issues/493)
2.  BatchExecutor在遍历执行sql语句之后立即关闭其语句。
3.  当解析xml文件失败时，将资源路径添加到异常消息中。
4.  @SelectProvider注解支持静态方法，详见ProviderSqlSource。  
    ~ 如下代码

```java
@SelectProvider(type=SqlGenerator.class, method="selectIds") Integer[] selectIds(); class SqlGenerator{ public static String selectIds(){ return "select id from login_user"; } }
```

5.  ProviderSqlSource构造sql语句时支持返回CharSequence。

```java
@SelectProvider(type=SqlGenerator.class, method="selectIds") Integer[] selectIds(); class SqlGenerator{ public static CharSequence selectIds(){ return "select id from login_user"; } }
```

6.  可以替换include中的占位符。  
    ~ 以下代码在版本3.4.6以前无效，不能填充include标识的sql

```xml
<select id="selectList" resultType="java.util.Map" resultOrdered="true"> select <include refid="colsIfAttribute"> <property name="value" value="y"/> </include> from login_user </select> <sql id="colsIfAttribute"> <if test="'${value}' == 'x'"> * </if> <if test="'${value}' == 'y'"> id </if> </sql>
```

#### 2、修复的错误

1.  修复自定义HashMap类型处理器发生ClassCastException异常的情况。  
    ~ [https://github.com/mybatis/mybatis-3/commit/9233556404a3b6cc122d42c8ce0c97ba9e01b7ce](https://github.com/mybatis/mybatis-3/commit/9233556404a3b6cc122d42c8ce0c97ba9e01b7ce)
2.  修复序列化和反序列化缓存的数据导致NullPointerException的错误。
3.  修复调用TypeHandlerRegistry.hasTypeHandler 后无法注册 TypeHandler的问题，详见TypeHandlerRegistry。

### 八、3.5.0

#### 1、主要的功能增强

1.  避免在 JDK 9+ 上出现“非法反射访问”警告。
2.  添加自动模块名称。
3.  支持java.util.Optional作为执行器返回值。

```java
public interface UserMapper{ @Select("select * from users where id = #{id}") Optional<User> findOne(Integer id); }
```

4.  将BaseTypeHandler中的wasNull判断移到子类去做。  
    ~ [https://github.com/mybatis/mybatis-3/pull/1244](https://github.com/mybatis/mybatis-3/pull/1244)
5.  xml中constructor标签支持columnPrefix。

```xml
<resultMap id="authorRM" type="Author"> <id property="id" column="id"/> <result property="name" column="name"/> </resultMap> <resultMap id="articleRM" type="Article"> <constructor> <idArg column="id" javaType="int"/> <arg column="name" javaType="String"/> <arg resultMap="authorRM" columnPrefix="author_" javaType="Author"/> <arg resultMap="authorRM" columnPrefix="coauthor_" javaType="Author"/> </constructor> </resultMap>
```

6.  提高寻找TypeHandler自动映射结果的性能。
7.  当没有找到keyProperty指定的key时抛出异常。
8.  新增SqlxmlTypeHandler类型处理器，用于将sqlxml转换为string，详见SqlxmlTypeHandler。
9.  标签可以直接将逗号跟在sql语句后面。

```xml
-- 版本3.5.0以前的写法 -- <set> <if test="name != null"> name = #{name} </if> <if test="desc != null"> ,desc = #{desc} </if> </set> -- 版本3.5.0以后的写法 -- <set> <if test="name != null"> name = #{name}, </if> <if test="desc != null"> desc = #{desc} </if> </set>
```

10.  运行OGNL访问表达式上私有、包私有和受保护的属性。

```java
@Getter public class Point{ private int lid; private int x; private int y; }
```

```xml
<insert id="save"> insert into points (lid, x, y) values <foreach item="point" separator=","> (#{point.lid}, #{point.x}, #{point.y}) </foreach> </insert>
```

11.  mapper xml中使用标签如果不指定resultType，自动从resultMap标签的type中获取。

```xml
<resultMap type="Owner" id="OwnerMap"> <id property="id" column="id"/> <result property="name" column="name"/> <discriminator javaType="String" column="vehicle_id"> <case value="car"> <association property="vehicle" column="vehicle_id" select="CarMapper.get"/> </case> </discriminator> </resultMap>
```

12.  支持 Log4J 2.6+。
13.  将测试框架升级到 JUnit 5。
14.  改进了与仅支持 JDBC 3 API 的驱动程序的兼容性。
15.  同时使用@CacheNamespace和不会再抛出异常。

#### 2、修复的错误

1.  修复OffsetDateTimeTypeHandler、OffsetTimeTypeHandler 和 ZonedDateTimeTypeHandler 丢失时区的问题。
2.  修复`Cursor`与 Db2 一起使用时发生的 SQLException。
3.  修复当类层次结构超过 3 个级别时，无法正确解析通用类型参数。

#### 3、不向后兼容的更改

1.  在3.5.0中使用Cursor需要 JDBC 4.1 API 的驱动程序。
2.  如果使用`org.apache.ibatis.type.BaseTypeHandler`自定义了类型处理器，需要检查wasNull()。
3.  更正 JdbcTransaction.java 中的错字（autoCommmit -> autoCommit）。
4.  keyProperty不再有默认值。

### 九、3.5.1

#### 1、主要的功能增强

1.  Add LanguageDriver to ProviderSqlSource。  
    ~ [https://github.com/mybatis/mybatis-3/pull/1391](https://github.com/mybatis/mybatis-3/pull/1391)
2.  LONGVARCHAR的默认类型处理器从ClobTypeHandler更改为StringTypeHandler。
3.  如果在使用@SelectProvider时，方法名和对应的provider方法名一样可以省略@SelectProvider中method参数。

```java
@SelectProvider(type=SqlGenerator.class) Integer[] selectIds(); class SqlGenerator{ public static CharSequence selectIds(){ return "select id from login_user"; } }
```

4.  ProviderContext构造sql语句时支持mybatis-config.xml配置的databaseId属性值。

```java
interface DatabaseIdMapper{ @SelectProvider(type=SqlGenerator.class) String selectDatabaseId(); } class SqlProvider{ public static String providerSql(ProviderContext context){ if("hsql".equals(context.getDatabaseId())){ return "select " + context.getDatabaseId() + " from user"; } else{ return "select " + context.getDatabaseId() + " from user"; } } }
```

#### 2、修复的错误

1.  修复LocalTimeTypeHandler丢失小数秒部分。
2.  修复LocalDateTypeHandler和LocalDateTimeTypeHandler处理日期的一些异常。
3.  修复keyProperty用参数名称指定在批量插入的时候报错。  
    ~ keyProperty = “n.id”在批量插入中会报错。

```java
@Options(useGeneratedKeys = true, keyProperty="n.id" keyColumn="id") int insert(@Param("n") User user);
```

4.  修复当枚举值中有实现方法时找不到类型处理器的错误。

#### 3、不向后兼容的更改

LocalDateTypeHandler、LocalTimeTypeHandler和LocalDateTimeTypeHandler只有在支持JDBC 4.2 API的JDBC驱动程序才有效。

### 十、3.5.2

#### 1、主要的功能增强

1.  sql builder支持LIMIT、OFFSET和FETCH FIRST。
2.  在Provider注解上增加value 属性，如InsertProvider。
3.  支持数组对象作为参数入参。  
    ~ [https://github.com/mybatis/mybatis-3/pull/1548](https://github.com/mybatis/mybatis-3/pull/1548)
4.  sql builder支持多行插入语法。

```java
public void insert(){ final String sql = new SQL(){{ .INSERT_INTO("user") .INTO_VALUES("id", "name") .ADD_ROW() .INTO_VALUES("#{id}", "#{name}") }}.toString(); }
```

5.  当只有单个参数条件的时候，允许任意数据类型。

#### 2、修复的错误

修复DefaultResultSetHandler可能存在的npe。

### 十一、3.5.3

#### 1、主要的功能增强

1.  更新了默认方法调用以支持 JDK 14+8。
2.  在Provider注解上增加value 属性，如InsertProvider。
3.  getter/setter仅在真正访问到时才会抛出ReflectionException。
4.  在标签中支持CDATA变量。

```xml
<sql id="colsSuffix"> <![CDATA[ col_${suffix} ]]> </sql>
```

#### 2、修复的错误

修复在迭代Cursor时，如果下一个元素为空出错的情况。

### 十二、3.5.4

#### 1、主要的功能增强

支持在同一个方法上多次使用@Arg和@Result注解。

```java
@Result(property="id", colum="id") @Result(property="name", colum="name") @Select("select * from user where id = #{id}") User selectOne(int id);
```

#### 2、修复的错误

1.  修复自动生成id时，如果重写了hashCode方法，不再调用对应hashCode方法，否则抛出npe。
2.  修复TypeParameterResolver 中的 TypeVariable 解析可能不正确。  
    ~ [https://github.com/mybatis/mybatis-3/issues/1794](https://github.com/mybatis/mybatis-3/issues/1794)
3.  解决TypeHandlerRegistry 中的竞争条件。  
    ~ [https://github.com/mybatis/mybatis-3/issues/1819](https://github.com/mybatis/mybatis-3/issues/1819)

### 十三、3.5.5

#### 1、主要的功能增强

支持在@One和@Many注解中获取resultMap和columnPrefix所需要的数据。

#### 2、修复的错误

1.  修复@CacheNamespaceRef可能出现的IllegalArgumentException。

### 十四、3.5.6

#### 1、主要的功能增强

1.  增加SQL\_SERVER\_SNAPSHOT事务隔离级别以支持MS SQL Server。
2.  当没有定义JEP-290 序列化过滤器时，将在反序列化对象流上记录一条 WARN 级别的消息。  
    ~[https://docs.oracle.com/pls/topic/lookup?ctx=javase15&id=GUID-8296D8E8-2B93-4B9A-856E-0A65AF9B8C66](https://docs.oracle.com/pls/topic/lookup?ctx=javase15&id=GUID-8296D8E8-2B93-4B9A-856E-0A65AF9B8C66)  
    ~ [https://github.com/mybatis/mybatis-3/pull/2079](https://github.com/mybatis/mybatis-3/pull/2079)
3.  指定defaultSqlProviderType以支持全局配置在sql provider的一些注解上@SelectProvider、@UpdateProvider，@InsertProvider和@DeleteProvider。

#### 2、修复的错误

1.  修复使用BlockingCache产生OutOfMemoryError的风险。  
    ~ [https://github.com/mybatis/mybatis-3/pull/2044](https://github.com/mybatis/mybatis-3/pull/2044)
2.  修复使用包作为子标签解析 typeAliases 元素时抛出 InvalidPathException。  
    ~ [https://github.com/mybatis/mybatis-3/issues/1974](https://github.com/mybatis/mybatis-3/issues/1974)

### 十五、3.5.7

#### 1、主要的功能增强

  改进jdk 8环境下的性能。

___