# seckill

## 项目介绍

> 何为秒杀？

所谓“秒杀”，就是网络卖家发布一些超低价格的商品，所有买家在同一时间网上抢购的一种销售方式。由于商品价格低廉，往往一上架就被抢购一空，有时只用一秒钟。

> 为何选择Java高并发秒杀作为实战项目？

- 秒杀业务场景具有典型事务特性
- 秒杀/红包类需求越来越常见

> 为何使用SpringMVC+Spring+MyBatis框架

- 框架易于使用和轻量级
- 低代码侵入性
- 成熟的社区和用户群

> 能从该项目得到什么收获？

- 框架的使用和整合技巧
- 秒杀分析过程与优化思路

> How to play

- 将下载的源码解压后作为Maven项目导入到IDE工具中；或者将从GitHub克隆下来的项目作为Maven项目导入到IDE工具中
- 打开项目中的jdbc.properties文件，修改里边的url,username和password
- 将项目部署到Tomcat上并启动
  - 可以直接用IDE内嵌的Tomcat启动项目
  - 或者将本项目通过**mvn clean package**命令打成war包并丢到本地安装的Tomcat的webapps目录下，接着启动Tomcat即可
- 在浏览器上访问：`http://localhost:8080/seckill`

## 开发环境

- **操作系统**：Windows 10
- **IDE工具**：Eclipse
- **JDK**：JDK1.7
- **中间件**：Tomcat 7.0
- **数据库**：MySQL 5.0
- **构建工具**：Maven
- **框架**：SSM

## 项目总结

### 数据层技术

- 数据库的设计和实现

  1. 创建数据库

      源码里有个sql文件夹，可以给出了sql语句；也可以选择自己手写。数据库一共就两个表：秒杀库存表、秒杀成功明细表。

     ```mysql
     -- 数据库初始化脚本
     
     -- 创建数据库
     CREATE DATABASE seckill;
     -- 使用数据库
     use seckill;
     CREATE TABLE seckill(
       `seckill_id` BIGINT NOT NUll AUTO_INCREMENT COMMENT '商品库存ID',
       `name` VARCHAR(120) NOT NULL COMMENT '商品名称',
       `number` int NOT NULL COMMENT '库存数量',
       `create_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
       `start_time` TIMESTAMP  NOT NULL COMMENT '秒杀开始时间',
       `end_time`   TIMESTAMP   NOT NULL COMMENT '秒杀结束时间',
       PRIMARY KEY (seckill_id),
       key idx_start_time(start_time),
       key idx_end_time(end_time),
       key idx_create_time(create_time)
     )ENGINE=INNODB AUTO_INCREMENT=1000 DEFAULT CHARSET=utf8 COMMENT='秒杀库存表';
     
     -- 初始化数据
     INSERT into seckill(name,number,start_time,end_time)
     VALUES
       ('8000元秒杀iphone XS MAX',100,'2019-01-01 00:00:00','2019-01-02 00:00:00'),
       ('1000元秒杀ipad',200,'2019-01-01 00:00:00','2019-01-02 00:00:00'),
       ('6600元秒杀mac pro',300,'2019-01-01 00:00:00','2019-01-02 00:00:00'),
       ('7000元秒杀iMac',400,'2019-01-01 00:00:00','2019-01-02 00:00:00');
     
     -- 秒杀成功明细表
     -- 用户登录认证相关信息(简化为手机号)
     CREATE TABLE success_killed(
       `seckill_id` BIGINT NOT NULL COMMENT '秒杀商品ID',
       `user_phone` BIGINT NOT NULL COMMENT '用户手机号',
       `state` TINYINT NOT NULL DEFAULT -1 COMMENT '状态标识:-1:无效 0:成功 1:已付款 2:已发货',
       `create_time` TIMESTAMP NOT NULL COMMENT '创建时间',
       PRIMARY KEY(seckill_id,user_phone),/*联合主键*/
       KEY idx_create_time(create_time)
     )ENGINE=INNODB DEFAULT CHARSET=utf8 COMMENT='秒杀成功明细表';
     
       -- SHOW CREATE TABLE seckill;#显示表的创建信息
     
     ```
  2. 创建数据表对应的实体类

​     seckill实体类

```java
public class Seckill {
    private Long seckillId;

    private String name;

    private Integer number;

    private Date createTime;

    private Date startTime;

    private Date endTime;

    public Long getSeckillId() {
        return seckillId;
    }

    public void setSeckillId(Long seckillId) {
        this.seckillId = seckillId;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name == null ? null : name.trim();
    }

    public Integer getNumber() {
        return number;
    }

    public void setNumber(Integer number) {
        this.number = number;
    }

    public Date getCreateTime() {
        return createTime;
    }

    public void setCreateTime(Date createTime) {
        this.createTime = createTime;
    }

    public Date getStartTime() {
        return startTime;
    }

    public void setStartTime(Date startTime) {
        this.startTime = startTime;
    }

    public Date getEndTime() {
        return endTime;
    }

    public void setEndTime(Date endTime) {
        this.endTime = endTime;
    }

