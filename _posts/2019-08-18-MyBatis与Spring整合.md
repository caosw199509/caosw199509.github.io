---
layout:     post
title:      MyBatis与Spring整合
subtitle:   实际项目中Spring与MyBatis都是整合在一起使用的，掌握传统DAO方式整合以及掌握Mapper接口方式整合。
date:       2019-08-18
author:     caosw
header-img: img/404-bg.jpg
catalog: true
tags:
    - Java
    - Spring
    - MyBatis
---

# MyBatis与Spring整合
***
### 1、整合环境搭建
这里创建的是maven工程，因此在pom.xml文件中需要引入如下jar包。

    <project xmlns="http://maven.apache.org/POM/4.0.0"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://maven.apache.org/POM/4.0.0  http://maven.apache.org/xsd/maven-4.0.0.xsd">
     <modelVersion>4.0.0</modelVersion>
     <groupId>com.epoint</groupId>
     <artifactId>MyBatis_Spring</artifactId>
     <version>0.0.1-SNAPSHOT</version>
     <properties>
         <org.springframework.version>4.3.7.RELEASE</org.springframework.version>
     </properties>
     <dependencies>
         <dependency>
              <groupId>org.mybatis</groupId>
              <artifactId>mybatis</artifactId>
              <version>3.5.2</version>
         </dependency>
         <dependency>
              <groupId>org.apache.ant</groupId>
              <artifactId>ant</artifactId>
              <version>1.9.6</version>
         </dependency>
         <dependency>
              <groupId>org.apache.ant</groupId>
              <artifactId>ant-launcher</artifactId>
              <version>1.9.6</version>
         </dependency>
         <dependency>
              <groupId>org.ow2.asm</groupId>
              <artifactId>asm</artifactId>
              <version>5.1</version>
         </dependency>
         <dependency>
              <groupId>cglib</groupId>
              <artifactId>cglib</artifactId>
              <version>3.2.4</version>
         </dependency>
         <dependency>
              <groupId>commons-logging</groupId>
              <artifactId>commons-logging</artifactId>
              <version>1.2</version>
         </dependency>
         <dependency>
              <groupId>org.javassist</groupId>
              <artifactId>javassist</artifactId>
              <version>3.21.0-GA</version>
         </dependency>
         <dependency>
              <groupId>log4j</groupId>
              <artifactId>log4j</artifactId>
              <version>1.2.17</version>
         </dependency>
         <dependency>
              <groupId>org.apache.logging.log4j</groupId>
              <artifactId>log4j-api</artifactId>
              <version>2.3</version>
         </dependency>
         <dependency>
              <groupId>org.apache.logging.log4j</groupId>
              <artifactId>log4j-core</artifactId>
              <version>2.3</version>
         </dependency>
         <dependency>
              <groupId>ognl</groupId>
              <artifactId>ognl</artifactId>
              <version>3.1.12</version>
         </dependency>
         <dependency>
              <groupId>org.slf4j</groupId>
              <artifactId>slf4j-api</artifactId>
              <version>1.7.22</version>
         </dependency>
         <dependency>
              <groupId>org.slf4j</groupId>
              <artifactId>slf4j-log4j12</artifactId>
              <version>1.7.22</version>
         </dependency>
         <dependency>
              <groupId>mysql</groupId>
              <artifactId>mysql-connector-java</artifactId>
              <version>5.1.47</version>
         </dependency>
         <dependency>
              <groupId>aopalliance</groupId>
              <artifactId>aopalliance</artifactId>
              <version>1.0</version>
         </dependency>
         <dependency>
              <groupId>org.apache.commons</groupId>
              <artifactId>commons-dbcp2</artifactId>
              <version>2.1.1</version>
         </dependency>
         <dependency>
              <groupId>org.apache.commons</groupId>
              <artifactId>commons-pool2</artifactId>
              <version>2.4.2</version>
         </dependency>
         <dependency>
              <groupId>org.mybatis</groupId>
              <artifactId>mybatis-spring</artifactId>
              <version>1.3.2</version>
         </dependency>
         <!-- spring start -->
         <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-aop</artifactId>
              <version>${org.springframework.version}</version>
         </dependency>
         <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-aspects</artifactId>
              <version>${org.springframework.version}</version>
         </dependency>
         <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-beans</artifactId>
              <version>${org.springframework.version}</version>
         </dependency>
         <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-context</artifactId>
              <version>${org.springframework.version}</version>
         </dependency>
         <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-context-support</artifactId>
              <version>${org.springframework.version}</version>
         </dependency>
         <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-core</artifactId>
              <version>${org.springframework.version}</version>
         </dependency>
         <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-expression</artifactId>
              <version>${org.springframework.version}</version>
         </dependency>
         <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-instrument</artifactId>
              <version>${org.springframework.version}</version>
         </dependency>
         <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-instrument-tomcat</artifactId>
              <version>${org.springframework.version}</version>
         </dependency>
         <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-jdbc</artifactId>
              <version>${org.springframework.version}</version>
         </dependency>
         <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-jms</artifactId>
              <version>${org.springframework.version}</version>
         </dependency>
         <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-messaging</artifactId>
              <version>${org.springframework.version}</version>
         </dependency>
         <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-orm</artifactId>
              <version>${org.springframework.version}</version>
         </dependency>
         <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-oxm</artifactId>
              <version>${org.springframework.version}</version>
         </dependency>
         <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-test</artifactId>
              <version>${org.springframework.version}</version>
         </dependency>
         <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-tx</artifactId>
              <version>${org.springframework.version}</version>
         </dependency>
         <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-web</artifactId>
              <version>${org.springframework.version}</version>
         </dependency>
         <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-webmvc</artifactId>
              <version>${org.springframework.version}</version>
         </dependency>
         <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-webmvc-portlet</artifactId>
              <version>${org.springframework.version}</version>
         </dependency>
         <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-websocket</artifactId>
              <version>${org.springframework.version}</version>
         </dependency>
         <!-- spring end -->
     </dependencies>
     <build>
         <plugins>
              <plugin>
                  <groupId>org.apache.maven.plugins</groupId>
                  <artifactId>maven-compiler-plugin</artifactId>
                  <version>3.7.0</version>
                  <configuration>
                       <source>1.8</source>
                       <target>1.8</target>
                       <encoding>UTF-8</encoding>
                  </configuration>
              </plugin>
         </plugins>
     </build>
    </project>

