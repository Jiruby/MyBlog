---
title: Mybatis学习笔记
date: 2020-04-19 12:41:51
tags:
	- Mybatis
categories: 学习笔记
---

## 三、Mybatis

### （一）入门

#### 1. 什么是Mybatis

mybatis 是一个优秀的基于java的持久层框架，它内部封装了jdbc，使开发者只需要关注sql语句本身，而不需要花费精力去处理加载驱动、创建连接、创建statement等繁杂的过程。

mybatis通过xml或注解的方式将要执行的各种 statement配置起来，并通过java对象和statement中sql的动态参数进行映射生成最终执行的sql语句。

最后mybatis框架执行sql并将结果映射为java对象并返回。采用ORM思想解决了实体和数据库映射的问题，对jdbc 进行了封装，屏蔽了jdbc api 底层访问细节，使我们不用与jdbc api 打交道，就可以完成对数据库的持久化操作。

#### 2. 实现步骤

##### 2.1 导入MyBatis的坐标和其他相关坐标

```xml
<!--mybatis坐标-->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.5</version>
</dependency>
<!--mysql驱动坐标-->
<dependency>    
    <groupId>mysql</groupId>   
    <artifactId>mysql-connector-java</artifactId>    
    <version>5.1.6</version>    
    <scope>runtime</scope>
</dependency>
<!--单元测试坐标-->
<dependency>    
    <groupId>junit</groupId>    
    <artifactId>junit</artifactId>    
    <version>4.12</version>    
    <scope>test</scope>
</dependency>
<!--日志坐标-->
<dependency>    
    <groupId>log4j</groupId>    
    <artifactId>log4j</artifactId>    
    <version>1.2.12</version>
</dependency>
```

##### 2.2  创建user数据表

##### 2.3  编写User实体

