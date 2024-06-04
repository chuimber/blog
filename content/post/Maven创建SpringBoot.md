---
title: "Maven创建SpringBoot"
date: 2024-06-04T17:05:01+08:00
lastmod: 2024-06-04T17:05:01+08:00
draft: false
keywords: []
description: ""
tags: []
categories: []
author: "chuimber"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---

<!--more-->
# 先说两种方式
- 继承SpringBoot提供的父Pom
- 在依赖关系管理\<dependencyManagement\>中通过添加带有 scope=import 的 spring-boot-dependencies 工件来进行依赖项管理
## 继承SpringBoot提供的父Pom
```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.0</version>
</parent>
```
这样做的目的是将 Spring Boot 的父 POM 作为当前 Maven 项目的父级，以便继承其默认配置和依赖项。

对工件溯源可以看到`spring-boot-starter-parent`的父pom为`spring-boot-dependencies`,在`spring-boot-dependencies`中对各种依赖版本进行了预定义，可以统一项目中的版本依赖，从而避免了潜在的版本冲突。在这种情况下，我们只需要声明所需的相关依赖，而不必显式指定版本号。

例如，如果要创建一个使用 Spring Web 模块的 Spring Boot 应用程序，则只需要添加以下依赖项即可,版本会自动指定为上面的2.5.0版本：
```java
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```
## 在依赖关系管理\<dependencyManagement\>中通过添加带有 scope=import 的 spring-boot-dependencies 工件来进行依赖项管理
Spring Boot 提供父 POM，以便更轻松地创建 Spring Boot 应用程序。
但是，如果我们已经有父 POM 可以继承，则使用父 POM 可能并不总是可取的。在实践中，我们可能会受到设计规则或其他偏好的限制，无法使用不同的父级。如果我们不使用父 POM，我们仍然可以通过添加带有 scope=import 的spring-boot-dependencies 工件来从依赖项管理中受益：
```java
<dependencyManagement>
     <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.5.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
如需创建SpringWeb模块的SpringBoot，同理：
```java
<dependencies>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
</dependencies>
```
> 另一方面，如果没有父 POM，我们将不再受益于插件管理。 这意味着我们需要显式添加`spring-boot-maven-plugin`：
```java
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```
# Maven中依赖优先级关系
在 Maven 中，依赖项的版本解析遵循以下优先级：
1. 项目内声明的依赖项版本具有最高优先级，即在项目 pom.xml 文件中直接声明的依赖项。
2. 如果项目内没有声明依赖项版本，但在父项目的 dependencyManagement 标签中声明了版本，那么这个版本将被使用。
3. 如果项目和父项目都没有声明依赖项版本，但是在 Maven 中心仓库中存在一个默认版本，那么将会使用这个版本。
4. 如果上述情况都不符合，那么 Maven 将会尝试解析最新版本的依赖项。
   
参考文献：
- https://www.kancloud.cn/hanxuming/java001/3167494
- https://www.baeldung.com/spring-boot-dependency-management-custom-parent；
- https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html