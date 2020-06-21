# mybatis

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

#### 属性名和字段名不一致

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

​	表示引入了namespace为`com.itheima.dao.RoleDao`的映射配置文件中id为`selectRole`的sql片段

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

