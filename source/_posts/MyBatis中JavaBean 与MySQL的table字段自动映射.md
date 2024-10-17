---
title: Markdown文章样式HTML
date: 2018-09-22 20:55:48
categories: "默认" #文章分類目錄 可以省略
tags: #Java 
     - MyBatis
     - JavaBean
     - MySQL
     - 自动映射
description: MyBatis中JavaBean 与MySQL的table字段自动映射
---



---


# MyBatis自动映射

 在简单的场景下，MyBatis可以替你自动映射查询结果。 如果遇到复杂的场景，你需要构建一个result map。 但是在这里你将看到，你也可以混合使用这两种策略。 
我们再深入看看自动映射是怎样工作的。当自动映射查询结果时，MyBatis会获取sql返回的列名并在java类中查找相同名字的属性（忽略大小写）。 这意味着如果Mybatis发现了ID列和id属性，Mybatis会将ID的值赋给id。通常数据库列使用大写单词命名，单词间用下划线分隔；而java属性一般遵循驼峰命名法。 为了在这两种命名方式之间启用自动映射，需要将 mapUnderscoreToCamelCase设置为true。自动映射甚至在特定的result map下也能工作。在这种情况下，对于每一个result map,所有的ResultSet提供的列， 如果没有被手工映射，则将被自动映射。自动映射处理完毕后手工映射才会被处理。 在接下来的例子中， id 和 userName列将被自动映射， hashed_password 列将根据配置映射。


### UserMapper.xml中的配置：
```xml
 <resultMap id="userResultMap" type="User">
  <result property="password" column="hash_password" />
  <result column="gmt_create" jdbcType="TIMESTAMP" property="gmtCreate" javaType="java.util.Date" />
  <result column="gmt_modified" jdbcType="TIMESTAMP" property="gmtModified" javaType="java.util.Date" />
 </resultMap>
```



### mybatis-config.xml中的配置：
```xml
<configuration>     
 <settings>       
  <setting name="mapUnderscoreToCamelCase" value="true" />
 </settings>  
</configuration>
```



# 标题一

## 标题二


## 英语听力考试试音时间

小车正穿行在洛基山脉蜿蜒曲折的盘山公路上，克里朵托夫里维静静地看着窗外，发现车子每当行驶到无路的关头，

路边都会出现一块交通指示牌：“前方转弯”或写着“注意急转弯”。而转过每一道弯道之后前方照例又是一片柳暗花明、豁然开朗。

山路弯弯、峰回路转，“前方转弯”几个大字一次次冲击着他的眼球，也渐渐敲醒了他的心扉。

原来不是路已到了尽头，而是该转弯了……路在脚下，更在心中，心随路转，心路常宽。

学会转弯也是人生的智慧，因为挫折往往是转折，危机同时是转机。

### 代码样式


### Create a new post

#### 四级标题





# Markdown中实现首行缩进的两种方法

### 方法1
半方大的空白&ensp;或&#8194;
### 方法2
全方大的空白&emsp;或&#8195;
### 方法3
不断行的空白格&nbsp;或&#160;

---

&emsp;&emsp;量子霸权的实现，将是量子计算发展的一座重要里程碑，代表「量子计算的超强计算能力」自 37 年前提出以来首次从理论走进实验，标志一个新的计算能力飞跃时代的开始。近年来，随着「实现量子霸权」的日益临近，「称霸标准」成为量子计算领域最重要的科学问题之一。

我国科学家最早开启了「称霸标准」问题的研究。最近，《国家科学评论》（National Science Review）以「A Benchmark Test of Boson Sampling on Tianhe-2 Supercomputer」为题正式发表了国防科技大学吴俊杰团队与上海交通大学金贤敏教授的合作研究成果，报道了玻色采样案例的「称霸标准」。