    @Override
    public String toString() {
        return "Seckill [seckillId=" + seckillId + ", name=" + name + ", number=" + number + ", createTime=" + createTime + ", startTime="
                + startTime + ", endTime=" + endTime + "]";
    }
}
```

SuccessKilled实体类

```java
public class SuccessKilled {
    private Byte state;

    private Date createTime;

    private Long seckillId;

    private Long userPhone;

    // 多对一,因为一件商品在库存中有很多数量，对应的购买明细也有很多。
    private Seckill seckill;

    public Seckill getSeckill() {
        return seckill;
    }

    public void setSeckill(Seckill seckill) {
        this.seckill = seckill;
    }

    public Long getSeckillId() {
        return seckillId;
    }

    public void setSeckillId(Long seckillId) {
        this.seckillId = seckillId;
    }

    public Long getUserPhone() {
        return userPhone;
    }

    public void setUserPhone(Long userPhone) {
        this.userPhone = userPhone;
    }

    public Byte getState() {
        return state;
    }

    public void setState(Byte state) {
        this.state = state;
    }

    public Date getCreateTime() {
        return createTime;
    }

    public void setCreateTime(Date createTime) {
        this.createTime = createTime;
    }

    @Override
    public String toString() {
        return "SuccessKilled [state=" + state + ", createTime=" + createTime + ", seckillId=" + seckillId
                + ", userPhone=" + userPhone + "]";
    }

}
```

- mybatis理解和使用技巧

> **MyBatis怎么用？SQL写在哪里？**

Mybatis有两种提供SQL的方式：XML提供SQL、注解提供SQL（注解是java5.0之后提供的一个新特性）。

> **如何去实现DAO接口？**

Mapper自动实现DAO（也就是DAO只需要设计接口，不需要去写实现类，MyBatis知道我们的参数、返回类型是什么，同时也有SQL文件，它可以自动帮我们生成接口的实现类来帮我们执行参数的封装，执行SQL，把我们的返回结果集封装成我们想要的类型） 。

第二种是通过API编程方式实现DAO接口（MyBatis通过给我们提供了非常多的API，跟其他的ORM和JDBC很像）。

在实际开发中建议使用Mapper自动实现DAO，这样可以直接只关注SQL如何编写，如何去设计DAO接口，帮我们节省了很多的维护程序，所有的实现都是MyBatis自动完成。

> **创建一个目录存放Mybatis的SQL映射**

按照Maven的规范，SQL映射文件应该放在src/main/resources包下，在该包下建立mapper目录，用来存放映射DAO接口的XML文件。这样Maven在编译时就会自动将src/main/resources下的这些配置文件编译进来。

我们也可以按照原本的习惯，在src/main/java下建立com.lewis.mapper包，将这些SQL映射存放到这里。由于Maven默认不会编译src/main/java下除源码以外的文件，所以需要在pom.xml中进行额外的配置。

```xml
<build>
    <finalName>seckill</finalName>
    <resources>
        <!--打包时包含源代码包下的资源文件，默认情况下只会打包src/main/java下的源代码 -->
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
        </resource>
    </resources>
</build>
```

在本项目中，我是采用的第二种方式存放Mybatis的SQL映射。（只是将映射DAO的mapper文件放在java包下，其他的关于Spring、MyBatis等的配置文件还是放在resources包下）。

> **在src/main/resources目录下配置mybatis-config.xml（配置MyBatis的全局属性）**

打开MyBatis的[官方文档](http://www.mybatis.org/mybatis-3/zh/index.html)（MyBatis的官方文档做的非常友好，提供了非常多版本的国际化支持），选择` 入门`，找到MyBatis全局配置，里面有XML的规范（XML的标签约束dtd文件），拷入到项目的MyBatis全局配置文件中，开始配置MyBatis，如下：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--配置全局属性 -->
    <settings>
        <!--使用jdbc的getGeneratekeys获取自增主键值，默认是false
            当inert一条记录时我们是不插入id的，id是通过自增去赋值的
            当插入完后想得到该插入记录的id时可以调用jdbc的getGeneratekeys -->
        <setting name="useGeneratedKeys" value="true" />

        <!--使用列别名替换列名 默认值为true（可以不用写出来，这里写出来只是为了讲解该配置的作用）
            select name as title(实体中的属性名是title) form table; 
            开启后mybatis会自动帮我们把表中name的值赋到对应实体的title属性中 -->
        <setting name="useColumnLabel" value="true" />

        <!--开启驼峰命名转换Table:create_time到 Entity(createTime) -->
        <setting name="mapUnderscoreToCamelCase" value="true" />
    </settings>

</configuration>
```

> **在src/main/java目录下的com.lewis.mapper包里创建SeckillDao.xml**

