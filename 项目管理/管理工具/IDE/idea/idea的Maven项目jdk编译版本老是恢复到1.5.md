---
title: 'idea的Maven项目jdk编译版本老是恢复到1.5'
Date: 2019-08-11 12:23:43
tag: 'maven'
---
idea中导入的普通maven项目，导入时设置了编译jdk版本为1.8，但每次重新打开都会恢复到1.5，
反反复复，如果每次都手动去更改版本，那可有够恶心！

在网上查了一下，大致有三种解决方法：
## 一、沙雕式
每次手动改：
```yml
1. ctrl+shift+alt+s: 打开项目配置，设置Modules的Language Level为8；

2. ctrl+alt+s: 打开设置，搜索"java compiler"，将默认jdk和当前modual的jdk版本切换为1.8。
```

## 二、修改项目 `pom.xml` 文件
在pom文件中指定compiler的版本。这种方式仅对当前项目来说有效，

但如果新导入其他项目就需要再对导入的项目进行配置，治标不治本；
```xml
<!-- 第一种写法：细节型 -->
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.5.0</version>
      <configuration>
        <source>1.8</source>
        <target>1.8</target>
      </configuration>
    </plugin>
  </plugins>
</build>

<!-- 第二种写法：简单型 -->
<properties>
  <maven.compiler.source>1.8</maven.compiler.source>
  <maven.compiler.target>1.8</maven.compiler.target>
</properties>
```

## 三、修改 Maven 配置
就是修改`setting.xml` 文件。这个方法是真正的一劳永逸型！

具体方法是在<profiles></profiles>节点中添加以下配置：
```xml
<profile>
  <id>jdk-1.8</id>
  <activation>
    <activeByDefault>true</activeByDefault>
    <jdk>1.8</jdk>
  </activation>
  <properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
  </properties>
</profile>
```