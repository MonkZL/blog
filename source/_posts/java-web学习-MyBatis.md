---
title: java-web学习 MyBatis
date: 2022-12-11 19:25:43
tags: MyBatis
categories: java-web
---

# 简介
+ MyBatis是一款优秀的`持久层框架`,用于简化JDBC开发
+ 官网：https://mybatis.org/mybatis-3/zh/index.html

# 安装和配置
参照官方文档：https://mybatis.org/mybatis-3/zh/getting-started.html

+ 安装
```
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>x.x.x</version>
</dependency>
```

+ 创建存放`xxxMapper.java`的包`com.xxx.mapper`

+ 在resources下面创建和`com.xxx.mapper`对应的存放`xxxMapper.xml`的文件夹`com.xxx.mapper`（注意这里创建的时候应该输入的是`com/xxx/mapper`而不是`com.xxx.mapper`），这样编译后`xxxMapper.class`就会和`xxxMapper.xml`保存在一个文件夹下面，从而符合mybatis的规范。

+ 在resources下面创建mybatis-config.xml文件
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "https://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
   <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql:///mybatis?useSSL=false"/>
                <property name="username" value="root"/>
                <property name="password" value="12345678"/>
            </dataSource>
        </environment>
    </environments>
  <mappers>
    <!-- 我们自己存放mapper的包名 -->
    <package name="com.xxx.mapper"/>
  </mappers>
</configuration>
```

# 使用

+ 创建`User.java`

```
package com.mybatis.pojo;

public class User {
    private Integer id;
    private String username;
    private String password;
    private String companyName;
    private String gender;
    private String addr;

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

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getCompanyName() {
        return companyName;
    }

    public void setCompanyName(String companyName) {
        this.companyName = companyName;
    }

    public String getGender() {
        return gender;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    public String getAddr() {
        return addr;
    }

    public void setAddr(String addr) {
        this.addr = addr;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", companyName='" + companyName + '\'' +
                ", gender='" + gender + '\'' +
                ", addr='" + addr + '\'' +
                '}';
    }
}

```

+ 创建`UserMapper.java`

```
package com.mybatis.mapper;

import com.mybatis.pojo.User;
import org.apache.ibatis.annotations.Param;

import java.util.List;

public interface UserMapper {

    List<User> selectAll();

    // 如果单独传参数的话需要指定参数名字，这样在UserMapper.xml那边才能通过#{id}的方式使用
    User selectById(@Param("id") int id);

    // 如果传入的是User，这样在UserMapper.xml那边能通过#{id}的方式使用
    List<User> selectUser(User user);
    
    // 如果添加了参数名字，这样在UserMapper.xml那边能通过#{user.id}的方式使用
    // List<User> selectUser(@Param("user") User user);

    void addUser(User user);

    void modifyUser(User user);

    void deleteById(@Param("id") int id);

}

```

+ 创建`UserMapper.xml`

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="mybatis.mapper.UserMapper">
    <!--  本地user的companyName和数据库储存的不对应，这里做转换  -->
    <resultMap id="userResultMap" type="com.mybatis.pojo.User">
        <result column="company_name" property="companyName"/>
    </resultMap>
    <select id="selectAll" resultType="com.mybatis.pojo.User">
        select * from user;
    </select>
    <select id="selectById" resultType="com.mybatis.pojo.User">
        select * from user where id = #{id};
    </select>
    <select id="selectUser" resultMap="userResultMap">
        select *
        from user
        <where>
            <if test="username != null and username != ''">
                and username = #{username}
            </if>
            <if test="password != null and password != ''">
                and password = #{password}
            </if>
            <if test="companyName != null and companyName != ''">
                and company_name = #{companyName}
            </if>

            <if test="gender != null and gender != ''">
                and gender = #{gender}
            </if>

            <if test="addr != null and addr != ''">
                and addr = #{addr}
            </if>
        </where>
    </select>
    <insert id="addUser" useGeneratedKeys="true" keyProperty="id">
        insert into user(username,password,company_name,gender,addr)
        values(#{username},#{password},#{companyName},#{gender},#{addr})
    </insert>
    <update id="modifyUser">
        update user
        <set>
            <if test="username != null and username != ''">
                username = #{username},
            </if>
            <if test="password != null and password != ''">
                password = #{password},
            </if>
            <if test="companyName != null and companyName != ''">
                company_name = #{companyName},
            </if>
            <if test="gender != null and gender != ''">
                gender = #{gender},
            </if>
            <if test="addr != null and addr != ''">
                addr = #{addr},
            </if>
        </set>
        where id = #{id}
    </update>
    <delete id="deleteById">
        delete from user where id = #{id}
    </delete>
</mapper>
```

+ 使用`UserMapper.java`的方法

```
package com.mybatis.test;

import com.mybatis.mapper.UserMapper;
import com.mybatis.pojo.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

public class MyBatisTest {
    @Test
    public void selectALl() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder(). build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        List<User> users = userMapper.selectAll();
        System.out.println(users);
    }

    @Test
    public void selectById() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder(). build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        User user = userMapper.selectById(1);
        System.out.println(user);
    }

    @Test
    public void selectUser() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder(). build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        User user = new User();
        user.setUsername("zl");
        user.setPassword("123");
        user.setAddr("cd");
        user.setGender("nan");
        user.setCompanyName("lp");
        List<User> users = userMapper.selectUser(user);
        System.out.println(users);
    }


    @Test
    public void addUser() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder(). build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession(true);
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        User user = new User();
        user.setUsername("zll");
        user.setPassword("123");
        user.setAddr("cd");
        user.setGender("nan");
        user.setCompanyName("lp");
        userMapper.addUser(user);
        System.out.println(user.getId());
    }

    @Test
    public void modifyById() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder(). build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession(true);
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        User user = new User();
        user.setId(1);
        user.setUsername("zlxl");
        user.setAddr("xxx2");
        user.setGender("xxx3");
        user.setCompanyName("xxx4");
        userMapper.modifyUser(user);
    }


    @Test
    public void deleteById() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder(). build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession(true);
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        userMapper.deleteById(2);
    }

}

```
