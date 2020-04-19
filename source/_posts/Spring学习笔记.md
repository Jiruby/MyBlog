---
title: Spring学习笔记
date: 2020-04-19 12:12:03
tags:	
	- Spring
categories: 学习笔记
---

## 一、Spring

### （一）Spring两大核心

1. #### IOC(Inverse Of Control:反转控制)

2. #### AOP(Aspect Oriented Programming:面向切面编程)

### （二）IOC详解

#### 1.创建UserDaoImpl实例

##### 1.1 无参构造方法

- applicationContext.xml配置文件（单例|初始化|销毁）**重点**

  默认是singleton

  ```xml
  <bean id="userDao" class="com.wyq.mapper.Impl.UserDaoImpl" scope="singleton" init-method="init" destroy-method="detory">
  ```

- UserDaoImpl类

  ```java
      public UserDaoImpl() {
          System.out.println("UserDaoImpl创建....");
      }
  
      public void init() {
          System.out.println("初始化方法.........");
      }
  
      public void detory() {
          System.out.println("销毁方法.........");
      }
  ```

##### 1.2 静态工厂

- xml配置

  ```xml
  <bean id="userDao" class="com.wyq.factory.StaticFactory" factory-method="getUserDao">
  ```

- StaticFactory类

  ```java
  public class StaticFactory {
      public static UserDao getUserDao(){
          return  new UserDaoImpl();
      }
  }
  ```

##### 1.3 动态工厂

- xml配置

  ```xml
   <bean id="factory" class="com.wyq.factory.DynamicFactory"></bean>
   <bean id="userDao" factory-bean="factory" factory-method="getUserDao"></bean>
  ```

- DynamicFactory类

  ```java
  public class DynamicFactory {
      public UserDao getUserDao(){
          return  new UserDaoImpl();
      }
  }
  ```

##### 1.4 小结

1. 动态工厂与静态工厂的区别

   由于动态工厂的方法调用前提：一个自身的实体对象，所以xml配置文件比静态工厂多一个bean配置 

2. scope：

   | 取值范围       | 说明                                                         |
   | :------------- | :----------------------------------------------------------- |
   | singleton      | 默认，单例                                                   |
   | prototype      | 多例                                                         |
   | request        | WEB   项目中，Spring   创建一个   Bean   的对象，将对象存入到   request   域中 |
   | session        | WEB   项目中，Spring   创建一个   Bean   的对象，将对象存入到   session   域中 |
   | global session | WEB   项目中，应用在   Portlet   环境，如果没有   Portlet   环境那么globalSession   相当于   session |

3. singleton与prototype的区别

   |            | singleton                        | prototype                  |
   | ---------- | -------------------------------- | -------------------------- |
   | 实例个数   | 1                                | 多个                       |
   | 实例化时机 | 当spring核心配置文件被加载时创建 | 当调用getBean()方法时创建  |
   | 对象的运行 | 容器存在，对象存在               | 对象在使用，对象存在       |
   | 对象的销毁 | 容器销毁，对象销毁               | 长时间不用，被java垃圾回收 |

#### 2.依赖注入

##### 2.1 概念

依赖注入（Dependency Injection）它是 Spring 框架核心 IOC 的具体实现。

在编程的过程中通过IOC，将对象的创建交给Spring，降低了代码间的耦合关系，但是没有消除

例如：Service 层调用 Dao层 的方法

业务层与持久层间的依赖关系，通过配置，也让Spring来管理。

##### 2.2 Bean的依赖注入方式

- 有参构造注入（了解）

- set方法注入

  实现类

  ```java
  public class UserServiceImpl implements UserService {
      private UserDao userDao;
      public void setUserDao(UserDao userDao) {
          this.userDao = userDao;  
          } 
      @Override    
      public void save() {      
     		 userDao.save();
  	}
  }
  ```

  xml配置

  注意：此处name 对应的是属性值，对应的是 setUserDao  ->  UserDao  ->  userDao

  ```xml
  <bean id="userDao" class="com.wyq.dao.impl.UserDaoImpl"/>
  <bean id="userService" class="com.wyq.service.impl.UserServiceImpl">
  	<property name="userDao" ref="userDao"/>
  </bean>
  ```

  也可以使用p命名空间配置xml（要先引入p命名空间）

  ```xml
  <bean id="userService" class="com.wyq.service.impl.UserServiceImpl" p:userDao-
   ref="userDao"/>
  
  ```