```xml
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace:指定为哪个接口提供配置 --> 
<mapper namespace="com.lewis.dao.SeckillDao">
    <!--目的:为dao接口方法提供sql语句配置， 即针对dao接口中的方法编写我们的sql语句 -->

    <!-- int reduceNumber(long seckillId, Date killTime);-->
    <!-- 这里id必须和对应的DAO接口的方法名一样 -->
    <update id="reduceNumber">
        UPDATE seckill
        SET number = number-1
        WHERE seckill_id=#{seckillId}
        AND start_time <![CDATA[ <= ]]>
        #{killTime}
        AND end_time >= #{killTime}
        AND number > 0;
    </update>

     <!-- parameterType:使用到的参数类型
        正常情况java表示一个类型的包名+类名，这直接写类名，因为后面有一个配置可以简化写包名的过程 -->
    <select id="queryById" resultType="Seckill" parameterType="long">
        <!-- 可以通过别名的方式列明到java名的转换，如果开启了驼峰命名法就可以不用这么写了 
             select seckill_id as seckillId
        -->
        SELECT seckill_id,name,number,create_time,start_time,end_time
        FROM seckill
        WHERE seckill_id=#{seckillId}
    </select>

    <select id="queryAll" resultType="Seckill">
        SELECT *
        FROM seckill
        ORDER BY create_time DESC
        limit #{offset},#{limit}
    </select>

</mapper>
```

> **在src/main/java目录下的com.lewis.mapper包里创建SuccessKilledDao.xml**

```xml
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.lewis.dao.SuccessKilledDao">

    <insert id="insertSuccessKilled">
        <!--当出现主键冲突时(即重复秒杀时)，会报错;不想让程序报错，加入ignore-->
        INSERT ignore INTO success_killed(seckill_id,user_phone,state)
        VALUES (#{seckillId},#{userPhone},0)
    </insert>

    <select id="queryByIdWithSeckill" resultType="SuccessKilled">

        <!--根据seckillId查询SuccessKilled对象，并携带Seckill对象-->
        <!--如何告诉mybatis把结果映射到SuccessKill属性同时映射到Seckill属性-->
        <!--可以自由控制SQL语句-->

        SELECT
            sk.seckill_id,
            sk.user_phone,
            sk.create_time,
            sk.state,
            s.seckill_id "seckill.seckill_id",
            s.name "seckill.name",
            s.number "seckill.number",
            s.start_time "seckill.start_time",
            s.end_time "seckill.end_time",
            s.create_time "seckill.create_time"
        FROM success_killed sk
        INNER JOIN seckill s ON sk.seckill_id=s.seckill_id
        WHERE sk.seckill_id=#{seckillId} and sk.user_phone=#{userPhone}
    </select>

</mapper>
```

- mybatis整合spring技巧

在resources目录下创建一个新的目录spring(存放所有Spring相关的配置)

> **在resources包下创建jdbc.properties，用于配置数据库的连接信息**

```
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/seckill?useUnicode=true&characterEncoding=utf-8
jdbc.username=root
password=123
```

> **在resources/spring目录下创建Spring关于DAO层的配置文件spring-dao.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!--配置整合mybatis过程
    1.配置数据库相关参数-->
    <context:property-placeholder location="classpath:jdbc.properties"/>

    <!--2.数据库连接池-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <!--配置连接池属性-->
        <property name="driverClass" value="${driver}" />

        <!-- 基本属性 url、user、password -->
        <property name="jdbcUrl" value="${url}" />
        <property name="user" value="${jdbc.username}" />
        <property name="password" value="${password}" />

        <!--c3p0私有属性-->
        <property name="maxPoolSize" value="30"/>
        <property name="minPoolSize" value="10"/>
        <!--关闭连接后不自动commit-->
        <property name="autoCommitOnClose" value="false"/>

        <!--获取连接超时时间-->
        <property name="checkoutTimeout" value="1000"/>
        <!--当获取连接失败重试次数-->
        <property name="acquireRetryAttempts" value="2"/>
    </bean>

    <!--约定大于配置-->
    <!--３.配置SqlSessionFactory对象-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--往下才是mybatis和spring真正整合的配置-->
        <!--注入数据库连接池-->
        <property name="dataSource" ref="dataSource"/>
        <!--配置mybatis全局配置文件:mybatis-config.xml-->
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
        <!--扫描entity包,使用别名,多个用;隔开-->
        <property name="typeAliasesPackage" value="com.lewis.entity"/>
        <!--扫描sql配置文件:mapper需要的xml文件-->
        <property name="mapperLocations" value="classpath:com/lewis/mapper/*.xml"/>
    </bean>

    <!--４:配置扫描Dao接口包,动态实现DAO接口,注入到spring容器-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!--注入SqlSessionFactory-->
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        <!-- 给出需要扫描的Dao接口-->
        <property name="basePackage" value="com.lewis.dao"/>
    </bean>

    <!--redisDao-->
   <!--  <bean id="redisDao" class="com.lewis.dao.cache.RedisDao">
        <constructor-arg index="0" value="localhost"/>
        <constructor-arg index="1" value="6379"/>
    </bean> -->
