---
title: 将JAR包发布到Maven中央仓库
date: 2021-03-20 23:10:00
categories: "Maven" #文章分类目录 可以省略
tags: #Java 
     - Maven
     - Sonatype
     - JAR
description: 将JAR包发布到Maven中央仓库操作过程记录  #
---



# 最新操作教程

- [发布jar包到maven中央仓库，完整记录，规避一些不必要的坑](https://blog.csdn.net/qq_41813208/article/details/112370194)
- [https://oss.sonatype.org/](https://oss.sonatype.org/)
- [https://issues.sonatype.org/](https://issues.sonatype.org/)
- []()



# 问题记录



#### 1.错误提示：Cannot run program "gpg": CreateProcess error=2

```java

Failed to execute goal org.apache.maven.plugins:maven-gpg-plugin:1.6:sign (sign-artifacts) on project dubbo-spring-boot-starter: Unable to execute gpg command: Error while executing process. Cannot run program "gpg": CreateProcess error=2, 系统找不到指定的文件。 -> [Help 1]
```

解决办法：   检查windows控制台中能否执行**gpg --version** 命令，如不能，需检查操作系统的环境变量**Path**的配置是否存在问题，GnuPG的安装路径下的bin目录是否已添加到Path变量中，若未添加，需手动添加进去。



**影响** 如果未配置成功，在发布时会无法执行签名校验，后续的源码包、文档包均无法发布上传到sonatype的仓库中，会影响jar包发布



### 2.Maven执行install时GPG Passphrase的解决办法

mvn install后加上参数**-Dgpg.skip**，例如：mvn install **-Dgpg.skip**
另外参数-DskipTests可以跳过test阶段。





## 仍然存在的问题

- 添加项目logo到https://mvnrepository.com/网站([how-to-create-a-logo-for-mvnrepository-artifact](https://stackoverflow.com/questions/40197177/how-to-create-a-logo-for-mvnrepository-artifact))
- 如何避免进入https://mvnrepository.com/网站需要进行人工验证码校验（官方出的插件安装后任然需要验证，还是需要点击验证）
- 



### 参考链接

- [https://mvnrepository.com/](https://mvnrepository.com/)
- [GPG入门教程](http://www.ruanyifeng.com/blog/2013/07/gpg.html)
- [GPG下载](https://www.gpg4win.org/thanks-for-download.html)
- [如何把jar发布到中央仓库](https://blog.csdn.net/huangjinjin520/article/details/78915789)
- [如何发布jar包到maven中央仓库详细教程](https://blog.csdn.net/qq_36838191/article/details/81027586)
- [使用gpg插件发布jar包到Maven中央仓库 完整实践](https://blog.csdn.net/alinyua/article/details/83687250)
- [https://search.maven.org/](https://search.maven.org/search?q=a:acanx-utils)
- [https://repo1.maven.org/maven2/com/acanx/acanx-utils/](https://repo1.maven.org/maven2/com/acanx/acanx-utils/)

