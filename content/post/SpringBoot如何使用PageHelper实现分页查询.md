+++
title = 'SpringBoot如何使用PageHelper实现分页查询'
date = 2024-05-31T01:22:06+08:00
draft = false
+++
在原始的分页查询方法中，需要编写复杂的SQL语句来限制查询结果的范围，通常需要使用LIMIT或者ROWNUM等数据库特定的语法来实现分页。在每个需要分页的查询方法中，都需要手动计算分页的起始位置和偏移量，通常需要根据页码和每页数量来计算，这部分代码会在不同的查询方法中重复出现。
# 原始分页方法
1. Controller
```java
    //原始分页方法
    @GetMapping("/chapter01/primitive")
    public List<UserEntity> chapter01(@RequestParam Integer pageNum, @RequestParam Integer pageSize){
        HashMap<String,Integer> map = new HashMap<>();
        map.put("offset",(pageNum-1)*pageSize);
        map.put("count",pageSize);
        return chapter01Mapper.findAll(map);
    }
```
2. Mapper接口
```java
public interface Chapter01Mapper {
    List<UserEntity> findAll(HashMap<String,Integer> map);
}
```
3. Mapper.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.example.Mapper.Chapter01Mapper">
    <resultMap id="UserResultMap" type="com.example.Chapter01.UserEntity">
        <id property="id" column="id"/>
        <result property="name" column="name"/>
        <result property="age" column="age"/>
        <result property="phone" column="phone"/>
        <result property="address" column="address"/>
    </resultMap>

    <sql id="baseSQL">
        select * from `table`
    </sql>
    <!-- 手动配置分页 -->
    <select id="findAll" parameterType="HashMap" resultMap="UserResultMap">
        <include refid="baseSQL"/>
        <if test="offset != null and count != null">
         limit #{offset},#{count}
        </if>
    </select>
</mapper>
```
# 使用PageHelper
1. 导入依赖
```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.4.6</version>
</dependency>
```
2. controller
```java
    //PageHelper插件方法
    @GetMapping("/chapter01/pageHelper")
    public PageInfo<UserEntity> chapter02(@RequestParam(defaultValue = "1") Integer pageNum, @RequestParam(defaultValue = "10") Integer pageSize){
        PageHelper.startPage(pageNum,pageSize);
        List<UserEntity> list =chapter01Mapper.findAllByPage();
        return new PageInfo<>(list);
    }
```
3. mapper接口
```java
public interface Chapter01Mapper {
    List<UserEntity> findAllByPage();
}
```
4. mapper.xml
```xml
    <!-- 省略 -->
    <!-- PageHelper分页 -->
    <select id="findAllByPage" resultMap="UserResultMap">
        <include refid="baseSQL"/>
    </select>
```
在使用PageHelper进行分页时，`PageHelper.startPage(pageNum,pageSize)`是开启分页功能的关键代码，通过传入当前页码和页面记录显示数即可自动计算分页相关的属性，并将计算出的所有数据封装在Page对象内，Page继承自ArrayList，所以后续不仅可以得到查询列表还可以通过调用Page的暴露方法得到Page内部与分页相关的数据，比如页码、页面大小、起始行、末行、总数、总页数、排序等。但实际上想要一次性返回所有关键性信息可以使用PageInfo对象，PageInfo包含的分页信息比Page更全，对于前端来说返回的数据更加完整规范，在例子中使用PageInfo的构造方法传入page对象将page转换为PageInfo返回，也可通过Page的toPageInfo方法直接转换。
# 小结
调试的时候发现在执行`List<UserEntity> list =chapter01Mapper.findAllByPage();`后，本应返回的List类型实际上被修改为Page。看了下源码和查阅资料得到一些粗糙简单的结论：
- 在MyBatis集成PageHelper的情况下，当调用chapter01Mapper.findAllByPage()方法时，PageHelper会在该方法执行前拦截并进行一些处理，其中之一就是开启分页，PageHelper通过AOP（面向切面编程）的方式，在findAllByPage()方法执行前动态地修改其行为，比如SQL语句后添加分页语句，以实现分页功能,并将结果封装成一个Page对象。