- 注入普通的属性

  ```xml
   <bean id="userDao" class="com.wyq.dao.Impl.UserDaoImpl">
          <!--集合数据类型（List<User>）的注入-->
          <property name="strList">
              <list>
                  <value>aaa</value>
                  <value>bbb</value>
                  <value>ccc</value>
              </list>
          </property>
          <!--集合数据类型（ Map<String,User> ）的注入-->
          <property name="map">
              <map>
                  <entry key="u1" value-ref="user1"/>
                  <entry key="u2" value-ref="user2"/>
              </map>
          </property>
          <!--集合数据类型（Properties）的注入-->
          <property name="pros">
              <props>
                  <prop key="p1">ppp1</prop>
                  <prop key="p2">ppp2</prop>
                  <prop key="p3">ppp3</prop>
              </props>
          </property>
      </bean>
      <!--准备注入对象-->
      <bean id="user1" class="com.wyq.domain.User">
          <property name="name" value="张三"/>
          <property name="age" value="18"/>
      </bean>
      <bean id="user2" class="com.wyq.domain.User">
          <property name="name" value="李四"/>
          <property name="age" value="20"/>
      </bean>
  
  ```

#### 3.注解开发

##### 3.1 原始注解

Spring原始注解主要是替代<Bean>的配置

| 注解           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| @Component     | 使用在类上，用于实例化Bean                                   |
| @Controller    | web层（见名知意）                                            |
| @Service       | Service层                                                    |
| @Repository    | Dao层                                                        |
| @Autowired     | 用于类属性上的（类型）依赖注入，使用后可以省略setXxx()方法   |
| @Qualifier     | 结合@Autowired使用，根据唯一id（名称）注入                   |
| @Resource      | 相当于@Autowired+@Qualifier                                  |
| @Value         | 普通属性注入，可结合 <context:property-placeholder location="classpath:"/> 解耦合 |
| @Scope         | 标注Bean的作用范围                                           |
| @PostConstruct | 使用在方法上标注该方法是Bean的初始化方法                     |
| @PreDestroy    | 使用在方法上标注该方法是Bean的销毁方法                       |

注意：

使用注解进行开发时，需要在applicationContext.xml中配置组件扫描，作用是指定哪个包及其子包下的Bean需要进行扫描以便识别使用注解配置的类、字段和方法。

```xml
<context:component-scan base-package="包名"></context:component-scan>

```

##### 3.2 新注解

| 注解            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| @Configuration  | 标记Spring配置类                                             |
| @ComponentScan  | 写在Spring配置类上，指定初始化Spring容器时需要扫描的包，作用和<context:component-scan base-package="包名"/> 一样 |
| @Bean           | 使用在方法上，标注将该方法的返回值存储到   Spring   容器中   |
| @PropertySource | 用于加载.properties   文件中的配置                           |
| @Import         | 用于导入其他配置类                                           |

##### 3.3 演示

```java
@Configuration
@ComponentScan("com.wyq")
@Import({DataSourceConfig.class})
public class SpringConfiguration {
}

@PropertySource("classpath:jdbc.properties") // 加载Properties
public class DataSourceConfig {
    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String username;
    @Value("${jdbc.password}")
    private String password;

    @Bean(name="dataSource")   //将该方法返回的对象存入 Spring 容器
    public DataSource getDataSource(){
        DruidDataSource dataSource=new DruidDataSource();
        dataSource.setDriverClassName(driver);
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        return dataSource;
    }
}

```

#### 4.Spring配置数据源

##### 4.1 数据源（连接池）的作用

数据源(连接池)可以提高程序性能，使用连接资源时从数据源中获取，使用完毕后将连接资源归还给数据源

常见的数据源(连接池)：DBCP、C3P0、BoneCP、Druid等

##### 4.2 步骤

1. 导入连接池坐标

2. 导入数据库驱动