##### 2.4  编写UserMapper映射文件 (注解开发可省略)

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="userMapper">    
	<select id="findAll" resultType="com.wyq.domain.User">        
		select * from User    
	</select>
     <!--插入-->
    <insert id="insert" parameterType="com.wyq.domain.User">
        insert into user values (#{id},#{username},#{password})
    </insert>
</mapper>
```

##### 2.5 编写MyBatis核心文件

```xml
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN“ "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>    
	<environments default="development">        
		<environment id="development">            
			<transactionManager type="JDBC"/>            
			<dataSource type="POOLED">                
				<property name="driver" value="com.mysql.jdbc.Driver"/>
				<property name="url" value="jdbc:mysql://localhost:3306/test"/>               
				<property name="username" value="root"/>
				<property name="password" value="root"/>            
			</dataSource>        
		</environment>    
	</environments>    
	<mappers> 
      	  <!--从mapper.xml文件加载-->
		<mapper resource="com/wyq/mapper/UserMapper.xml"/>
          <!--扫包, 从注解中加载-->
          <!--<package name="com.wyq.mapper"/>-->
	</mappers>
</configuration>
```

##### 2.6 测试

```java
@Test  //插入
public void test2() throws IOException {
    //0. 模拟user
    User user=new User();
    user.setUsername("jack");
    user.setPassword("qwe");
    //1. 加载xml文件   
    //Resources ：org.apache.ibatis.io.Resources;
    InputStream resourceAsStream = Resources.getResourceAsStream("SqlMapConfig.xml");
    //2. 获取工厂对象
    SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    //3. 创建Session
    SqlSession sqlSession = sessionFactory.openSession();
    //4. 执行sql
    sqlSession.insert("userMapper.insert",user);
    /**
     * 注意: MyBatis默认不会自动提交事务    // openSession(true) 自动提交
     */
    sqlSession.commit();
    //5. 释放资源
    sqlSession.close();
}
```

#### 3. 映射文件详解

![](http://image.jiruby.cn/2020-04-19%20124400.jpg)

#### 4. 配置文件详解

##### 4.1  environments标签

![](http://image.jiruby.cn/2020-04-19%20124300.jpg)

##### 4.2  mapper标签

该标签的作用是加载映射关系

##### 4.3  properties标签

该标签可以加载额外配置的properties文件

##### 4.4  typeAliases标签

类型别名是为Java 类型设置一个短的名字（\src\main\resources\SqlMapconfig.xml）

```xml
<typeAliases>
    <!--指定类设置别名-->
    <typeAlias type="com.wyq.domain.User"  alias="user"/>
    <!--扫描POJO包，全部设置别名-->
    <package name="com.wyq.domain"/>
</typeAliases>
```

整合的时候也可在Spring配置文件中设置（SSMTest\src\main\resources\applicationContext-dao.xml）

```xml
<!--配置整合bean-->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <!--配置别名-->
    <property name="typeAliasesPackage" value="com.wyq.ssm.po"/>
</bean>

<!--配置mapper扫描器，生产代理对象-->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.wyq.ssm.mapper"/>
</bean>
```

#### 5. 相关API

##### 5.1 工厂构建器SqlSessionFactoryBuilder

常用API：SqlSessionFactory  build(InputStream inputStream)

通过加载mybatis的核心文件的输入流的形式构建一个SqlSessionFactory对象

##### 5.2 工厂对象SqlSessionFactory

常用API：SqlSession openSession()   | SqlSession openSession(boolean autoCommit)

##### 5.3 会话对象SqlSession

执行语句的方法主要有：

```java
<T> T selectOne(String statement, Object parameter) 
<E> List<E> selectList(String statement, Object parameter) 
int insert(String statement, Object parameter) 
int update(String statement, Object parameter) 
int delete(String statement, Object parameter)
```

操作事务的方法主要有：

```java
void commit()  
void rollback() 
```

### （二）进阶篇

#### 1.Dao层实现

##### 1.1 传统开发方式

1. 编写UserDao接口
2. 编写UserDaoImpl实现
3. 测试

##### 1.2 代理开发方式

1. 代理开发方式介绍

   采用 Mybatis 的代理开发方式实现 DAO 层的开发---主流。

   Mapper 接口开发方法只需要程序员编写Mapper 接口（相当于Dao 接口），由Mybatis 框架根据接口定义创建接口的动态代理对象，由代理对象的方法实现功能。

   Mapper 接口开发需要遵循以下规范：

   **1) Mapper.xml文件中的namespace与mapper接口的全限定名相同**

   **2) Mapper接口方法名和Mapper.xml中定义的每个statement的id相同**

   **3) Mapper接口方法的输入参数类型和mapper.xml中定义的每个sql的parameterType的类型相同**

   **4) Mapper接口方法的输出参数类型和mapper.xml中定义的每个sql的resultType的类型相同**

2. 编写UserMapper接口

3. 测试

   ```java
   // 获取配置文件流
   InputStream resourceAsStream = Resources.getResourceAsStream("SqlMapconfig.xml");
   // 获取工厂对象
   SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
   // 开启会话对象
   SqlSession sqlSession = sqlSessionFactory.openSession(true);
   // 获取Mapper代理对象，使用接口接收
   UserMapper mapper = sqlSession.getMapper(UserMapper.class);
   User user = mapper.findById(1);
   
   ```

#### 2.动态SQL

##### 2.1 概述

Mybatis 的映射文件中，前面写SQL 都是比较简单的，有些时候业务逻辑复杂时，我们的 SQL是动态变化的，此时在前面的学习中我们的 SQL 就不能满足要求了。例如：多条件搜索

##### 2.2 动态 SQL  之  if

例如：SELECT * FROM USER WHERE id= 1 and  username = zhangsan

替换之前的解决的方案 where 1=1 and id=1 and username=zhangsan 的拼接方案

```xml
<select id="findByCondition" parameterType="user" resultType="user">
    select * from User
    <where>
        <if test="id!=0">
            and id=#{id}
        </if>
        <if test="username!=null">
            and username=#{username}
        </if>
    </where>
</select>

```

设置log4j （log4j.rootLogger=debug, stdout），观察测试代码执行的SQL语句

##### 2.3 动态 SQL  之  foreach

循环执行sql的拼接操作，例如：SELECT * FROM USER WHERE id IN (1,2,5)   

括号内的条件不确定

```xml
<select id="findByIds" parameterType="list" resultType="user">
    select * from user
    <!--<include refid="selectUser"/>-->
    <where>
        <foreach collection="list" open="id in (" close=")" item="id" separator=",">
            #{id}
        </foreach>
    </where>
</select>

```

**<foreach>**标签用于遍历集合，它的属性：

•collection：代表要遍历的集合元素，注意编写时不要写#{}

•open：代表语句的开始部分

•close：代表结束部分

•item：代表遍历集合的每个元素，生成的变量名

•sperator：代表分隔符

##### 2.4 SQL 片段抽取

Sql 中可将重复的 sql 提取出来，使用时用 include 引用即可，方便维护

#### 3. MyBatis核心配置文件补充

##### 3.1 typeHandlers标签

开发步骤：

1. 定义转换类继承类BaseTypeHandler<T>
2. 覆盖4个未实现的方法，其中setNonNullParameter为java程序设置数据到数据库的回调方法，getNullableResult为查询时 mysql的字符串类型转换成 java的Type类型的方法
3. 在MyBatis核心配置文件中进行注册

例如需求：一个Java中的Date数据类型，我想将之存到数据库的时候存成一个1970年至今的毫秒数，取出来时转换成java的Date，即java的Date与数据库的varchar毫秒值之间转换。

```java
public class DateTypeHandler extends BaseTypeHandler<Date> {
    public void setNonNullParameter(PreparedStatement ps, int i, Date parameter, JdbcType jdbcType) throws SQLException {
        ps.setLong(i,parameter.getTime());
    }
    public Date getNullableResult(ResultSet rs, String columnName) throws SQLException {
        long aLong = rs.getLong(columnName);
         return new Date(aLong);
    }
    public Date getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        long aLong = rs.getLong(columnIndex);
         return new Date(aLong);     
    }
    public Date getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        long aLong = cs.getLong(columnIndex);
         return new Date(aLong);
    }
}

