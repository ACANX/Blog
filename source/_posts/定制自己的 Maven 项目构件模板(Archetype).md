---
title: 定制自己的 Maven 项目构件模板（Archetype）
date: 25-05-15 20:17:00
categories: "Maven" #文章分类目录 可以省略
tags: #Java 
     - Java
     - Maven
     - archetype
description: 定制自己的 Maven 项目构件模板（Archetype）  #你對本頁的描述 可以省略
---

当你创建一个新的Maven模块时，要么使用maven官方的构件模板（Archetype），因为项目好久都没更新，生成的模板插件比较臃肿，项目模块开多了以后，用起来就不太方便了；要么基于已有的项目复制粘贴再修改调整，费时费力还容易出错。因此决定定制自己的 Maven 项目构件模板（Archetype），具体的步骤如下：

**一、创建原型模板项目**
==============

创建原型模板既可以使用 Maven 的 `archetype` 插件基于现有的某个项目来生成模板，也可以手动创建，下面分别做介绍。

方式一 **从现有项目生成**
---------------

选择一个符合目标模板的Maven项目模块，使用以下命令来创建原型模板：

```bash
mvn archetype:create-from-project -Dinteractive=false
```

* 在项目根目录运行此命令，Maven 会生成一个 `target/generated-sources/archetype` 目录，即你的模板项目。

方式二 **手动创建**
------------

```bash
mvn archetype:generate  -DarchetypeGroupId=org.apache.maven.archetypes -DarchetypeArtifactId=maven-archetype-archetype  -DarchetypeVersion=1.4
```

* 根据提示输入 `groupId`、`artifactId` 和 `version`，生成一个空的模板项目。

**二、 配置模板结构**
=============

模板的核心是 `src/main/resources/archetype-resources` 目录和 `archetype-metadata.xml` 文件。

**关键文件结构**
----------

    my-custom-archetype/
    ├── pom.xml
    └── src/
        └── main/
            ├── resources/
            │   ├── archetype-resources/             # 项目模板文件目录
            │   │   ├── src/
            │   │   ├── pom.xml
            │   └── META-INF/
            │       └── maven/
            │           └── archetype-metadata.xml  # 元数据配置
            └── archetype.properties                # 可选属性

**三、 编辑调整 `archetype-metadata.xml`**
====================================

在 `archetype-metadata.xml` 中定义模板的包含文件和变量替换规则。

**示例配置**
--------

```xml
<?xml version="1.0" encoding="UTF-8"?>
<archetype-descriptor xsi:schemaLocation="http://maven.apache.org/plugins/maven-archetype-plugin/archetype-catalog/1.1.0 http://maven.apache.org/xsd/archetype-catalog-1.1.0.xsd" name="tool-archetype"
    xmlns="http://maven.apache.org/plugins/maven-archetype-plugin/archetype-catalog/1.1.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <fileSets>
    <fileSet filtered="true" packaged="true" encoding="UTF-8">
      <directory>src/main/java</directory>
      <includes>
        <include>**/*.java</include>
      </includes>
    </fileSet>
    <fileSet filtered="true" encoding="UTF-8">
      <directory>src/main/resources</directory>
      <includes>
        <include>**/*.properties</include>
      </includes>
    </fileSet>
    <fileSet encoding="UTF-8">
      <directory>src/main/resources</directory>
      <includes>
        <include>**/*.json</include>
        <include>**/*.yaml</include>
      </includes>
    </fileSet>
    <fileSet filtered="true" packaged="true" encoding="UTF-8">
      <directory>src/test/java</directory>
      <includes>
        <include>**/*.java</include>
      </includes>
    </fileSet>
    <fileSet filtered="true" encoding="UTF-8">
      <directory>src/test/resources</directory>
      <includes>
        <include>**/*.properties</include>
      </includes>
    </fileSet>
    <fileSet encoding="UTF-8">
      <directory>src/test/resources</directory>
      <includes>
        <include>**/*.json</include>
        <include>**/*.yaml</include>
      </includes>
    </fileSet>
    <fileSet encoding="UTF-8">
      <directory>.mvn</directory>
      <includes>
        <include>**/*.config</include>
      </includes>
    </fileSet>
    <fileSet encoding="UTF-8">
      <directory></directory>
      <includes>
        <include>.gitignore</include>
      </includes>
    </fileSet>
  </fileSets>
  <requiredProperties>
    <requiredProperty key="customProperty"/>
  </requiredProperties>
</archetype-descriptor>
```

* `filtered="true"`：启用变量替换（如 `${groupId}`、`${artifactId}`）。
* `packaged="true"`：自动将文件放入包路径（如 `com/example`）。

