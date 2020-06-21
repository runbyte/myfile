# Mybatis

## 配置文件

**application.properties**

```properties
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/demo?serverTimezone=UTC&useSSL=true
spring.datasource.username=root
spring.datasource.password=root
# xml文件扫描
mybatis.mapper-locations=classpath:mapper/*.xml
```

## CRUD

### 插入功能

```xml
<!-------------用法--------------->

<!--parameterType：是方法的参数类型，写全限定类名-->
<insert id="方法名" parameterType="参数类型">
    <!-- selectKey标签：用于向数据库添加数据之后，获取最新的主键值 -->
    <!-- resultType属性：得到的最新主键值的类型 -->
    <!-- keyProperty属性：得到的最新主键值，保存到JavaBean的哪个属性上 -->
    <!-- order属性：在insert语句执行之前还是之后，查询最新主键值。MySql是AFTER -->
	<selectKey resultType="主键值类型" keyProperty="主键值设置给哪个属性" order="AFTER">
    	select last_insert_id()
    </selectKey>
    insert into  表名称 (....) values (#{属性名}, #{属性名}, ...)
</insert>

<!-------------案例--------------->


<insert id="save" parameterType="com.hisilicon.domain.User">

    <selectKey resultType="int" keyProperty="id" order="AFTER">
        select last_insert_id()
    </selectKey>
    insert into user (id, username, birthday, address, sex) 
    values (#{id}, #{username}, #{birthday},#{address},#{sex})
</insert>
```

### 更新功能

```xml
<!--------------用法-------------->

<update id="方法名" parameterType="参数类型">
	update user set username=#{useranme},... where id = #{id}
</update>

<!-------------案例--------------->

<update id="edit" parameterType="com.hisilicon.domain.User">
    update user set username = #{username}, birthday = #{birthday}, 
    address = #{address}, sex = #{sex} where id = #{id}
</update>
```

### 删除功能

```xml
<!--------------用法-------------->

<delete id="方法名" parameterType="参数类型">
	delete from user where id = #{abc}
</delete>

<!-------------案例--------------->

<delete id="delete" parameterType="int">
    delete from user where id = #{id}
</delete>
```

### 查询功能

```xml
<!-- 根据主键查询 -->
<select id="findById" parameterType="int" resultType="com.hisilicon.domain.User">
    select * from user where id = #{id}
</select>

<!-- 模糊查询 -->
<select id="findByName" parameterType="string" resultType="com.hisilicon.domain.User">
    select * from user where username like #{username}
    或者
    select * from user where INSTR(username,#{username, jdbcType=VARCHAR}) > 0
    或者
    select * from user where username like '%${value}%'
</select>

<!-- 聚合函数查询 -->
<select id="findTotalCount" resultType="int">
    select count(*) from user
</select>
```



## 参数和结果集

### parameterType

**参数类型，用来告知mybatis你传入的参数是什么类型**

**简单类型**

```properties
参数写法：
例如：int, double, short 等基本数据类型，或者string
或者：java.lang.Integer, java.lang.Double, java.lang.Short,java.lang.String

SQL语句里获取参数：
如果是一个简单类型参数，写法是：#{随意}
```

**JavaBean**

```properties
参数写法：要写全限定类名
例如：parameterType是com.hisilicon.domain.User

SQL语句里获取参数：
#{JavaBean的属性名}
```

**复杂的 JavaBean**

```properties
参数写法：
在web应用开发中，通常有综合条件的搜索功能，例如：根据商品名称和所属分类同时进行搜索。这时候通常是把搜索条件封装成JavaBean对象；JavaBean中可能还有JavaBean。

SQL里取参数：#{xxx.xx.xx}
```



### resultType

**resultType是查询select标签上才有的，用来设置查询的结果集要封装成什么类型的**

**简单类型**

```properties
结果集写法：
例如：int, double, short 等基本数据类型，或者string
或者：java.lang.Integer, java.lang.Double, java.lang.Short, java.lang.String
```

**JavaBean**

