# 2017/11/24
## SpringBoot整合MyBatis

**1.pom文件中配置**

    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>1.1.1</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>

**2.Application启动类上配置注解**

    @MapperScan("com.forest.core.dao")
    //让程序能够扫描到mapper对象

**3.配置application.yml**

    spring:
      datasource:
          driver-class-name: com.mysql.jdbc.Driver
          url: jdbc:mysql://localhost:3306/test?useSSL=false
          username: root
          password: 123
    # 制定Mapper的地址
    mybatis:
      mapper-locations: classpath:mapper/*.xml

**4. 显示sql语句**

SpringBoot默认支持logback.xml，无需另外添加jar包。可以在resource路径下设置logback.xml。将level设置成debug，就可以在日志中输出sql语句了

    <?xml version="1.0" encoding="UTF-8"?>
    <configuration>
        <include resource="org/springframework/boot/logging/logback/base.xml" />
        <logger name="com.forest.core" level="DEBUG" />
        <springProfile name="staging">
            <logger name="com.forest.core" level="TRACE" />
        </springProfile>
    </configuration>