其中包含了Spring框架所需的jar包、MyBatis框架所需要的jar包、Mybatis与Spring整合的中间jar包mybatis-spring、数据库驱动包以及数据源所需jar包（整合时所使用的DBCP数据源）。

### 2、编写配置文件
##### 2.1、创建db.properties文件
该文件中除了配置了连接数据库基本4项还包括了数据库连接池的最大连接数（maxTotal）、最大空闲连接数（maxIdle）以及初始化连接数（initialSize）。

    jdbc.driver=com.mysql.jdbc.Driver
    jdbc.url=jdbc:mysql://localhost:3306/mybatis
    jdbc.username=root
    jdbc.password=Gepoint
    jdbc.maxTotal=30
    jdbc.maxIdle=10
    jdbc.initialSize=5

##### 2.2、创建applicationContent.xml文件
该文件中定义了读取properties文件配置，然后配置数据源，接下来配置事务管理器并开启事务注解，最后配置MyBaits工厂和Spring整合。

    <beans  xmlns="http://www.springframework.org/schema/beans"
     xmlns:context="http://www.springframework.org/schema/context"
     xmlns:p="http://www.springframework.org/schema/p"
     xmlns:aop="http://www.springframework.org/schema/aop"
     xmlns:tx="http://www.springframework.org/schema/tx"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://www.springframework.org/schema/beans  http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
     http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context-4.0.xsd
     http://www.springframework.org/schema/aop  http://www.springframework.org/schema/aop/spring-aop-4.0.xsd http://www.springframework.org/schema/tx  http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
     http://www.springframework.org/schema/util  http://www.springframework.org/schema/util/spring-util-4.0.xsd">
        <!-- 读取db.properties -->
        <context:property-placeholder  location="classpath:db.properties"/>
        <!-- 配置数据源 -->
        <bean id="dataSource"  class="org.apache.commons.dbcp2.BasicDataSource">
            <!-- 数据库驱动 -->
            <property name="driverClassName"  value="${jdbc.driver}" />
            <!-- 连接数据库的url -->
            <property name="url" value="${jdbc.url}" />
            <!-- 用户名 -->
            <property name="username"  value="${jdbc.username}" />
            <!-- 密码 -->
            <property name="password"  value="${jdbc.password}" />
            <!-- 最大连接数 -->
            <property name="maxTotal"  value="${jdbc.maxTotal}" />
            <!-- 最大空闲连接 -->
            <property name="maxIdle"  value="${jdbc.maxIdle}" />
            <!-- 初始化连接数 -->
            <property name="initialSize"  value="${jdbc.initialSize}" />
        </bean>
        <!-- 事务管理器，依赖于数据源 -->
        <bean id="transactionManager"  class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <property name="dataSource" ref="dataSource"  />
        </bean>
        <!-- 开启事务注解 -->
        <tx:annotation-driven  transaction-manager="transactionManager"/>
        <!-- 配置MyBatis工厂 -->
        <bean id="sqlSessionFactory"  class="org.mybatis.spring.SqlSessionFactoryBean">
            <!-- 注入数据源 -->
            <property name="dataSource" ref="dataSource"  />
            <!-- 指定核心配置文件位置 -->
            <property name="configLocation"  value="classpath:mybatis-config.xml" />
        </bean>
    </beans>