```

```xml
<!--注册类型自定义转换器-->
<typeHandlers>
    <typeHandler handler="com.itheima.typeHandlers.MyDateTypeHandler"></typeHandler>
</typeHandlers>

```

##### 3.2 分页插件PageHelper

MyBatis可以使用第三方的插件来对功能进行扩展，分页助手PageHelper是将分页的复杂操作进行封装，使用简单的方式即可获得分页的相关数据

开发步骤：

1. 导入通用PageHelper的坐标
2. 在mybatis核心配置文件中配置PageHelper插件
3. 测试分页数据获取

导入坐标

```xml
<!-- 分页助手 -->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>3.7.5</version>
</dependency>
<dependency>
    <groupId>com.github.jsqlparser</groupId>
    <artifactId>jsqlparser</artifactId>
    <version>0.9.1</version>
</dependency>

```

在mybatis核心配置文件中配置PageHelper插件

```xml
<!-- 注意：分页助手的插件  配置在通用馆mapper之前 -->
<plugin interceptor="com.github.pagehelper.PageHelper">
    <!-- 指定方言 -->
    <property name="dialect" value="mysql"/>
</plugin>

```

```java
@Test
public void test3() throws IOException {
    //设置起始页码和每页的显示条数
    PageHelper.startPage(2,3);   //public static Page startPage(int pageNum, int pageSize)
	//查询所有的数据
    //DEBUG findAll_PageHelper:159 - ==>  Preparing: select * from user limit ?,?   
    //DEBUG findAll_PageHelper:159 - ==> Parameters: 3(Integer), 3(Integer)
    List<User> list = mapper.findAll();
    for (User user : list) {
        System.out.println(user);
    }
    //传入list集合，构造页面对象
    PageInfo<User> info=new PageInfo<User>(list); // public PageInfo(List<T> list) 
    //
    System.out.println("当前页是否是最后一页"+info.isIsLastPage());
    System.out.println("总页数为："+info.getPages());
    System.out.println("总记录数为："+info.getTotal());
    System.out.println("当前页码为："+info.getPageNum());
    System.out.println("当前是否是第一页"+info.isIsFirstPage()); 
}

```

#### 4. 多表查询

注意：在数据库中，表与表之前的关系用外键表示，1  ------→  n ， 例如 order.uid=user.id

​	   在java中，对象间的关系，使用属性关联，和数据库不同，不是使用id关联，属性值指向一个完整的对象

##### 4.1 一对一

POJO

```java
public class Order {
    private int id;
    private Date ordertime;
    private double total;
    //代表当前订单从属于哪一个客户
    private User user;
}

public class User {
    private int id;
    private String username;
    private String password;
    private Date birthday;
}

```

Mapper.xml配置

```xml
<resultMap id="orderMap" type="order">
    <id column="oid" property="id"/>
    <result column="ordertime" property="ordertime"/>
    <result column="total" property="total"/>

    <!--
                一对一  一个订单对应一个用户
                property: order的属性名
                javaType: javabean 全类名
            -->
    <association property="user" javaType="user">
        <id column="uid" property="id"/>
        <result column="username" property="username"/>
        <result column="password" property="password"/>
        <result column="birthday" property="birthday"/>
    </association>
</resultMap>