3. 准备xx.properties配置文件

4. 加载xx.properties配置文件

5. 抽取配置文件属性到Bean交给Spring管理

   ```xml
   <context:property-placeholder location="classpath:jdbc.properties"/>
   <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
       <property name="driverClass" value="${jdbc.driver}"/>
       <property name="jdbcUrl" value="${jdbc.url}"/>
       <property name="user" value="${jdbc.username}"/>
       <property name="password" value="${jdbc.password}"/>
   </bean>
   
   ```

#### 5.Spring集成Junit

##### 5.1 集成的作用

简化代码，让Spring容器去生成需要测试的Bean

##### 5.2 步骤

1. 导入Spring集成Junit的坐标

   ```xml
   <!--此处需要注意的是，spring5 及以上版本要求 junit 的版本必须是 4.12 及以上-->
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-test</artifactId>
       <version>5.0.2.RELEASE</version>
   </dependency>
   <dependency>
       <groupId>junit</groupId>
       <artifactId>junit</artifactId>
       <version>4.12</version>
       <scope>test</scope>
   </dependency>
   
   ```

2. 使用@Runwith注解替换原来的运行期

3. 使用@ContextConfiguration指定配置文件或配置类

4. 使用@Autowired注入需要测试的对象

   **注意：只要是容器中的对象都可以拿过来测试**

5. 创建测试方法进行测试

   ```java
   @RunWith(SpringJUnit4ClassRunner.class)
   @ContextConfiguration("classpath:applicationContext.xml")
   public class SpringJdbcCRUDTest {
       @Autowired
       private JdbcTemplate jdbcTemplate;
   
       @Test
       public void testQueryCount(){
           Long num = jdbcTemplate.queryForObject("select count(*) from account", Long.class);
           System.out.println("num = " + num);
       }
   }
   
   ```

   

### （三）AOP详解

#### 1.什么是AOP

AOP 为 Aspect Oriented Programming 的缩写，意思为**面向切面编程**，是通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。 

AOP 是 OOP（*面向对象编程*） 的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。**利用AOP可以对业务逻辑的各个部分进行隔离**，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

- 作用：在程序运行期间，在不修改源码的情况下对方法进行功能增强  
- 优势：减少重复代码，提高开发效率，并且便于维护 

#### 2.动态代理方式

- JDK代理：基于j接口
- cglib代理：基于父类

![1564976085648](C:\Users\29789\AppData\Local\Temp\1564976085648.png)

##### 2.1 JDK代理

1. 准备TargetInterface接口和Target类

2. 准备Advice通知类

3. 动态代理代码

   ```java
   public class ProxyTest {
       public static void main(String[] args) {
           final Target target=new Target();
           final Advice advice=new Advice();
           // target 与 proxy 是兄弟关系 所以要用接口接收
           TargetInterface proxy = (TargetInterface) Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), new InvocationHandler() {
               public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                   advice.before();
                   Object invoke = method.invoke(target, args);
                   advice.after();
                   return invoke;
               }
           });
           //测试
           proxy.show();
       }
   }
   
   ```

Ps：在web阶段时，filter用于敏感词过滤有使用到JDK的动态代理

##### 2.2 cglib代理

1. 准备Target类

2. 准备Advice通知类

3. 动态代理代码

   ```java
   public class ProxyTest {
       public static void main(String[] args) {
           //1. 创建目标对象
           final Target target = new Target();
           final Advice advice = new Advice();
           //2. 创建加强器
           Enhancer enhancer = new Enhancer();
           //3. 设置父类
           enhancer.setSuperclass(target.getClass());
           //4. 设置回调函数
           enhancer.setCallback(new MethodInterceptor() {
               public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                   advice.before();
                   Object invoke = method.invoke(target, objects);
                   advice.after();
                   return invoke;
               }
           });
           // 代理对象 与  目标对象是 父子关系
           //5. 获取代理对象
           Target proxy = (Target) enhancer.create();
           proxy.show();
       }
   }
   
   ```

Ps：dubbo中使用事务管理不能正确地发布服务，要指定使用cglib来解决

#### 3.AOP实现步骤

##### 3.1 重点概念