</beans>
```

### 业务层技术

- 业务接口设计和封装（站在使用者的角度）

> **业务接口的编写**

初学者总是关注细节，关注接口如何去实现，这样设计出来的接口往往比较冗余。业务接口的编写要站在“使用者”的角度定义，三个方面：方法定义的粒度、参数、返回值。

1. 方法定义粒度：关注接口的功能本身，至于这个功能需要包含哪些步骤那是具体的实现，也就是说，功能明确而且单一。
2. 参数：方法所需要的数据，供使用者传入，明确方法所需要的数据，而且尽可能友好，简练。
3. 返回值：一般情况下，entity数据不够，需要自定义DTO,也有可能抛出异常，需要自定义异常，不管是DTO还是异常，尽可能将接口调用的信息返回给使用者，哪怕是失败信息。

> **DTO与entity的区别**

DTO数据传输层：用于Web层和Service层之间传递的数据封装。

entity：用于业务数据的封装，比如数据库中的数据。

- springIOC配置技巧

> **为什么用IOC**

- 对象创建统一托管，（之前是new）
- 规范的生命周期管理。（init，销毁等）
- 灵活的依赖注入(第三方框架自动整合，注解，编程)
- 一致的获取对象

> **Spring-IOC注入方式和场景**

- XML方式：主要用于配置第三方类库或需要命名空间配置
- 注解：自己编写的类，直接在代码中使用注解，如@Service，@Controller
- Java配置类：需要通过代码控制对象创建逻辑的场景，如自定义修改依赖类库

> **Spring-IOC配置**

通过Spring提供的组件自动扫描机制，可以在类路径下寻找标注了上述注解的类，并把这些类纳入进spring容器中管理，这些注解的作用和在xml文件中使用bean节点配置组件时一样的。

```
<context:component-scan base-package=”xxx.xxx.xxx”>
```

component-scan标签默认情况下自动扫描指定路径下的包(含所有子包)，将带有@Component、@Repository、@Service、@Controller标签的类自动注册到spring容器。getBean的默认名称是类名（头字母小写），如果想自定义，可以@Service(“aaaaa”)这样来指定。这种bean默认是“singleton”的，如果想改变，可以使用@Scope(“prototype”)来改变。

当使用`<context:component-scan/>`后，就可以将`<context:annotation-config/>`移除了，前者包含了后者。

另外，@Resource，@Inject 是J2EE规范的一些注解

@Autowired是Spring的注解，可以对类成员变量、方法及构造函数进行标注，完成自动装配的工作。通过 @Autowired的使用来消除setter/getter方法，默认按类型装配，如果想使用名称装配可以结合@Qualifier注解进行使用，如下：

```
@Autowired() @Qualifier("baseDao")
private BaseDao baseDao; 
```

- spring声明式事务使用和理解

> **什么是声明式事务？**

**目的**：交给第三方框架spring来管理，**解脱事务代码**。(开发时不用关注什么时候开启事务，什么时候提交，什么时候回滚)

> **声明式事务使用方法**

- 在Spring早期版本（2.0）中是使用ProxyFactoryBean+XMl方式来配置事务。
- 在Spring配置文件使用tx:advice+aop命名空间，好处就是一次配置永久生效，你无须去关心中间出的问题，不过出错了你很难找出来在哪里出了问题。
- 注解@Transactional的方式，注解可以在方法定义、接口定义、类定义、public方法上，但是不能注解在private、final、static等方法上，因为Spring的事务管理默认是使用Cglib动态代理的： 

1. private方法因为访问权限限制，无法被子类覆盖
2. final方法无法被子类覆盖
3. static是类级别的方法，无法被子类覆盖
4. protected方法可以被子类覆盖，因此可以被动态字节码增强

> **不能被Spring AOP事务增强的方法**

| 序号 | 动态代理策略        | 不能被事务增强的方法                                         |
| ---- | ------------------- | ------------------------------------------------------------ |
| 1    | 基于接口的动态代理  | 除了public以外的所有方法，并且public static的方法也不能被增强 |
| 2    | 基于Cglib的动态代理 | private、static、final的方法                                 |

> **关于Spring的组件注解、注入注解**

- @Component：标识一个组件，当不知道是什么组件，或者该组件不好归类时使用该注解
- @Service：标识业务层组件
- @Repository：标识DAO层组件
- @Controller：标识控制层组件

### WEB技术

- Restful接口运用

> **Restful规范**

1. Restful规范是一种优雅的URI表达方式：/模块/资源/{标识}/集合1/···
2. GET -> 查询操作
3. POST -> 添加/修改操作（用于非幂等操作）
4. PUT -> 修改操作（用于幂等操作）
5. DELETE -> 删除操作

> **怎么实现Restful接口**

- @RequestMapping(value = “/path”,method = RequestMethod.GET)
- @RequestMapping(value = “/path”,method = RequestMethod.POST)
- @RequestMapping(value = “/path”,method = RequestMethod.PUT)
- @RequestMapping(value = “/path”,method = RequestMethod.DELETE)

> **秒杀API的URL设计**

get /seckill/list	秒杀列表
get /seckill/{id}/detail	详情页
get /seckill/time/now	系统时间
post /seckill/{id}/exposer	暴露秒杀
post /seckill/{id}/{md5}/execution	执行秒杀

> **为什么设计url**

对于初学者或者是不太严格的系统来说，url其实是很薄弱的一环，就是大家都很不重视，就随便去写，而不是通过一个基于页面或者是基于系统的业务去严格的设计url，那么当我们有一个良好的url规范去设计一个友好的url时，对于接口使用者：前端，web系统，搜索引擎...主要是工程师进行交互时，可以通过url很好的体现出来。

- springMVC使用技巧

> ### **配置web.xml**

```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                      http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         version="3.0"
         metadata-complete="true">
    <!--用maven创建的web-app需要修改servlet的版本为3.0 -->
    <!--配置DispatcherServlet -->
    <servlet>
        <servlet-name>seckill-dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 配置SpringMVC 需要配置的文件 spring-dao.xml，spring-service.xml,spring-web.xml 
            MyBatis -> Spring -> SpringMVC -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring/spring-*.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>seckill-dispatcher</servlet-name>
        <!--默认匹配所有请求 -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