其中，MyBatis工厂的作用就是构建SqlSessionFactory，通过mybatis-spring包中提供的org.mybatis.spring.SqlSessionFactoryBean类进行配置。通常该配置中需要提供两个参数：一个是数据源，另一个是MyBatis的配置文件路径。

##### 2.3、创建mybatis-config.xml文件
由于数据源已在Spring中进行配置，因此在MyBatis的配置文件中不需要再配置数据源信息。只需要使用`<typeAliases>`和`<mappers>`元素来配置文件别名以及指定Mapper文件位置即可。

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE configuration PUBLIC  "http://mybatis.org/dtd/mybatis-3-config.dtd"  "mybatis-3-config.dtd" >
    <configuration>
        <!-- 配置别名 -->
        <typeAliases>
            <package name="com.epoint.po"/>
        </typeAliases>
        <!-- 配置Mapper的位置 -->
        <mappers>
        ...
        </mappers>
    </configuration>

##### 2.4、 创建log4j.properties文件
配置日志记录文件。

    # Global logging confguration
    log4j.rootLogger=ERROR, stdout
    # MyBatis logging confguration ...
    log4j.logger.com.epoint=DEBUG
    # Console output. ..
    log4j.appender.stdout=org.apache.log4j.ConsoleAppender
    log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
    log4j.appender.stdout.layout.ConversionPattern=%5p [%t]  - %m%n

### 3、传统DAO方式的开发整合
采用传统DAO开发方式进行MyBatis与Spring框架的整合时，需要编写DAO接口以及接口实现类，并需要将DAO实现类注入SqlSessionFactory。
##### 3.1、创建表customer

    CREATE TABLE `t_customer` (
    `id` INT (32) NOT NULL AUTO_INCREMENT,
    `username` VARCHAR (50) DEFAULT NULL,
    `jobs` VARCHAR (50) DEFAULT NULL,
    `phone` VARCHAR (16) DEFAULT NULL,
    PRIMARY KEY (`id`)
    ) ENGINE = INNODB AUTO_INCREMENT = 9 DEFAULT CHARSET = utf8mb4

##### 3.2、创建持久化类
创建一个com.epoint.po包，并在包中创建持久化类Customer，并添加setter和getter方法以及重写toString方法。

    package com.epoint.po;
    public class Customer {
        private Integer id;
        private String username;
        private String jobs;
        private String phone;
        public Integer getId() {
            return id;
        }
        public void setId(Integer id) {
            this.id = id;
        }
        public String getUsername() {
            return username;
        }
        public void setUsername(String username) {
            this.username = username;
        }
        public String getJobs() {
            return jobs;
        }
        public void setJobs(String jobs) {
            this.jobs = jobs;
        }
        public String getPhone() {
            return phone;
        }
        public void setPhone(String phone) {
            this.phone = phone;
        }
        public String toString() {
            return "Customer [id=" + id + ", username=" +  username + ", jobs=" + jobs + ", phone=" + phone + "]";
        }
    }