<select id="findAll" resultMap="orderMap">
    SELECT * , o.`id` oid FROM `orders` o, `user` u  WHERE o.`uid`=u.`id`
</select>

```

##### 4.2 一对多

用户表和订单表的关系为，一个用户有多个订单，一个订单只从属于一个用户

一对多查询的需求：查询一个用户，与此同时查询出该用户具有的订单

```xml
<resultMap id="userMap" type="user">
    <id column="uid" property="id"/>
    <result column="username" property="username"/>
    <result column="password" property="password"/>
    <result column="birthday" property="birthday"/>

    <!--
            一个用户对应多个订单
        -->
    <collection property="orderList" ofType="order">
        <id column="oid" property="id"/>
        <result column="ordertime" property="ordertime"/>
        <result column="total" property="total"/>
    </collection>
</resultMap>

<select id="findAll" resultMap="userMap">
    SELECT * ,o.`id` oid FROM `user` u, orders o WHERE u.`id`=o.`uid`
</select>

```

##### 4.3 多对多

```xml
<resultMap id="userRoleMap" type="user">
    <id column="userid" property="id"/>
    <result column="username" property="username"/>
    <result column="password" property="password"/>
    <result column="birthday" property="birthday"/>

    <!--
            一个用户对应多个角色   有中间表
        -->
    <collection property="roleList" ofType="role">
        <id column="roleId" property="id"/>
        <result column="roleName" property="roleName"/>
        <result column="roleDesc" property="roleDesc"/>
    </collection>
</resultMap>

<select id="findUserAndRoleAll" resultMap="userRoleMap">
    SELECT * FROM USER , sys_user_role ur, sys_role r WHERE user.`id`=ur.`userId` AND ur.`roleId`=r.`id`
</select>


```

#### 5. 注解开发

##### 5.1 常用注解

@Insert：实现新增

@Update：实现更新

@Delete：实现删除

@Select：实现查询

@Result：实现结果集封装

@Results：可以与@Result 一起使用，封装多个结果集

@One：实现一对一结果集封装

@Many：实现一对多结果集封装

##### 5.2 MyBatis的核心配置文件（SqlMapConfig.xml）

```xml
<mappers>
    <!--扫描使用注解的类-->
   <!-- <mapper class="com.wyq.mapper.UserMapper"></mapper> -->
    <!--扫描使用注解的类所在的包-->
    <package name="com.wyq.mapper"></package>
</mappers>

```

##### 5.3 多表查询

1. 一对一

   ```java
   public interface OrderMapper {  
       @Select("select * from orders")
       @Results({
               @Result(id = true ,property = "id",column = "id"),
               @Result(property = "ordertime" ,column = "ordertime"),
               @Result(property = "total" ,column = "total"),
               @Result(
                       property = "user",
                       column = "uid",
                       javaType = User.class,
                       one = @One(select = "wyq.mapper.UserMapper.findById")
               )
       })
       public List<Order> findAll();
       @Select("select * from orders where uid=#{uid}")
       public List<Order> findByUid(int uid);
   }
   
   ```

2. 一对多

   ```java
   //一对多查询
   @Select("select * from user")
   @Results({
       @Result(id = true,property = "id",column = "id"),
       @Result(property = "username",column = "username"),
       @Result(property = "password",column = "password"),
       @Result(property = "birthday",column = "birthday"),
       @Result(
           	property = "orderList",
               column = "id",
               javaType = List.class,
               many = @Many(select = "wyq.mapper.OrderMapper.findByUid" )
              )
   })
   List<User> findUserAndOrderAll();
   
   ```

3. 多对多

   ```java
   //多对多查询
   @Select("select * from user")
   @Results({
       @Result(id = true,property = "id",column = "id"),
       @Result(property = "username",column = "username"),
       @Result(property = "password",column = "password"),
       @Result(property = "birthday",column = "birthday"),
       @Result(
           property = "roleList",
           column = "id",
           javaType = List.class,
           many = @Many(select = "wyq.mapper.RoleMapper.findByUid")
       )
   })
   List<User> findUserAndRoleAll();
   
   ```

4. 其他

   ```java
   @Insert("insert into user values (#{id},#{username},#{password},#{birthday})")
   void insert(User user);
   @Update("update user set username=#{username} , password=#{password}, birthday=#{birthday} where id =#{id}")
   void update(User user);
   @Delete("delete from user where id =#{id}")
   void delete(int i);
   @Select("select * from user where id =#{id}")
   User findById(int i);
   @Select("select * from user")
   List<User> findAll();
   
   ```

   