注意：

- 这里的Servlet版本是3.0，对应Tomcat7.0版本
- 由于我们的配置文件都是以spring-开头命名的，所以可以用通配符*一次性全部加载
- url-pattern设置为/，这是使用了Restful的规范；在使用Struts框架时我们配置的是*.do之类的，这是一种比较丑陋的表达方式

> **src/main/resources/spring包下建立spring-web.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!--配置spring mvc-->
    <!--1,开启springmvc注解模式
    a.自动注册DefaultAnnotationHandlerMapping,AnnotationMethodHandlerAdapter
    b.默认提供一系列的功能:数据绑定，数字和日期的format@NumberFormat,@DateTimeFormat
    c:xml,json的默认读写支持-->
    <mvc:annotation-driven/>

    <!--2.静态资源默认servlet配置-->
    <!--
        1).加入对静态资源处理：js,gif,png
        2).允许使用 "/" 做整体映射
    -->
    <mvc:default-servlet-handler/>

    <!--3：配置JSP 显示ViewResolver-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <!--4:扫描web相关的controller-->
    <context:component-scan base-package="com.lewis.web"/>
</beans>
```

> **Controller设计**

Controller中的每一个方法都对应我们系统中的一个资源URL，其设计应该遵循Restful接口的设计风格。

> **在dto包下新建一个SeckillResult**

SeckillResult是一个VO类(View Object)，属于DTO层，用来封装json结果，方便页面取值；在这里，将其设计成泛型，就可以和灵活地往里边封装各种类型的对象。

- 前端交互分析过程

> **前端页面流程**

![前端页面流程](https://blogs-image.oss-cn-beijing.aliyuncs.com/seckill/process.jpg?Expires=1551413543&OSSAccessKeyId=TMP.AQHOpFLmWavLr74KLW_1XMq-0eEwe3B1qM_IN198Rnm4WWK6-_DKvhmTkw8JAAAwLAIUEOOx9kyjopQtXg0fZgXGXwHetgMCFAOtzFwzd3tCAfJu3O1Z4N1BJzxg&Signature=5ljsgfR4tvZz%2Fl1wDe2zsArVBuY%3D)

> **详情页流程逻辑**

![详情页流程逻辑](https://blogs-image.oss-cn-beijing.aliyuncs.com/seckill/detail-logic.jpg?Expires=1551413596&OSSAccessKeyId=TMP.AQHOpFLmWavLr74KLW_1XMq-0eEwe3B1qM_IN198Rnm4WWK6-_DKvhmTkw8JAAAwLAIUEOOx9kyjopQtXg0fZgXGXwHetgMCFAOtzFwzd3tCAfJu3O1Z4N1BJzxg&Signature=wyfIEqDpn5kFbKmmHDUHnYgLsc8%3D)

- Bootstrap和JS使用

由于项目的前端页面都是由Bootstrap开发的,所以需要先去下载Bootstrap或者是使用在线的CDN服务。而Bootstrap又是依赖于jQuery的，所以需要先引入jQuery。

> **在webapp下建立resources目录，接着建立script目录，建立seckill.js**

```javascript
//存放主要交互逻辑的js代码
// javascript 模块化(package.类.方法)

