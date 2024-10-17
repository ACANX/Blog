---
title: Java运算符
date: 2018-06-27 20:55:48
categories: "Java" #文章分類目錄 可以省略
tags: #Java 
     - Java
     - 运算符
description: Java基本运算符  #你對本頁的描述 可以省略
---


## java 原码、反码、补码计算 以及取反(~)运算

### 1.原码、反码、补码：
     (1)在Java中，所有数据的表示方式都是以补码形式来表示
     (2)正数：原码、反码、补码相同
     (3)负数：符号位为1，其余各位是对原码取反，然后整个数加1
     (4)~按位取反（反码加1称为补码。）步骤就是先求出这个数（因为java存的数是补码）的原码，然后对原码取反得到X，这个X就是我们要求的那个数的补码

### 2.取反(~)运算
     （1）n=37 ，二进制数就是 100101
     因为在Java中，所有数据的表示方式都是以补码形式来表示，如果没有特别的说明，
     Java 中的数据类型默认为int，int数据类型的长度为8位，就是32字节，32bit的意思，因此，n=100101的原码=补码（因为是正数，所以原=补=反）运算过程就是：
     原码：00000000 00000000 00000000 00100101 ＝37


     ~n（对n的原码）取反运算得：  11111111 11111111 11111111 11011010很明显，最高位是1，意思是原码为负数，负数的补码是其绝对值的原码取反，末尾再加1，因此，我们可将这个二进制数的补码进行还原：
     首先，末尾减1得反码：11111111 11111111 11111111 11011001
     其次，将各位取反得原码：00000000 00000000 00000000 00100110这个就是~n的绝对值形式，|~n|=38   ，

     所以，~n＝－38，这个就是Java虚拟机的运算结果


     （2）n=-4，取反 （~-4）
     -4补码：10000000 00000000 00000000 00000100
     -4反码：10000000 00000000 00000000 00000011
     -4原码：11111111 11111111 11111111 11111100
     对原码取反：00000000 00000000 00000000 00000011  （3）
     因为是正数，所以补码等于原码等于反码=3，所以~-4=3


### Create a new post

``` bash
deploy:
  - type: git
    repo: # 像这样设置多个 git 仓库，`名称: 地址,分支`，逗号后面没有空格。
      github: git@github.com:XXXXXX/XXXXXX.git,branch
      coding: git@git.coding.net:XXXXXXX/XXXXXXX,coding-pages
    message: Site updated by Hexo at {{ now('YYYY-MM-DD HH:mm:ss') }}.
  - type: rsync
    host: YOUR VPS IP # 你的服务器的 IP 地址
    user: YOUR USERNAME # 你刚刚复制密钥的那个用户
    root: YOUR DESTINATION # 你想把文件上传到哪里，比如我的是 `~/stackharbor.alynx.xyz/`
    port: 22 # 这是 ssh 默认的端口，如果你修改了，这里也要改
    args: --progress # 额外的 rsync 参数，我这里添加了一个进度条参数，你也可以不设置
    delete: true # 是否删除旧的文件
    verbose: true # 是否同步时显示详细状态
    ignore_errors: false # 忽略错误
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/deployment.html)