```properties
结果集写法：类的全限定名
例如：com.hisilicon.domain.User

注意：JavaBean的属性名要和查询到的字段名保持一致
```



### resultMap属性名和字段名不一致

**当JavaBean的属性名和数据库表字段名不一致的时候，查出来的结果集是无法封装到JavaBean中的**



**解决方案一：（用的少）**

**查询出来的结果集用别名表示，别名和JavaBean的属性名保持一致**

```xml
<!-- 例如：User中的属性是Username,表中的是name -->
select name as username from user;
```



**解决方案二：（推荐）**

**使用resultMap配置字段名和属性名的对应关系**

```xml
<!-- resultMap标签：设置结果集中字段名和JavaBean属性的对应关系 -->
<!-- id属性：resultMap 的唯一标识 -->
<!-- type属性：要把查询结果的数据封装成什么对象，写全限定类名 -->
<resultMap id="userMap" type="com.hisilicon.domain.User">
    <!--id标签：主键字段配置 -->
	<!-- property：JavaBean的属性名；column：字段名 -->
    <id property="userId" column="id"/>
    <!--result标签：非主键字段配置。 property：JavaBean的属性名；  column：字段名-->
    <result property="username" column="username"/>
    <result property="userBirthday" column="birthday"/>
    <result property="userAddress" column="address"/>
    <result property="userSex" column="sex"/>
</resultMap>

<select id="queryAll" resultMap="userMap">
    select * from user
</select>
```



## 动态SQL

**实际开发中，SQL语句通常是动态拼接的，mybatis提供了一些xml的标签，用来实现动态SQL拼接**

**常用的标签有：**

```xml
<if></if>：用来进行判断，相当于Java里面if判断

<where></where>：通畅和if配合，用来代替sql语句中的 where 1=1

<foreach></foreach>：用来遍历集合，把集合里的内容拼接到SQL语句中
例如拼接：in(value1,value2,...)

<sql></sql>：用于定义SQL片段，达到重复使用的目的
```



### `<if>`标签

**语法介绍**

```xml
<if test="判断条件，使用OGNL表达式进行判断">
	SQL语句内容， 如果判断为true，这里的SQL语句就会进行拼接
</if>

<!-- 需求：根据用户的名称和性别搜索用户信息。把搜索条件放到User对象里，传递给SQL语句 -->
<select id="search" parameterType="user" resultType="user">
    select * from user where 1=1
    <if test="username != null">
        and username like #{username}
    </if>
    <if test="sex != null">
        and sex = #{sex}
    </if>
</select>
```

**注意：**

**SQL语句中， `where 1=1` 不能省略**

**在if标签的test属性中，直接写OGNL表达式，从parameterType中取值进行判断，不需要加#{}或者${}**



### `where`标签

**语法介绍**

上述案例中，写了`where 1=1`，如果不写的话，SQL语句或出现语法错误，可以用`<where>`标签替代

```xml
<!--  需求：优化上面的代码  -->
<select id="search" parameterType="user" resultType="user">
    select * from user
    <where>
        <if test="username != null">
            and username like #{username}
        </if>
        <if test="sex != null">
            and sex = #{sex}
        </if>
    </where>
</select>
```

**注意：**

**`where`标签替代了`where 1=1`**

**`where`标签内拼接的SQL没有变化，每个if的SQL中都要写`and`**

**使用`where`标签时，mybatis会自动处理掉条件中的第一个`and`，以保证SQL语法正确**



### `<foreach>`标签

**语法介绍**

foreach标签，通常用于循环遍历一个集合，把集合的内容拼接到SQL语句中

```xml
<!-- 例如，我们要根据多个id查询用户信息，SQL语句 -->
select * from user where id = 1 or id = 2 or id = 3;
select * from user where id in (1, 2, 3);
```

假如我们传参了id的集合，那么在xml文件中，遍历集合拼接SQL语句可以使用`foreach`标签实现