##### 3.3、创建映射文件
在包com.epoint.mapper中创建映射文件CustomerMapper.xml，在该文件中编写根据id查询客户信息的映射语句。

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC  "http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd"  "mybatis-3-mapper.dtd" >
    <mapper namespace="com.epoint.mapper.CustomerMapper">
        <!-- 根据id查询客户信息 --->
        <select id="findCustomerById"  parameterType="Integer"
         resultType="customer">
            select * from t_customer where id = #{id}
        </select>
    </mapper>

##### 3.4、在MyBatis核心配置文件添加映射文件

    <mapper resource="com/epoint/mapper/CustomerMapper.xml"/>

##### 3.5、创建接口
创建一个com.epoint.dao包，并在包中创建接口CustomerDao，在接口中编写一个通过id查询客户信息的方法findCustomerById()。

    package com.epoint.dao;
    import com.epoint.po.Customer;
    public interface CustomerDao {
        // 通过id查询客户
        public Customer findCustomerById(Integer id);
    }

##### 3.6、创建接口实现类
创建一个com.epoint.dao.impl包，并在包中创建CustomerDao接口的实现类CustomerDaoImpl，该类还需要继承SqlSessionDaoSupport类。

    package com.epoint.dao.impl;
    import org.mybatis.spring.support.SqlSessionDaoSupport;
    import com.epoint.dao.CustomerDao;
    import com.epoint.po.Customer;
    public class CustomerDaoImpl extends SqlSessionDaoSupport implements CustomerDao {
        // 通过id查询客户
        public Customer findCustomerById(Integer id) {
            return this.getSqlSession().selectOne("com.epoint.mapper.CustomerMapper.findCustomerById" ,id);
        }
    }

##### 3.7、applicationContext.xml添加配置

    <!-- 实例化Dao -->
    <bean id="customerDao"  class="com.epoint.dao.impl.CustomerDaoImpl">
        <!-- 注入SqlSessionFactory对象实例 -->
        <property name="sqlSessionFactory"  ref="sqlSessionFactory" />
    </bean>

##### 3.8、整合测试
创建一个com.epoint.test包，在包中创建测试类DaoTest，并编写测试方法findCustomerByIdDaoTest()。

    package com.epoint.test;
    import org.junit.Test;
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    import com.epoint.dao.CustomerDao;
    import com.epoint.po.Customer;
    public class DaoTest {
    @Test
    public void findCustomerByIdDaoTest() {
            ApplicationContext act = new ClassPathXmlApplicationContext("applicationContext.xml");
            // 根据容器中bean的id来获取指定的bean
            CustomerDao customerDao = (CustomerDao) act.getBean("customerDao");
            Customer customer = customerDao.findCustomerById(1);
            System.out.println(customer);
        }
    }

运行，测试结果如下

    DEBUG [main] - ==>  Preparing: select * from t_customer  where id = ?
    DEBUG [main] - ==> Parameters: 1(Integer)
    DEBUG [main] - <==      Total: 1
    Customer [id=1, username=Joy, jobs=docter,  phone=111111111]

### 4、Mapper接口方式的开发整合
虽然传统的DAO开发方式可以实现所需功能，但是采用这种方式在实现类中会出现大量的重复代码，在开发是也需要指定映射文件中执行语句的id，并不能保证编写时id的正确性（运行时才能知道）。因此MyBatis提供了另外一种编程方式，即使用Mapper接口编程。
##### 4.1、创建接口
在包com.epoint.mapper中创建CustomerMapper接口

    package com.epoint.mapper;
    import com.epoint.po.Customer;
    public interface CustomerMapper {
        // 通过id查询客户
        public Customer findCustomerById(Integer id);
    }

