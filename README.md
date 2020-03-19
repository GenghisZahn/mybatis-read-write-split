[TOC]

### 1. Introduction

Some time ago, there was a need for the project to separate read and write, so this class library was completed `mybatis-read-write-split` to achieve read and write separation.

* Features
Supports two modes of active / standby separation：
1. Business transparent read and write separation. Automatically parse SQL read and write types and perform routing and forwarding.
2. Annotation-based read and write separation. Read and write separation through the configuration in the annotation.

The above two modes can be mixed: the default read and write types of SQL are automatically parsed. If the annotation has a specified data source, routing is performed according to the annotation.

### 2. Usage
* pom.xml Add dependency

```xml
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-read-write-split-core</artifactId>
            <version>2.0-SNAPSHOT</version>
        </dependency>
```

* Configure the data source

```xml
    <!-- Replace the original DataSource -->
    <bean id="dataSource" class="org.mybatis.rw.MultiReadDataSource">
        <property name="masterDataSource" ref="masterDataSource"/>
        <property name="slaveDataSourceList">
            <list>
                <ref bean="slaveDataSource1"></ref>
                <ref bean="slaveDataSource2"></ref>
            </list>
        </property>
    </bean>
    
    <bean id="masterDataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
        <!-- Configuration of each data source -->
    </bean>
    
    <bean id="slaveDataSource1" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
        <!-- Configuration of each data source -->
    </bean>
    
    <bean id="slaveDataSource2" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
        <!-- Configuration of each data source -->
    </bean>
```

#### 2.1. Business transparent read and write

mybatis Automatically analyze read or write operations and perform corresponding routing operations

* mybatis Profile added interceptor

```xml
   <plugins>
        <plugin interceptor="org.mybatis.rw.interceptor.ReadWriteDistinguishInterceptor">
        </plugin>
    </plugins>
```

#### 2.2. Distinguish reads and writes by annotations

Display the specified read master or standby library through the annotation on the method

* Add on target method `@DataSource()`

Such as

```java
    @DataSource(DataSourceType.MASTER)
    public User getUserByIdFromMaster(Integer userId) {
        //some operation
    }
```


### 3.Internal implementation

#### 3.1. Business transparent read and write
![](https://raw.githubusercontent.com/chenzz/static-resource/master/941DC39B-846A-4F86-8F61-F810F9543AB0.png)

1. Mapper calls MyBatis for reading and writing
2. MyBatis analyzes read and write types and stores them in ThreadLocal
3. Custom DataSource gets the read and write types from ThreadLocal and routes to the corresponding child DataSource
4. Use the corresponding child DataSource for read and write operations

#### 2.2. Read and write distinction through annotations

1. Aspects of Spring read the contents of annotations, analyze read / write operations
2. Throw the analysis results into ThreadLocal
3. Custom DataSource gets the DataSource type from ThreadLocal and routes it to the corresponding child DataSource
4. Use the corresponding child DataSource Read and write operations

