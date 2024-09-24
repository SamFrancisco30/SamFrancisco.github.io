---
layout: post
title: MyBatis笔记
categories: MyBatis
description: MyBatis笔记
keywords: MyBatis
excerpt: MyBatis Notebook
---

# 什么是MyBatis
MyBatis是一个简化数据库交互的persistence层框架，它抽象了大量与 JDBC 相关的模板代码（如打开/关闭连接、处理准备语句、将结果集映射到 Java 对象等）。 它使 CRUD（创建、读取、更新、删除）操作更加方便，尤其是复杂的 SQL 查询。

# 特点
1. 省去了很多用JDBC要做的事，比如打开/关闭数据库连接等
2. MyBatis可自动将 SQL 结果集映射到 Java 对象 (POJO，Plain Old Java Objects)，否则就需要在 JDBC 中进行手动处理
3. MyBatis 支持外部 XML 文件或注释中定义 SQL，使您的代码更模块化、更简洁

# 使用步骤
1. 创建配置文件mybatis-config.xml

示例：
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <!-- Configure environments for different database setups -->
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC" />
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/mydb" />
                <property name="username" value="root" />
                <property name="password" value="password" />
            </dataSource>
        </environment>
    </environments>

    <!-- Specify locations of mapper XML files -->
    <mappers>
        <mapper resource="com/example/mappers/UserMapper.xml" />
    </mappers>
</configuration>
```

2. 创建Mapper interface，用于声明映射到 SQL 查询的方法。

示例：
```
package com.example.mappers;

import com.example.models.User;
import java.util.List;

public interface UserMapper {
    User selectUserById(int id);
    List<User> selectAllUsers();
    void insertUser(User user);
    void updateUser(User user);
    void deleteUser(int id);
}
```

3. 创建XML mapper文件，如UserMapper.xml

该 XML 文件将 UserMapper 接口中的方法映射到与数据库交互的 SQL 语句中。

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mappers.UserMapper">

    <!-- Query to select a user by ID -->
    <select id="selectUserById" parameterType="int" resultType="com.example.models.User">
        SELECT * FROM users WHERE id = #{id}
    </select>

    <!-- Query to select all users -->
    <select id="selectAllUsers" resultType="com.example.models.User">
        SELECT * FROM users
    </select>

    <!-- Query to insert a new user -->
    <insert id="insertUser" parameterType="com.example.models.User">
        INSERT INTO users (name, email) VALUES (#{name}, #{email})
    </insert>

    <!-- Query to update an existing user -->
    <update id="updateUser" parameterType="com.example.models.User">
        UPDATE users SET name = #{name}, email = #{email} WHERE id = #{id}
    </update>

    <!-- Query to delete a user by ID -->
    <delete id="deleteUser" parameterType="int">
        DELETE FROM users WHERE id = #{id}
    </delete>

</mapper>
```

4. 定义一个POJO
POJO 表示数据库中的用户表。 表中的每一列都映射到该类中的一个字段。