```xml
<!-- 
foreach标签：
	属性：
		collection：被循环遍历的对象，使用OGNL表达式获取，注意不要加#{}
		open：SQL语句的开始部分
		item：定义变量名，代表被循环遍历中每个元素，生成的变量名
		separator：分隔符
		close：SQL语句的结束部分
	标签体：
		使用#{OGNL}表达式，获取到被循环遍历对象中的每个元素
-->
<foreach collection="" open="id in(" item="id" separator="," close=")">
    #{id}
</foreach>
id in(第0个id的值, 第1个id的值,...)

<!-- 需求：QueryVO中有一个属性ids， 是id值的集合。根据QueryVO中的ids查询用户列表 -->
<select id="findByIds" resultType="com.*.user" parameterType="com.*.queryvo">
    select * from user
    <where>
        <foreach collection="ids" open="and id in (" item="id" separator="," close=")">
            #{id}
        </foreach>
    </where>
</select>
```



### `<sql>`标签

在映射配置文件中，我们发现有很多SQL片段是重复的，比如：`select * from user`

Mybatis提供了一个`<sql>`标签，把重复的SQL片段抽取出来，可以重复使用。

**语法介绍**

```xml
在映射配置文件中定义SQL片段：
<sql id="唯一标识">sql语句片段</sql>

在映射配置文件中引用SQL片段：
<include refid="sql片段的id"></include>
```

扩展：

如果想要引入其它映射xml中的sql片段，那么`<include>`标签的refid的值，需要在sql片段的id前指定namespace。

例如：

​	`<include refid="com.hisilicon.dao.RoleDao.selectRole"></include>`

​	表示引入了namespace为`com.hisilicon.dao.RoleDao`的映射配置文件中id为`selectRole`的sql片段

```xml
<!-- 在查询用户的SQL中，需要重复编写 select 把这部分SQL提取成SQL片段以重复使用 -->
<sql id="selectUser">
    select * from user
</sql>

<select id="findByIds" resultType="user" parameterType="queryvo">
    <!-- refid属性：要引用的sql片段的id -->
    <include refid="selectUser"></include> 
    <where>
        <foreach collection="ids" open="and id in (" item="id" separator="," close=")">
            #{id}
        </foreach>
    </where>
</select>
```



## 多表关联查询

### **准备实体**

**用户 - User **

**账号 - Account**

**对应关系：一个账号只能对应一个用户，一个用户可以对应多个账号**

```java
public class Account {
    private Integer id;
    private Integer userId;
    private Double money;
    //get/set...
    //toString
}

public class User {
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;
    //get/set...
    //toString
}
```



### 一对一（多对一）关联查询

需求：查询账户（Account）信息，及其关联的用户（User）信息



**方案一：类继承方式（不推荐）**

**方案二：类引用的方式（推荐）**

**JavaBean中要有关联JavaBean的引用**

```properties
在Account中增加一个属性User，指向User对象
	把查询结果集中，Account信息封装到Account中
	把查询结果集中，User信息封装到Account对象的User中
```

**修改JavaBean：Account类**

**增加`User user`属性，用作封装User信息**

```java
public class Account {
    private Integer id;
    private Integer userId;
    private Double money;
    
    // 加上 User 属性用作封装User信息
    private User user;

    //get/set...
    //toString...
}
```

**在AccountDao中增加方法**

```java
List<Account> queryAllAccount();
```

**在AccountDao.xml中增加statement**

```xml
<select id="queryAllAccounts2" resultMap="AccountUserMap">
    SELECT
    	a.id aid,
    	a.userId userId,
    	a.money money,
    	u.* 
    FROM
    	account a 
    LEFT JOIN 
    	USER u ON a.userId = u.id
</select>

<resultMap id="AccountUserMap" type="com.hisilicon.domain.account">
    <id property="id" column="aid"/>
    <result property="userId" column="userId"/>
    <result property="money" column="money"/>
    <!-- 
	association：用于把结果集中某些列的数据，封装到JavaBean中关联的一个对象上。用于一对一情形
		property：把数据封装到哪个属性关联的对象上
		javaType：关联的对象的类型
	-->
    <association property="user" javaType="com.hisilicon.domain.User">
        <id property="id" column="id"/>
        <result property="username" column="username"/>
        <result property="birthday" column="birthday"/>
        <result property="sex" column="sex"/>
        <result property="address" column="address"/>
    </association>
</resultMap>
```

