---
title: 树莓派安装MySQL8.0.25数据库
date: 2021-07-27 23:12:15
categories: "树莓派" #文章分类目錄 可以省略
tags: 
     - 树莓派 
     - MySQL8
description: 树莓派安装MySQL8.0.25数据库  #本页的描述 可以省略
---




## 1.安装  
安装mysql-server 

```bash
sudo apt install mysql-server
```

#另外自己看下数据库配置是不是mysql_native_password方式，如果不是就在my.cnf文件中加上。



```
#skip-name-resolve   // 禁用DNS查询

[mysqld]
#skip-grant-tables   // 允许免密登录（去掉前面的#，然后重启Mysql服务后生效）
default_authentication_plugin = mysql_native_password
```



## 2 授权
 免密登录(重置密码遇到ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using passwor:yes)问题)
文章转载自：https://www.cnblogs.com/gumuzi/p/5711495.html
一般这个错误是由密码错误引起，解决的办法自然就是重置密码。
假设我们使用的是root账户。
1.重置密码的第一步就是跳过MySQL的密码认证过程，方法如下：

```
#vim /etc/my.cnf           // (注：windows下修改的是my.ini)
```

在文档内搜索mysqld定位到[mysqld]文本段：
/mysqld(在vim编辑状态下直接输入该命令可搜索文本内容)
在[mysqld]后面任意一行添加“skip-grant-tables”用来跳过密码验证的过程，如下图所示：

```
[mysqld]
skip-grant-tables
```


保存文档并退出

2.接下来我们需要重启MySQL：

```
/etc/init.d/mysql restart(有些用户可能需要使用/etc/init.d/mysqld restart)
```

3.重启之后输入mysql即可进入mysql。 

4.接下来就是用sql来修改root的密码
进入到终端当中，敲入 mysql -u root -p 命令然后回车，当需要输入密码时，直接按enter键，便可以不用密码登录到数据库当中

```
mysql> ALTER USER ‘root’@’%’ IDENTIFIED WITH mysql_native_password BY ‘12345678’;
mysql> flush privileges;
mysql> quit
```

注意：如果在执行该步骤的时候出现ERROR 1290 (HY000): The MySQL server is running with the --skip-grant-tables option so it cannot execute this statement 错误。则执行下 flush privileges 命令，再执行该命令即可。
到这里root账户就已经重置成新的密码了。
5.编辑my.cnf,去掉刚才添加的内容，然后重启MySQL。大功告成！
 网上有很多关于这个问题的解决说明，很多刚接触的朋友可能比较迷惑的是在自己的平台上找不到my.cnf或者my.ini文件，如果你是Linux,使用如下方式可以搜索到：

```
GRANT ALL PRIVILEGES ON *.* TO 'acanx'@'%' WITH GRANT OPTION;
```

## 3.Ubuntu下配置Mysql允许远程连接

- [Ubuntu下设置mysql允许远程连接](https://blog.csdn.net/u011120720/article/details/51096695?utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control)

默认情况下，mysql只允许本地连接，如果需要开启远程连接，则需要修改配置。

首先打开配置文件进行修改
```
cd /etc/mysql/mysql.conf.d/
vim mysqld.cnf
```

找到bind-address    =   127.0.0.1

```//注释掉如下
#bind-address   =   127.0.0.1
```

保存退出后，使用如下命令，输入密码后进入mysql
```
mysql -u username  -p
```

为需要远程登录的用户赋予权限

```
mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '12345678';
mysql> flush privileges;    // 刷新使设置立即生效
```

-- 查看权限是否已授予

```
mysql> SELECT host,user,select_priv,insert_priv,delete_priv,update_priv,drop_priv FROM user;
mysql>select user,host,authentication_string from mysql.user;
mysql->exit;  // 退出
sudo /etc/init.d/mysql restart
```