##### 4.2、配置映射文件

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC  "http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd"  "mybatis-3-mapper.dtd" >
    <mapper namespace="com.epoint.mapper.CustomerMapper">
        <!-- 根据id查询客户信息 --->
        <select id="findCustomerById"  parameterType="Integer"
            resultType="customer">
            select * from t_customer where id = #{id}
        </select>
    </mapper>

##### 4.3、在MyBatis核心配置文件添加映射文件

    <mapper resource="com/epoint/mapper/CustomerMapper.xml"/>

##### 4.4、applicationContext.xml添加配置

    <!-- Mapper代理开发 -->
    <bean id="customerMapper"  class="org.mybatis.spring.mapper.MapperFactoryBean">
        <property name="mapperInterface"  value="com.epoint.mapper.CustomerMapper" />
        <property name="sqlSessionFactory"  ref="sqlSessionFactory" />
    </bean>

##### 4.5、整合测试
在DaoTest测试类中，添加测试方法findCustomerByIdMapperTest()。

    @Test
    public void findCustomerByIdMapperTest() {
        ApplicationContext act = new  ClassPathXmlApplicationContext("applicationContext.xml");
        CustomerMapper customerMapper =  act.getBean(CustomerMapper.class);
        Customer customer =  customerMapper.findCustomerById(1);
        System.out.println(customer);
    }

运行，测试结果如下

    DEBUG [main] - ==>  Preparing: select * from t_customer  where id = ?
    DEBUG [main] - ==> Parameters: 1(Integer)
    DEBUG [main] - <==      Total: 1
    Customer [id=1, username=Joy, jobs=docter,  phone=111111111]

### 5、事务测试
在MyBatis与Spring整合中，事务是由Spring进行管理的。在进行配置applicationContext.xml文件时已经配置了事务管理器，并开启了事务注解。
在实际项目开发中，业务层（service）既是处理业务的地方，又是管理数据库事务的地方。
##### 5.1、编写添加操作的测试方法
在CustomerMapper接口中，编写测试方法addCustomer()。

    // 添加客户
    public void addCustomer(Customer customer);

##### 5.2、在映射文件中编写执行插入操作的SQL

    <!-- 添加客户信息 -->
    <insert id="addCustomer" parameterType="customer">
        insert into t_customer(username,jobs,phone)
        values(#{username},#{jobs},#{phone})
    </insert>

##### 5.3、编写业务接口
创建一个com.epoint.service包，并在包中创建接口CustomerService，在接口中编写添加客户的方法addCustomer()

    package com.epoint.service;
    import com.epoint.po.Customer;
    public interface CustomerService {
        public void addCustomer(Customer customer); 
    }

##### 5.4、编写接口实现类
创建一个com.epoint.service.impl包，并在包中创建CustomerServiceImpl接口实现类，来实现接口中的方法。

    package com.epoint.service.impl;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;
    import com.epoint.mapper.CustomerMapper;
    import com.epoint.po.Customer;
    import com.epoint.service.CustomerService;
    @Service
    @Transactional
    public class CustomerServiceImpl implements CustomerService{
        // 注解注入CustomerMapper
        @Autowired
        private CustomerMapper customerMapper;
        // 添加客户
        public void addCustomer(Customer customer) {
            this.customerMapper.addCustomer(customer);
            int i=1/0;    // 模拟添加操作后系统突然出现异常
        }
    }

使用Spring的注解@Service来标识业务层的类，使用了@Transactional注解标识事务处理的类，通过@Autowired注解将CustomerMapper接口注入到类中。
##### 5.5、applicationContext.xml中开启注解扫描

    <!-- 开启扫描 -->
    <context:component-scan base-package="com.epoint.service" />
##### 5.6、整合测试
在DaoTest测试类中，添加测试方法transactionTest()。

    @Test
    public void transactionTest() {
        ApplicationContext act = new  ClassPathXmlApplicationContext("applicationContext.xml");
        CustomerService customerService =  act.getBean(CustomerService.class);
        Customer customer = new Customer();
        customer.setUsername("test");
        customer.setJobs("manager");
        customer.setPhone("9999999999");
        customerService.addCustomer(customer);
    }

运行，测试结果查看数据中测试数据是否插入成功。

`参考书籍：Java EE企业级应用开发教程（Spring Spring MVC MyBatis）`