var seckill = {

    //封装秒杀相关ajax的url
    URL: {
        now: function () {
            return '/seckill/seckill/time/now';
        },
        exposer: function (seckillId) {
            return '/seckill/seckill/' + seckillId + '/exposer';
        },
        execution: function (seckillId, md5) {
            return '/seckill/seckill/' + seckillId + '/' + md5 + '/execution';
        }
    },

    //验证手机号
    validatePhone: function (phone) {
        if (phone && phone.length == 11 && !isNaN(phone)) {
            return true;//直接判断对象会看对象是否为空,空就是undefine就是false; isNaN 非数字返回true
        } else {
            return false;
        }
    },

    //详情页秒杀逻辑
    detail: {
        //详情页初始化
        init: function (params) {
            //手机验证和登录,计时交互
            //规划我们的交互流程
            //在cookie中查找手机号
            var userPhone = $.cookie('userPhone');
            //验证手机号
            if (!seckill.validatePhone(userPhone)) {
                //绑定手机 控制输出
                var killPhoneModal = $('#killPhoneModal');
                killPhoneModal.modal({
                    show: true,//显示弹出层
                    backdrop: 'static',//禁止位置关闭
                    keyboard: false//关闭键盘事件
                });

                $('#killPhoneBtn').click(function () {
                    var inputPhone = $('#killPhoneKey').val();
                    console.log("inputPhone: " + inputPhone);
                    if (seckill.validatePhone(inputPhone)) {
                        //电话写入cookie(7天过期)
                        $.cookie('userPhone', inputPhone, {expires: 7, path: '/seckill'});
                        //验证通过　　刷新页面
                        window.location.reload();
                    } else {
                        //todo 错误文案信息抽取到前端字典里
                        $('#killPhoneMessage').hide().html('<label class="label label-danger">手机号错误!</label>').show(300);
                    }
                });
            }

            //已经登录
            //计时交互
            var startTime = params['startTime'];
            var endTime = params['endTime'];
            var seckillId = params['seckillId'];
            $.get(seckill.URL.now(), {}, function (result) {
                if (result && result['success']) {
                    var nowTime = result['data'];
                    //时间判断 计时交互
                    seckill.countDown(seckillId, nowTime, startTime, endTime);
                } else {
                    console.log('result: ' + result);
                    alert('result: ' + result);
                }
            });
        }
    },

    handlerSeckill: function (seckillId, node) {
        //获取秒杀地址,控制显示器,执行秒杀
        node.hide().html('<button class="btn btn-primary btn-lg" id="killBtn">开始秒杀</button>');

        $.get(seckill.URL.exposer(seckillId), {}, function (result) {
            //在回调函数种执行交互流程
            if (result && result['success']) {
                var exposer = result['data'];
                if (exposer['exposed']) {
                    //开启秒杀
                    //获取秒杀地址
                    var md5 = exposer['md5'];
                    var killUrl = seckill.URL.execution(seckillId, md5);
                    console.log("killUrl: " + killUrl);
                    //绑定一次点击事件
                    $('#killBtn').one('click', function () {
                        //执行秒杀请求
                        //1.先禁用按钮
                        $(this).addClass('disabled');//,<-$(this)===('#killBtn')->
                        //2.发送秒杀请求执行秒杀
                        $.post(killUrl, {}, function (result) {
                            if (result && result['success']) {
                                var killResult = result['data'];
                                var state = killResult['state'];
                                var stateInfo = killResult['stateInfo'];
                                //显示秒杀结果
                                node.html('<span class="label label-success">' + stateInfo + '</span>');
                            }
                        });
                    });
                    node.show();
                } else {
                    //未开启秒杀(浏览器计时偏差)
                    var now = exposer['now'];
                    var start = exposer['start'];
                    var end = exposer['end'];
                    seckill.countDown(seckillId, now, start, end);
                }
            } else {
                console.log('result: ' + result);
            }
        });

    },

    countDown: function (seckillId, nowTime, startTime, endTime) {
        console.log(seckillId + '_' + nowTime + '_' + startTime + '_' + endTime);
        var seckillBox = $('#seckill-box');
        if (nowTime > endTime) {
            //秒杀结束
            seckillBox.html('秒杀结束!');
        } else if (nowTime < startTime) {
            //秒杀未开始,计时事件绑定
            var killTime = new Date(startTime + 1000);//todo 防止时间偏移
            seckillBox.countdown(killTime, function (event) {
                //时间格式
                var format = event.strftime('秒杀倒计时: %D天 %H时 %M分 %S秒 ');
                seckillBox.html(format);
            }).on('finish.countdown', function () {
                //时间完成后回调事件
                //获取秒杀地址,控制现实逻辑,执行秒杀
                console.log('______fininsh.countdown');
                seckill.handlerSeckill(seckillId, seckillBox);
            });
        } else {
            //秒杀开始
            seckill.handlerSeckill(seckillId, seckillBox);
        }
    }
};
```

> **特殊说明**

由于本人的Eclipse内嵌的Tomcat设置的原因，我需要在URL里的所有路径前加上`/seckill`（我的项目名）才可以正常映射到Controller里对应的方法，如下

```
//封装秒杀相关ajax的url
URL: {
    now: function () {
        return '/seckill/seckill/time/now';
    },
    exposer: function (seckillId) {
        return '/seckill/seckill/' + seckillId + '/exposer';
    },
    execution: function (seckillId, md5) {
        return '/seckill/seckill/' + seckillId + '/' + md5 + '/execution';
    }
},
```

如果有同学在后边测试页面时找不到路径，可以将这里的路径里的`/seckill`删掉

### 并发优化

- 前端控制

暴露接口，按钮防重复（点击一次按钮后就变成灰色，禁止重复点击按钮）

- 动静态数据分离

CDN缓存，后端缓存

> **为什么使用Redis**

Redis属于NoSQL，即非关系型数据库，它是key-value型数据库，是直接在内存中进行存取数据的，所以有着很高的性能。

利用Redis可以减轻MySQL服务器的压力，减少了跟数据库服务器的通信次数。秒杀的瓶颈就在于跟数据库服务器的通信速度（MySQL本身的主键查询非常快）

> **为什么不用Redis的hash来存储对象？**

第一：通过Jedis储存对象的方式有大概三种

本项目采用的方式：将对象序列化成byte字节，最终存byte字节；
对象转hashmap，也就是你想表达的hash的形式，最终存map；
对象转json，最终存json，其实也就是字符串。

第二：其实如果你是平常的项目，并发不高，三个选择都可以，这种情况下以hash的形式更加灵活，可以对象的单个属性，但是问题来了，在秒杀的场景下，三者的效率差别很大。

第三：结果如下：

|  10w数据  | 时间 | 内存占用 |
| :-------: | :--: | :------: |
|  存json   | 10s  |   14M    |
|  存byte   |  6s  |    6M    |
| 存jsonMap | 10s  |   20M    |
| 存byteMap |  4s  |    4M    |
|  取json   |  7s  |          |
|  取byte   |  4s  |          |
| 取jsonMap |  7s  |          |
| 取byteMap |  4s  |          |

第四：你该说了，bytemap最快啊，为啥不用啊，因为项目用了超级强悍的序列化工具啊。

- 事务竞争优化

减少事务行级锁的持有时间

> **sql语句的简单优化**

**优化SeckillServiceImpl的executeSeckill()**

用户的秒杀操作分为两步：减库存、插入购买明细，我们在这里进行简单的优化，就是将原本先update（减库存）再进行insert（插入购买明细）的步骤改成：先insert再update。

```java
public SeckillExecution executeSeckill(long seckillId, long userPhone, String md5) throws SeckillException,
        RepeatKillException, SeckillCloseException {

    if (md5 == null || !md5.equals(getMD5(seckillId))) {
        throw new SeckillException("seckill data rewrite");// 秒杀数据被重写了
    }
    // 执行秒杀逻辑:减库存+增加购买明细
    Date nowTime = new Date();

    try {

        // 否则更新了库存，秒杀成功,增加明细
        int insertCount = successKilledDao.insertSuccessKilled(seckillId, userPhone);
        // 看是否该明细被重复插入，即用户是否重复秒杀
        if (insertCount <= 0) {
            throw new RepeatKillException("seckill repeated");
        } else {

            // 减库存,热点商品竞争
            int updateCount = seckillDao.reduceNumber(seckillId, nowTime);
            if (updateCount <= 0) {
                // 没有更新库存记录，说明秒杀结束 rollback
                throw new SeckillCloseException("seckill is closed");
            } else {
                // 秒杀成功,得到成功插入的明细记录,并返回成功秒杀的信息 commit
                SuccessKilled successKilled = successKilledDao.queryByIdWithSeckill(seckillId, userPhone);
                return new SeckillExecution(seckillId, SeckillStatEnum.SUCCESS, successKilled);
            }
        }
    } catch (SeckillCloseException e1) {
        throw e1;
    } catch (RepeatKillException e2) {
        throw e2;
    } catch (Exception e) {
        logger.error(e.getMessage(), e);
        // 将编译期异常转化为运行期异常
        throw new SeckillException("seckill inner error :" + e.getMessage());
    }

}
```

> **为什么要先insert再update**

首先是在更新操作的时候给行加锁，插入并不会加锁，如果更新操作在前，那么就需要执行完更新和插入以后事务提交或回滚才释放锁。而如果插入在前，更新在后，那么只有在更新时才会加行锁，之后在更新完以后事务提交或回滚释放锁。

在这里，插入是可以并行的，而更新由于会加行级锁是串行的。

也就是说是更新在前加锁和释放锁之间两次的网络延迟和GC，如果插入在前则加锁和释放锁之间只有一次的网络延迟和GC，也就是减少的持有锁的时间。

这里先insert并不是忽略了库存不足的情况，而是因为insert和update是在同一个事务里，光是insert并不一定会提交，只有在update成功才会提交，所以并不会造成过量插入秒杀成功记录。

> **深度优化**

**写一个存储过程procedure，然后在MySQL控制台里执行它**

```mysql
-- 秒杀执行储存过程
DELIMITER $$ -- 将定界符从;转换为$$
-- 定义储存过程
-- 参数： in输入参数   out输出参数
-- row_count() 返回上一条修改类型sql(delete,insert,update)的影响行数
-- row_count:0:未修改数据 ; >0:表示修改的行数； <0:sql错误
CREATE PROCEDURE `seckill`.`execute_seckill`
  (IN v_seckill_id BIGINT, IN v_phone BIGINT,
   IN v_kill_time  TIMESTAMP, OUT r_result INT)
  BEGIN
    DECLARE insert_count INT DEFAULT 0;
    START TRANSACTION;
    INSERT IGNORE INTO success_killed
    (seckill_id, user_phone, state)
    VALUES (v_seckill_id, v_phone, 0);
    SELECT row_count() INTO insert_count;
    IF (insert_count = 0) THEN
      ROLLBACK;
      SET r_result = -1;
    ELSEIF (insert_count < 0) THEN
        ROLLBACK;
        SET r_result = -2;
    ELSE
      UPDATE seckill
      SET number = number - 1
      WHERE seckill_id = v_seckill_id
            AND end_time > v_kill_time
            AND start_time < v_kill_time
            AND number > 0;
      SELECT row_count() INTO insert_count;
      IF (insert_count = 0) THEN
        ROLLBACK;
        SET r_result = 0;
      ELSEIF (insert_count < 0) THEN
          ROLLBACK;
          SET r_result = -2;
      ELSE
        COMMIT;
        SET r_result = 1;
      END IF;
    END IF;
  END;