- Pointcut（切入点）：被增强的方法
- Advice（通知/ 增强）：封装增强业务逻辑的方法
- Aspect（切面）：切点+通知
- Weaving（织入）：将切点与通知结合的过程

##### 3.2 通知类型

| 名称         | 标签                    | 说明                             |
| ------------ | ----------------------- | -------------------------------- |
| 前置通知     | <aop:before >           | 切点方法之前执行                 |
| 后置通知     | <aop:after-returnning > | 切点方法之后执行                 |
| 环绕通知     | <aop:around >           | 切点方法前后都执行               |
| 异常抛出通知 | <aop:throwing >         | 切点方法抛出异常时执行           |
| 最终通知     | <aop:after >            | 无论切点方法是否正常，执行都执行 |

##### 3.3 基于xml的AOP

1. 导入 AOP 相关坐标

   ```xml
   <!--导入spring的context坐标，context依赖aop-->
   <dependency>
     <groupId>org.springframework</groupId>
     <artifactId>spring-context</artifactId>
     <version>5.0.5.RELEASE</version>
   </dependency>
   <!-- aspectj的织入 -->
   <dependency>
     <groupId>org.aspectj</groupId>
     <artifactId>aspectjweaver</artifactId>
     <version>1.8.13</version>
   </dependency>
   
   ```

2. 创建目标接口和目标类（内部有切点）

3. 创建切面类（内部有增强方法）

   ```java
   public class MyAspect {
       public void before(){
           System.out.println("前置通知.........");
       }
   
       public void afterReturning(){
           System.out.println("后置通知...........");
       }
   
       public Object around(ProceedingJoinPoint pjp) throws Throwable {
           System.out.println("前环绕通知...........");
           Object proceed = pjp.proceed();
           System.out.println("后环绕通知...........");
           return proceed;
       }
   
       public void throwing(){
           System.out.println("异常抛出通知...........");
       }
   
       public void after(){
           System.out.println("最终通知.........");
       }
   }
   
   ```

4. 将目标类和切面类的对象创建权交给 spring

5. 在 applicationContext.xml 中配置织入关系

   ```xml
   <!--目标对象-->
   <bean id="target" class="cn.wyq.proxy.aop.Target"/>
   <!--切面对象(声明后生效)-->
   <bean id="myAspect" class="cn.wyq.proxy.aop.MyAspect"/>
   <!--配置织入-->
   <aop:config>
       <!--声明切面对象-->
       <aop:aspect ref="myAspect">
           <!--切点表达式抽取-->
           <aop:pointcut id="myPointcut" expression="execution( * cn.wyq.proxy.aop.*.*(..))"/>
           <!--切面: 通知 + 切点 -->
           <!--<aop:after method="after" pointcut="execution( * cn.wyq.proxy.aop.*.*(..))"/>-->
           <aop:around method="around" pointcut-ref="myPointcut"/>
           <aop:after method="after" pointcut-ref="myPointcut"/>
       </aop:aspect>
   </aop:config>
   
   ```

6. 测试

##### 3.4 基于注解的AOP

1. 导入 AOP 相关坐标

2. 创建目标接口和目标类（内部有切点）

   ```java
   @Component
   public class Target implements TargetInterface {
       public void method() {
           System.out.println("this is show!!");
       }
   }
   
   ```

3. 创建切面类（内部有增强方法）

   ```java
   @Component
   @Aspect
   public class MyAspect {
       //切点表达式抽取
       @Pointcut("execution( * cn.wyq.proxy.anno.Target.*(..))")
       public void myPoint() {
       }
   
       @Before("myPoint()")
       public void before() {
           System.out.println("前置通知.........");
   
       }
       @AfterReturning("myPoint()")
       public void afterReturning(){
           System.out.println("后置通知...........");
       }
   
       @Around("myPoint()")
       public Object around(ProceedingJoinPoint pjp) throws Throwable {
           System.out.println("前环绕通知...........");
           Object proceed = pjp.proceed();
           System.out.println("后环绕通知...........");
           return proceed;
       }
       @AfterThrowing("myPoint()")
       public void throwing(){
           System.out.println("异常抛出通知...........");
       }
   
       @After("myPoint()")
       public void after(){
           System.out.println("最终通知.........");
       }
   }
   
   ```