**注意：SQL语句查询结果集中，不能有重名列。如果有，给列起别名保证没有重名列**



### 一对多（多对多）关联查询

需求：查询所有用户（User）信息，以及每个用户拥有的所有账号信息（Account信息）



**JavaBean中要有关联JavaBean的集合**

```properties
在User中增加一个属性accounts，类型是List<Account>
	把查询结果集中，User的信息封装到User中
	把查询结果集中，Account的信息封装到User对象的accounts中
```

**修改JavaBean：User类**

**增加`List<Account> accounts`用于保存用户拥有的账号信息集合**

```java
public class User {
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;
    
    /**增加属性：account的集合*/
    private List<Account> accounts;

    //get/set...
    //toString...
}
```

**在UserDao中增加方法**

```java
List<User> queryAllUser();
```

**在UserDao.xml中增加statement**

```xml
<select id="queryAllUsers" resultMap="userAccountsMap">
    SELECT
    	a.id aid,
    	a.userId userId, 
    	a.money money, 
    	u.*
    FROM
    	USER u 
    LEFT JOIN 
    	account a ON u.id = a.userId
</select>

<resultMap id="userAccountsMap" type="com.hisilicon.domain.User">
    <id property="id" column="id"/>
    <result property="username" column="username"/>
    <result property="birthday" column="birthday"/>
    <result property="sex" column="sex"/>
    <result property="address" column="address"/>

    <!--
       collection:用于封装JavaBean中某一属性关联的集合，用于一对多情形
       		property：封装哪个属性关联的集合
       		ofType：集合中的数据类型是什么
    -->
    <collection property="accounts" ofType="com.hisilicon.domain.Account">
        <id property="id" column="aid"/>
        <result property="userId" column="userId"/>
        <result property="money" column="money"/>
    </collection>
</resultMap>
```

**注意：SQL语句查询结果集中，不能有重名列。如果有，给列起别名保证没有重名列**



## mybaits的延迟加载

在**多表关联查询**时，比如查询用户信息，及其关联的帐号信息，在查询用户时就直接把帐号信息也一并查询出来了。但是在实际开发中，并不是每次都需要立即使用帐号信息，这时候，就可以使用延迟加载策略了。



### 延迟加载概念

**立即加载：**

* 不管数据是否需要使用，只要调用了方法，就立即发起查询。
* 比如：查询帐号，得到关联的用户；查询用户，得到关联的帐号



**延迟加载：（按需加载、懒加载）**

* 只有当真正使用到数据的时候，才发起查询。不使用不发起查询
* 比如：查询用户信息，不使用accounts的时候，不查询帐号的数据；只有当使用了用户的accounts，Mybatis再发起查询帐号的信息



优点：先从单表查询，需要使用关联数据时，才进行关联数据的查询。单表查询的速度要比多表关联查询速度快，性能高；内存占用小



缺点：当需要使用数据时才会执行SQL。这样大批量的SQL执行的情况下，会造成查询等待时间比较长



### 使用场景

一对一（多对一），通常不使用延迟加载（建议）。比如：查询帐号，关联加载用户信息

一对多（多对多），通常使用延迟加载（建议）。比如：查询用户，关联加载帐号信息



## mybatis的缓存

### 一级缓存

**一级缓存介绍**

* 一级缓存，是SqlSession对象提供的缓存（无论用不用，始终都有的）。
* 执行一次查询之后，查询的结果（JavaBean对象）会被缓存到SqlSession中。
* 再次查询同样的数据，Mybatis会优先从缓存中查找；如果找到了，就不再查询数据库。

**一级缓存的清除**

* 当调用了SqlSession对象的修改、添加、删除、commit()、close()、clearCache()等方法时，一级缓存会被清空。

**效果演示**