$$
-- 储存过程定义结束
-- 将定界符重新改为;
DELIMITER ;

-- 定义一个用户变量r_result
SET @r_result = -3;
-- 执行储存过程
CALL execute_seckill(1003, 13502178891, now(), @r_result);
-- 获取结果
SELECT @r_result;
```

> **在SeckillDao里添加调用存储过程的方法声明**

```java
/**
 *  使用储存过程执行秒杀
 * @param paramMap
 */
void killByProcedure(Map<String,Object> paramMap);
```

> **接着在SeckillDao.xml里添加该方法对应的sql语句**

```mysql
<!--调用储存过程 -->
<select id="killByProcedure" statementType="CALLABLE">
    CALL execute_seckill(
        #{seckillId,jdbcType=BIGINT,mode=IN},
        #{phone,jdbcType=BIGINT,mode=IN},
        #{killTime,jdbcType=TIMESTAMP,mode=IN},
        #{result,jdbcType=INTEGER,mode=OUT}
    )
</select>
```

> **在SeckillService接口里添加一个方法声明**

```java
/**
 * 调用存储过程来执行秒杀操作，不需要抛出异常
 * 
 * @param seckillId 秒杀的商品ID
 * @param userPhone 手机号码
 * @param md5 md5加密值
 * @return 根据不同的结果返回不同的实体信息
 */
