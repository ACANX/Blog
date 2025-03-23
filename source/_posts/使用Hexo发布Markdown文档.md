---
title: 使用Hexo发布Markdown文档
date: 2025-03-20 12:00:00
categories: "Markdown" #文章分类目录 可以省略
tags: 
     - Hexo
     - Markdown
     - 静态站点生成器
description: 使用Hexo编写技术文档，常用操作说明  #
---

Hexo 是一个快速、简洁且高效的博客框架。 Hexo 使用 [Markdown](http://daringfireball.net/projects/markdown/)（或其他标记语言）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。


## 快速开始

### 新建一篇文章

命令格式

``` bash
$ hexo new [layout] <title>
```

如果没有设置 layout 的话，默认使用 _config.yml 中的 default_layout 参数代替。 使用布局 draft 来创建草稿。 如果标题包含空格的话，请使用引号括起来。


``` bash
$ hexo new "My New Post"
```

更多信息参考: [Writing](https://hexo.io/docs/writing.html)



### 启动服务器


``` bash
$ hexo server
```

默认情况下，访问网址为： http://localhost:4000/。更多信息参考: [Server](https://hexo.io/docs/server.html)



### 生成静态文件


``` bash
$ hexo generate
```

更多信息参考: [Generating](https://hexo.io/docs/generating.html)



### 部署站点

``` bash
$ hexo deploy
```

更多信息参考: [Deployment](https://hexo.io/docs/deployment.html)


### 更多推荐的主题

- [hexo-theme-aria](https://github.com/AlynxZhou/hexo-theme-aria/)



### 获取帮助

使用 [Hexo](https://hexo.io/)! 使用快速上手 [文档](https://hexo.io/docs/) ，遇到问题可查看 [troubleshooting](https://hexo.io/docs/troubleshooting.html) 中的解答，或者在 [GitHub](https://github.com/hexojs/hexo/issues) 上提问。