```java
/**
 * 测试 Mybatis的一级缓存：
 * SQL语句执行了一次、输出user1和user2的地址是相同的
 * 说明Mybatis使用了缓存
 */
@Test
public void testLevel1Cache(){
    User user1 = dao.findUserById(41);
    System.out.println(user1);

    User user2 = dao.findUserById(41);
    System.out.println(user2);
}

/**
 * 测试  清除Mybatis的一级缓存
 * 两次打印的User地址不同，执行了两次SQL语句
 * SqlSession的修改、添加、删除、commit()、clearCache()、close()都会清除一级缓存
 */
@Test
public void testClearLevel1Cache(){
    User user1 = dao.findUserById(41);
    System.out.println(user1);

    session.clearCache();

    User user2 = dao.findUserById(41);
    System.out.println(user2);
}
```



### 二级缓存

**二级缓存概念**

* 指Mybatis中SqlSessionFactory对象的缓存。
* 由同一个SqlSessionFactory对象生产的SqlSession对象，同样的映射器Mapper共享其缓存。
* 注意：
  * 二级缓存，缓存的是序列化之后的数据；
  * 当从缓存里取数据时，要进行反序列化还原成JavaBean对象
  * 要求JavaBean必须实现序列化接口`Serializable`
  * **二级缓存需要手动开启**

**修改Mybatis核心配置文件，开启全局的二级缓存开关**

**修改映射配置文件UserDao.xml，让映射器支持二级缓存**

```xml
<mapper namespace="com.hisilicon.dao.UserDao">
    <!-- 把cache标签加到映射配置文件 mapper标签里 -->
    <cache/>
    ......
</mapper>
```

**修改映射配置文件UserDao中的findById，让此方法（statement）支持二级缓存**

```xml
<!-- 如果statement的标签上，设置有的useCache="true"，表示此方法要使用二级缓存 -->
<select id="findById" parameterType="int" resultType="user" useCache="true">
    select * from user where id = #{id}
</select>
```

**修改JavaBean：User，需要实现Serializable接口**

**效果演示**

```java
/**
 * 测试二级缓存。
 * 测试结果：
 *      虽然输出的user1和user2地址不同，但是SQL语句只执行了一次，说明第二次用了缓存。
 *      Mybatis的二级缓存，保存的不是JavaBean对象，而是散列的数据。
 *      当要获取缓存时，把这些数据重新组装成一个JavaBean对象，所以地址不同
 */
@Test
public void testLevel2Cache(){
    SqlSession session1 = factory.openSession();
    UserDao dao1 = session1.getMapper(UserDao.class);
    User user1 = dao1.findUserById(41);
    System.out.println(user1);
    session1.close();

    SqlSession session2 = factory.openSession();
    UserDao dao2 = session2.getMapper(UserDao.class);
    User user2 = dao2.findUserById(41);
    System.out.println(user2);
    session2.close();
}
```



## mybatis的注解开发

Mybatis也支持注解开发。但是需要明确的是，Mybatis仅仅是把**映射配置文件** 使用注解代替了；而Mybatis的核心配置文件，仍然是xml配置。



### 常用注解介绍

- @Select：相当于映射配置文件里的select标签：用于配置查询方法的语句
- @Insert：相当于映射配置文件里的insert标签
- @SelectKey：相当于映射配置文件里的selectKey标签，用于添加数据后获取最新的主键值
- @Update：相当于映射配置文件里的update标签
- @Delete：相当于映射配置文件里的delete标签
- @Results：相当于映射配置文件里的resultMap标签
- @Result：相当于映射配置文件里的result标签，和@Results配合使用，封装结果集的
- @One：相当于映射配置文件里的association，用于封装关联的一个JavaBean对象
- @Many：相当于映射配置文件里的collection标签，用于封装关联的一个JavaBean对象集合

**注意：实际开发中，通常是注解开发加xml开发。**

* 一个功能，不能重复配置：注解配置一次， xml配置一次，mybatis会报错
* 映射配置文件名称、位置，要和映射器一样的
* 开发时：简单功能使用注解，复杂功能使用xml



### 注解实现简单的CRUD

**查询所有用户**