SeckillExecution executeSeckillProcedure(long seckillId,long userPhone,String md5);
```

> **在SeckillServiceImpl里实现这个方法**

```java
@Override
public SeckillExecution executeSeckillProcedure(long seckillId, long userPhone, String md5) {
    if (md5 == null || !md5.equals(getMD5(seckillId))) {
        return new SeckillExecution(seckillId, SeckillStatEnum.DATE_REWRITE);
    }
    Date killTime = new Date();
    Map<String, Object> map = new HashMap<>();
    map.put("seckillId", seckillId);
    map.put("phone", userPhone);
    map.put("killTime", killTime);
    map.put("result", null);
    // 执行储存过程,result被复制
    seckillDao.killByProcedure(map);
    // 获取result
    int result = MapUtils.getInteger(map, "result", -2);
    if (result == 1) {
        SuccessKilled successKilled = successKilledDao.queryByIdWithSeckill(seckillId, userPhone);
        return new SeckillExecution(seckillId, SeckillStatEnum.SUCCESS, successKilled);
    } else {
        return new SeckillExecution(seckillId, SeckillStatEnum.stateOf(result));
    }
}
```

> **存储过程优化总结**

1. 存储过程优化:事务行级锁持有的时间
2. 不要过度依赖存储过程
3. 简单的逻辑依赖存储过程
4. QPS:一个秒杀单6000/qps

经过简单生sql语句优化和深度优化之后，本项目大概能达到一个秒杀单6000qps，这个数据对于一个秒杀商品来说其实已经挺ok了，注意这里是指同一个秒杀商品6000qps，如果是不同商品不存在行级锁竞争的问题。

QAQ，好多啊，终于写完了。。。

## 项目截图

#### 秒杀列表

![秒杀列表](https://blogs-image.oss-cn-beijing.aliyuncs.com/seckill/list.jpg)

#### 秒杀详情页

![秒杀详情页](https://blogs-image.oss-cn-beijing.aliyuncs.com/seckill/detail.jpg)

#### 错误提示

![错误提示](https://blogs-image.oss-cn-beijing.aliyuncs.com/seckill/error.jpg)

#### 开始秒杀

![开始秒杀](https://blogs-image.oss-cn-beijing.aliyuncs.com/seckill/seckill.jpg)

#### 秒杀成功

![秒杀成功](https://blogs-image.oss-cn-beijing.aliyuncs.com/seckill/seckill-success.jpg)

#### 重复秒杀

![重复秒杀](https://blogs-image.oss-cn-beijing.aliyuncs.com/seckill/repeat-seckill.jpg)

#### 秒杀倒计时

![秒杀倒计时](https://blogs-image.oss-cn-beijing.aliyuncs.com/seckill/countdown.jpg)

#### 秒杀结束

![秒杀结束](https://blogs-image.oss-cn-beijing.aliyuncs.com/seckill/stoped.jpg)