**四、自定义模板文件**
==============

在 `archetype-resources` 目录中放置项目模板文件，使用占位符变量：  


![输入图片说明](https://foruda.gitee.com/images/1747290518766801447/96c0eb5b_526602.png "屏幕截图")

**4.1 示例 `pom.xml` 模板**
-----------------------

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>


    <groupId>${groupId}</groupId>
    <artifactId>${artifactId}</artifactId>
    <version>${version}</version>
    <packaging>jar</packaging>
    <name>${artifactId}</name>
    <url>https://www.acanx.com</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.release>17</maven.compiler.release>
        <maven-compiler-plugin.version>3.14.0</maven-compiler-plugin.version>
        <fastjson2.version>2.0.54</fastjson2.version>
        <lombok.version>1.18.36</lombok.version>
        <junit-jupiter.version>5.12.2</junit-jupiter.version>
    </properties>

    <dependencies>
<!--        <dependency>-->
<!--            <groupId>com.alibaba.fastjson2</groupId>-->
<!--            <artifactId>fastjson2</artifactId>-->
<!--            <version>${fastjson2.version}</version>-->
<!--        </dependency>-->
<!--        <dependency>-->
<!--            <groupId>org.projectlombok</groupId>-->
<!--            <artifactId>lombok</artifactId>-->
<!--            <version>1.18.36</version>-->
<!--            <scope>provided</scope>-->
<!--        </dependency>-->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>${junit-jupiter.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>



    <dependencyManagement>
    </dependencyManagement>


    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.14.0</version>
            </plugin>
        </plugins>
    </build>
</project>

```

**4.2 Java 类模板**
----------------

* App.java

```java
#set( $symbol_pound = '#' )  
#set( $symbol_dollar = '$' )  
#set( $symbol_escape = '\' )  
package ${package};  
  
public class App {  
    public static void main(String[] args) {        System.out.println("Custom Property: ${customProperty}");    }}  
```

**4.3 Java 单元测试类模板**
--------------------

* AppTest.java

```java
#set( $symbol_pound = '#' )  
#set( $symbol_dollar = '$' )  
#set( $symbol_escape = '\' )package ${package};  
  
  
import org.junit.jupiter.api.Assertions;  
import org.junit.jupiter.api.Test;  
  
/**  
 * Unit test for simple App. */public class AppTest {  
  
    /**  
     * Rigorous Test :-)     */    @Test  
    public void shouldAnswerWithTrue() {  
        Assertions.assertTrue(true);  
    }  
  
}
```

4.4资源文件
-------

此处可以自定义自己需要添加的配置文件，比如日志、数据库、应用、单元测试配置文件

**4.5. 高级配置（可选）**
-----------------

* **动态变量** ：在 `archetype.properties` 中支持定义默认值。
* **多模块模板** ：通过多个 `fileSet` 和子模块配置支持多模块项目。
* **按条件生成文件** ：可以结合 Velocity 模板引擎（`.vm` 文件）实现逻辑控制。

**五、 安装及部署模板**
==============

**5.1 安装到本地仓库**
---------------

在模板项目根目录运行Maven命令：

```bash
mvn clean install
```

![输入图片说明](https://foruda.gitee.com/images/1747290633810551522/6ea2e383_526602.jpeg "254.jpg")

* 生成的 Archetype 会安装到本地 Maven 仓库（默认位置`~/.m2/repository`）。

**5.2. 将模板部署到私有仓库**
-------------------

若要部署到私有仓库，在 `pom.xml` 中配置仓库地址后运行以下命令：

```bash
mvn deploy
```

**六、 使用自定义模板来生成新Maven模块**
=========================

6.1 生成新Maven模块\*\*
------------------

```bash
mvn archetype:generate -DarchetypeGroupId=pkg-group  -DarchetypeArtifactId=custom-archetype-id -DarchetypeVersion=1.0.0  -DgroupId=com.example   -DartifactId=new-project -Dversion=1.0.0  -DcustomProperty=hello  
  
mvn archetype:generate -DarchetypeGroupId=com.acanx.java.tool  -DarchetypeArtifactId=tool-archetype -DarchetypeVersion=0.1.6 -DgroupId=com.acanx.java.tool -DartifactId=tool-scf -Dversion=0.2.2
```

![输入图片说明](https://foruda.gitee.com/images/1747290699328149765/91369f14_526602.jpeg "255.jpg")

通过以上步骤，可以实现一个高度定制化的 Maven 项目模板，来统一自己或者团队内的项目结构和依赖管理。