4. 设置主配置文件

   ```java
   //@ContextConfiguration("classpath:applicationContext.xml")  设置主加载配置类
   @Configuration
   //<context:component-scan base-package="cn.wyq"/>   设置扫描路径
   @ComponentScan("cn.wyq")
   //<aop:aspectj-autoproxy></aop:aspectj-autoproxy>  开启自动代理
   @EnableAspectJAutoProxy
   public class MyConfig {
   }
   
   ```

5. 测试

   ```java
   @RunWith(SpringJUnit4ClassRunner.class)
   @ContextConfiguration(classes = MyConfig.class)
   public class AnnoTest {
   
       @Autowired
       private TargetInterface target;
   
       @Test
       public void show1(){
           target.method();
       }
   }
   
   ```

#### 4. 小结

- aop有什么作用：在不改源码的情况下增强代码
- aop的底层原理：动态代理
- jdk和cglib动态搭理的区别： jdk是基于接口，cglib基于继承
- 切面指的是： 通知+切点
- 切点表达式的写法  execution([修饰符] 返回值类型 包名.类名.方法名(参数）
- 定义切面的注解是： @Aspect  
- 定义前置通知注解的是： @Before

### （四）JdbcTemplate与Spring事务管理

#### 1. JdbcTemplate

##### 1.1 概念

它是spring框架中提供的一个对象，是对原始繁琐的Jdbc API对象的简单封装。

Spring框架为我们提供了很多的操作模板类。例如：操作关系型数据的JdbcTemplate和HibernateTemplate，操作nosql数据库的RedisTemplate，操作消息队列的JmsTemplate等等。

##### 1.2 入门步骤

1. 导入spring-jdbc和spring-tx依赖，但同时不能忘记导入mysql驱动，数据库连接池的依赖。

2. 创建表和实体，实体的字段属性需要get/set方法，同时添加上toString方法 

3. 创建JdbcTemplate对象，然后设置数据源，因此需要提前创建数据源，同时设置好数据库的相关信息

   ```xml
   <!-- 将数据库的连接信息配置抽取成外部的属性配置文件，跟spring的配置文件分开，有利于后期的维护 -->
   <context:property-placeholder location="classpath:jdbc.properties"/>
   
   <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
       <property name="driverClassName" value="${jdbc.driver}"></property>
       <property name="url" value="${jdbc.url}"></property>
       <property name="username" value="${jdbc.username}"></property>
       <property name="password" value="${jdbc.password}"></property>
   </bean>
   
   <bean id="jdbctmp" class="org.springframework.jdbc.core.JdbcTemplate">
       <property name="dataSource" ref="dataSource"></property>
   </bean>
   
   ```

4. 调用JdbcTempate的update方法传入sql语句和参数完成插入操作 

#### 2.事务管理

##### 2.1 编程式事务控制相关对象

1. PlatformTransactionManager 平台事务管理器

   PlatformTransactionManager 是接口类型，**不同的 Dao 层技术则有不同的实现类**。

   - Dao 层技术是jdbc|mybatis ：org.springframework.jdbc.datasource.DataSourceTransactionManager
   - Dao 层技术是hibernate时：org.springframework.orm.hibernate5.HibernateTransactionManager

2. TransactionDefinition 事务定义

   - 事务隔离级别

     | 名称                       | 描述                           |
     | -------------------------- | ------------------------------ |
     | ISOLATION_DEFAULT          | 和数据库默认的隔离级别保持一致 |
     | ISOLATION_READ_UNCOMMITTED | 读未提交                       |
     | ISOLATION_READ_COMMITTED   | 读已提交（可解决脏读）         |
     | ISOLATION_REPEATABLE_READ  | 可以重复读（可解决不可重复读） |
     | ISOLATION_SERIALIZABLE     | 串行化（可以解决虚读）效率低   |

   - 事务传播行为

     REQUIRED：如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。一般的选择（默认值）

     SUPPORTS：支持当前事务，如果当前没有事务，就以非事务方式执行（没有事务）

     MANDATORY：使用当前的事务，如果当前没有事务，就抛出异常

     REQUERS_NEW：新建事务，如果当前在事务中，把当前事务挂起。

     NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起

     NEVER：以非事务方式运行，如果当前存在事务，抛出异常

     NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行 REQUIRED 类似的操作

     超时时间：默认值是-1，没有超时限制。如果有，以秒为单位进行设置

     是否只读：建议查询时设置为只读

3. TransacionStatus  事务状态

##### 2.2 声明式事务管理

1. 定义
   在配置文件中声明，用在 Spring 配置文件中声明式的处理事务来代替代码式的处理事务。 

   注意：**Spring 声明式事务控制底层就是AOP**。 

2. 基于xml配置实现步骤

   - 引入tx 命名空间

   - 配置事务增强

     ```xml
     <!--平台事务管理器-->
     <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
         <property name="dataSource" ref="dataSource"></property>
     </bean>
     
     <!--事务通知-->
     <tx:advice id="txAdvice" transaction-manager="transactionManager">
         <!--设置事务的属性信息-->
         <tx:attributes>
              <!--name：切点的方法名称-->
             <tx:method name="*"/>
         </tx:attributes>
     </tx:advice>
     
     ```

   - 配置事务AOP织入

     ```xml
     <aop:config>
         <!--事务通知+切点表达式-->
         <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.wyq.service.impl.*.*(..))"></aop:advisor>
     </aop:config>
     
     ```

   - 测试

3. 基于注解实现步骤

   要点：	平台事务管理器配置（xml方式）

   ​		事务通知的配置（@Transactional注解配置）

   ​		事务注解驱动的配置 <tx:annotation-driven/ >

- 编写被事务管理的类

  **事务定义采用就近原则**

  ```java
  @Service("accountService")
  @Transactional(isolation = Isolation.DEFAULT)
  public class AccountServiceImpl implements AccountService {
  
      @Autowired
      private AccountDao accountDao;
      
      @Transactional(isolation = Isolation.REPEATABLE_READ ,propagation = Propagation.REQUIRED, timeout = -1, readOnly = false)
      public void transfer(String outMan, String inMan, double money) {
          accountDao.out(outMan,money);
          int i=1/0;
          accountDao.in(inMan,money);
      }
  }
  
  ```

- xml配置

  ```xml
  <!--平台事务管理器-->
  <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
      <property name="dataSource" ref="dataSource"></property>
  </bean>
  
  <!--声明注解驱动 设置事务管理器-->
  <tx:annotation-driven transaction-manager="transactionManager"/>
  
  ```

### （五）Spring与Web集成

#### 1. ApplicationContext应用上下文获取方式

应用上下文对象通过new ClasspathXmlApplicationContext(spring配置文件) 获取，每次从Spring容器获取Bean对象都需要先获取该对象，这样的弊端是配置文件加载多次，应用上下文对象创建多次。

在Web项目中，可以使用ServletContextListener监听Web应用的启动，我们可以在Web应用启动时，就加载Spring的配置文件，创建应用上下文对象ApplicationContext，在将其存储到最大的域servletContext域中，这样就可以在任意位置从域中获得应用上下文ApplicationContext对象了。

#### 2. Spring提供获取应用上下文的工具

Spring提供了一个监听器**ContextLoaderListener**就是对上述功能的封装，该监听器内部加载Spring配置文件，创建应用上下文对象，并存储到ServletContext域中，提供了一个客户端工具WebApplicationContextUtils供使用者获得应用上下文对象。

#### 3.步骤

##### 3.1 导入Spring集成web坐标

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.0.5.RELEASE</version>
</dependency>

```

##### 3.2 配置ContextLoaderListener监听器

```xml
<!--全局参数-->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
</context-param>
<!--Spring的监听器-->
<listener>
	<listener-class>
       org.springframework.web.context.ContextLoaderListener
   </listener-class>
 </listener>

```

##### 3.3 通过工具获得应用上下文对象 （后期不再使用）

```java
ApplicationContext applicationContext =    
    WebApplicationContextUtils.getWebApplicationContext(servletContext);
    Object obj = applicationContext.getBean("id");

```

## 