调用该方法的时候，自动返回 `List<User>`

```java
@Select("select * from user")
List<User> queryAll();
```

**根据ID查询**

调用该方法的时候，自动返回`User`

```java
@Select("select * from user where id = #{id}")
User findById(Integer id);
```

**增加用户**

```java
@Insert("insert into user (id,username,birthday,sex,address) values (#{id},#{username},#{birthday},#{sex},#{address})")
@SelectKey(
    statement = "select last_insert_id()", //查询最新主键值的SQL语句
    resultType = Integer.class,  //得到最新主键值的类型
    keyProperty = "id",  //得到最新主键值，保存到哪个属性里
    before = false  //是否在insert操作之前查询最新主键值
)
void save(User user);
```

**修改用户**

```java
@Update("update user set username=#{username},birthday=#{birthday},sex=#{sex},address=#{address} where id=#{id}")
void edit(User user);
```

**删除用户**

```java
@Delete("delete from user where id = #{id}")
void delete(Integer id);
```



### 属性名和字段名不一致

```java
@Select("select * from user")
@Results({
    @Result(property = "userId", column = "id", id = true),
    @Result(property = "username", column = "username"),
    @Result(property = "userBirthday", column = "birthday"),
    @Result(property = "userSex", column = "sex"),
    @Result(property = "userAddress", column = "address")
})
List<User> queryAllUser();
```



### 注解实现多表关联查询

#### 一对一（多对一）关联查询，实现懒加载

需求：查询帐号信息，及其关联的用户信息



修改映射器接口UserDao，增加方法findById（供关联查询时使用）

```java
@Select("select * from user where id = #{id}")
User findById(Integer id);
```

创建JavaBean：Account，Account中要有User的引用，前边已经准备好

修改映射器接口AccountDao，增加方法

```java
@Select("select * from account where id = #{id}")
@Results({
    @Result(property = "id", column = "id", id = true),
    @Result(property = "uid", column = "uid"),
    @Result(property = "money", column = "money"),
    @Result(
        property = "user",
        javaType = User.class,
        column = "uid",
        one = @One(
            //一对一关联查询，调用select配置的statement，得到关联的User对象
            select = "com.hisilicon.dao.UserDao.findById",
            //FetchType.LAZY 表示要使用延迟加载
            fetchType = FetchType.LAZY
        )
    )
})
Account findById(Integer id);
```



#### 一对多（多对多）关联查询，实现懒加载

需求：查询用户信息，及其关联的帐号集合信息



修改映射器AccountDao，增加方法findAccountsByUid（供关联查询时使用）

```java
@Select("select * from account where uid = #{uid}")
List<Account> findAccountsByUid(Integer uid);
```

修改JavaBean：User，User中需要有Account的集合，前边已经准备好

修改映射器接口UserDao，增加方法

```java
/**
     * 查询用户信息，及其关联的帐号信息集合
     * @param id
     * @return
     */
@Select("select * from user where id = #{id}")
@Results({
    @Result(property = "id",column = "id",id = true),
    @Result(property = "username",column = "username"),
    @Result(property = "birthday",column = "birthday"),
    @Result(property = "sex",column = "sex"),
    @Result(property = "address",column = "address"),
    @Result(
        property = "accounts",
        javaType = List.class, //注意，这里是List.class，而不是Account.class
        column = "id",
        many = @Many(
            //一对多关联查询，调用select对应的statement，得到帐号集合
            select = "com.itheima.dao.AccountDao.findAccountsByUid",
            //FetchType.LAZY 表示要使用延迟加载
            fetchType = FetchType.LAZY
        )
    )
})
User findUserAccountsById(Integer id);
```



### 注解开发总结

注解用来代替映射配置文件

常用注解：
* 配置statement的注解
  * `@Select`
  * `@insert`和`@SelectKey`
  * `@Update`
  * `@Delete`
* 结果集封装的注解：
  * `@Results`：相当于resultMap标签
  * `@Result`：相当于result/id/collection/association标签
  * `@One`：用于配置`association`
  * `@Many`：用于配置`collection`