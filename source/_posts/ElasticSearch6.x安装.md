---
title: ElasticSearch6.x及Elasticsearch-Head插件的安装
date: 2018-09-05 20:10:00
categories: "ElasticSearch" #文章分类目录 可以省略
tags: #Java 
     - Java
     - ElasticSearch
     - Elasticsearch-Head
description: ElasticSearch6.x及elasticsearch-head插件的安装  #你對本頁的描述 可以省略
---


# ElasticSearch6.x及elasticsearch-head插件的安装


## 安装环境
- Windows10企业版/专业版 X64
- JDK10.0.1
- ElasticSearch-5.0.0
- Node-v8.11.3-x64.msi
- Git客户端
- Powershell


## 1.下载相关必备软件：
- Windows系统，不多废话。
- ES官网下载页面：https://www.elastic.co/downloads/elasticsearch
- ES6.4 Windows版下载链接：https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.4.0.zip

- JDK 安装略
- Node 安装略  
    - 设置环境变量，把NODE_HOME设置到环境变量里(安装包也可以自动加入PATH环境变量)。测试一下node是否生效：
    - Powershell/CMD测试命令：node -v
    - ![9200-default](/Blog/static/image/20180905/node-v.png)
- Git 安装略




## 2安装ElasticSearch
### 2.1安装
将1中下载的elasticsearch-6.4.0.zip压缩包解压，打开压缩包的bin目录，点击elasticsearch.bat批处理文件就可以默认启动了，安装过程可以说非常简单。
访问:http://localhost:9200，如下所示：
![9200-default](/Blog/static/image/20180905/9200-default.png)

### 2.2命令行下启动
在Powershell中开启ES-6.x，直接执行bat文件：
D:\ProgramFiles\ElasticSearch\elasticsearch-6.4.0\bin\elasticsearch.bat
也在系统的环境变量中将路径"D:\ProgramFiles\ElasticSearch\elasticsearch-6.4.0\bin\"添加到"Path"中
可以直接使用如下命令运行：
```cmd
elasticsearch.bat
```
启动效果：
![9200-default](/Blog/static/image/20180905/ES启动.png)

## 3.elasticsearch-head插件安装
### 3.1安装grunt
grunt是一个很方便的构建工具，可以进行打包压缩、测试、执行等等的工作，6.x里的head插件需要通过grunt启动。因此需要安装grunt：

- 设置全局安装(-g代表全局安装,安装在D:\ProgramFiles\NodeJs\node_modules\目录下)：
```cmd
npm install -g grunt-cli
```

### 3.2下载elasticsearch-head插件的源码
源码地址：https://github.com/mobz/elasticsearch-head.git
#### 2.2.1使用git clone命令下载下来：
```
git clone git://github.com/mobz/elasticsearch-head.git
```
效果如图：
![9200-default](/Blog/static/image/20180905/git-clone.png)

### 3.3配置elasticsearch-head
由于head的代码还是旧版本的，直接执行有很多限制，比如无法跨机器访问。因此需要修改几个地方：

#### 3.3.1修改elasticsearch-head源码配置-Gruntfile.js

①目录：elasticsearch-head/Gruntfile.js，增加hostname属性，设置为*：

```js
connect: {
    server: {
        options: {
            port: 9100,
            hostname: '*',
            base: '.',
            keepalive: true
        }
    }
}
```
#### 3.3.2修改elasticsearch-head源码配置-app.js
②修改elasticsearch-head的连接地址
目录：elasticsearch-head/_site/app.js
修改连接地址:
```js
this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://localhost:9200";
```
把localhost修改成你es的服务器地址，如：
```js
this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://127.0.0.1:9200";
```

### 3.4添加Elasticsearch的配置参数

修改Elasticsearch使用的参数。编辑elasticsearch-6.4.0/config/elasticsearch.yml配置文件，在文件末尾追加如下配置：
```yml
# 设置集群的名字，对应为http://127.0.0.1:9200/中的"cluster_name" : "acanx-es-demo",
cluster.name: acanx-es-demo
# 节点名字，对应为http://127.0.0.1:9200/中的"name" : "node-1001",
node.name: node-1001
node.master: true
node.data: true
# 修改ES的监听地址，以便其他机器也可以访问
network.host: 0.0.0.0
# 默认端口
http.port: 9200
# 增加新的CORS参数，这样head插件可以访问ES
http.cors.enabled: true
http.cors.allow-origin: "*"
```
- 修改完成后记得Save保存！
- 特别注意，设置参数的时候:后面要有空格！格式不规范在启动ES时会抛异常，配置时注意检查书写格式

### 3.5启动ElasticSearch
   操作同上第二节

```cmd
elasticsearch.bat
```


### 3.6再运行elasticsearch-head
 在elasticsearch-head源码目录下，执行npm install 下载的包：
```
 npm install
```
效果如图：
![9200-default](/Blog/static/image/20180905/npm-install.png)

- 初次运行安装可能会报警告或错误。可以重新运行一次npm install。最后，在head源代码目录下启动Nodejs：

```
grunt server
```

效果如图：
![9200-default](/Blog/static/image/20180905/grunt-server.png)


访问:http://localhost:9100
这个时候，访问http://localhost:9100就可以使用elasticsearch-head插件查看ES中的数据了了：
![9200-default](/Blog/static/image/20180905/9100-port.png)





## 参考链接

- http://www.cnblogs.com/xuxy03/p/6039999.html
- https://blog.csdn.net/fwhezfwhez/article/details/79380275
- https://blog.csdn.net/qq_40454655/article/details/79291106
- https://blog.csdn.net/sinat_28224453/article/details/51516061