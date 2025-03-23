---
title: 使用MkDocs发布Markdown文档
date: 2025-03-20 13:00:00
categories: "Markdown" #文章分类目录 可以省略
tags:  
     - MkDocs
     - Markdown
     - 静态站点生成器
description: 使用MkDocs编写技术文档，常用操作说明  #
---

[MkDocs](https://www.mkdocs.org/) 是一款快速、简单且极其出色的静态站点生成器，旨在构建项目文档。文档源文件以 Markdown 编写，并使用单个 YAML 配置文件进行配置。首先阅读[入门教程](https://www.mkdocs.org/getting-started/)，然后查看 [用户指南](https://www.mkdocs.org/user-guide/)以了解更多信息。


# 安装

要安装 MkDocs，请从命令行运行以下命令：

```bash
pip install mkdocs
```

有关详细信息，请参阅安装指南。

# 创建新项目

入门非常简单。要创建新项目，请从命令行运行以下命令：

```bash
mkdocs new my-project
cd my-project
```

请花一点时间来检查一下为您创建的初始项目。

![layout](https://www.mkdocs.org/img/initial-layout.png)


## 初始 MkDocs 布局

有一个名为 的配置文件mkdocs.yml，以及一个名为 的文件夹 docs，其中包含您的文档源文件（是docs_dirdocs配置设置的默认值）。目前，该 文件夹仅包含一个名为 的文档页面。docsindex.md

MkDocs 带有内置开发服务器，可让您在处理文档时预览文档。确保您与mkdocs.yml 配置文件位于同一目录中，然后通过运行以下命令启动服务器mkdocs serve ：

```bash
$ mkdocs serve
INFO    -  Building documentation...
INFO    -  Cleaning site directory
INFO    -  Documentation built in 0.22 seconds
INFO    -  [15:50:43] Watching paths for changes: 'docs', 'mkdocs.yml'
INFO    -  [15:50:43] Serving on http://127.0.0.1:8000/
```

在浏览器中打开http://127.0.0.1:8000/，您将看到显示的默认主页：

![screenshot](https://www.mkdocs.org/img/screenshot.png)

## MkDocs 实时服务器

开发服务器还支持自动重新加载，并且只要配置文件、文档目录或主题目录中的任何内容发生变化，就会重建您的文档。

在您选择的文本编辑器中打开docs/index.md文档，将初始标题更改为MkLorum，然后保存更改。您的浏览器将自动重新加载，您应该立即看到更新后的文档。

现在尝试编辑配置文件：mkdocs.yml。将设置更改 site_name为MkLorum并保存文件。

```yaml
site_name: MkLorum
```

您的浏览器应立即重新加载，并且您将看到新的网站名称生效。



```yaml
site_name 设置
```
![site-name](https://www.mkdocs.org/img/site-name.png)

注意：配置选项site_name是配置文件中唯一必需的选项。

## 添加页面

现在向您的文档添加第二页：

```bash
curl 'https://jaspervdj.be/lorem-markdownum/markdown.txt' > docs/about.md
```

由于我们的文档站点将包含一些导航标题，您可能需要编辑配置文件并通过添加设置来添加有关导航标题中每个页面的顺序、标题和嵌套的一些信息nav ：

```yaml
site_name: MkLorum
nav:
  - Home: index.md
  - About: about.md
```

保存更改，您将看到一个导航栏， 左侧有Home和项目，右侧有、和项目。AboutSearchPreviousNext

![multipage](https://www.mkdocs.org/img/multipage.png)

尝试菜单项并在页面之间来回导航。然后点击 Search。将出现一个搜索对话框，允许您搜索任何页面上的任何文本。请注意，搜索结果包括网站上搜索词的每次出现，并直接链接到搜索词出现的页面部分。您无需付出任何努力或进行任何配置即可获得所有这些！

![search](https://www.mkdocs.org/img/search.png)

## 主题化我们的文档
现在通过更改主题来更改配置文件以改变文档的显示方式。编辑mkdocs.yml文件并添加theme设置：

```yaml
site_name: MkLorum
nav:
  - Home: index.md
  - About: about.md
theme: readthedocs
```

保存您的更改，您将看到正在使用的 ReadTheDocs 主题。

![readthedocs](https://www.mkdocs.org/img/readthedocs.png)


### 更多推荐的主题

- [mkdocs-material](https://squidfunk.github.io/mkdocs-material/)


## 更改网站图标

默认情况下，MkDocs 使用MkDocs 网站图标。要使用其他图标，请在目录img中创建一个子目录docs并将您的自定义文件复制favicon.ico 到该目录。MkDocs 将自动检测并使用该文件作为您的网站图标。

## 建立网站
看起来不错。您已准备好部署文档的第一遍MkLorum 。首先构建文档：

```bash
mkdocs build
```

这将创建一个名为 的新目录site。查看目录内部：

```bash
$ ls site
about  fonts  index.html  license  search.html
css    img    js          mkdocs   sitemap.xml
```

请注意，您的源文档已输出为两个 HTML 文件，分别名为 index.html和about/index.html。您还有各种其他媒体已复制到site目录中作为文档主题的一部分。您甚至有一个sitemap.xml文件和mkdocs/search_index.json。

如果您使用源代码控制，例如git您可能不想将您的文档构建检查到存储库中。 site/在您的.gitignore文件中添加一行。

```bash
echo "site/" >> .gitignore
```

如果您正在使用其他源代码控制工具，您将需要检查其文档以了解如何忽略特定目录。

## 其他命令和选项
还有各种其他命令和选项可用。要获取完整的命令列表，请使用以下--help标志：

```bash
mkdocs --help
```

要查看给定命令的可用选项列表，请将--help标志与该命令一起使用。例如，要获取该命令可用的所有选项的列表， build请运行以下命令：

```bash
mkdocs build --help
```

# 部署

您刚刚构建的文档站点仅使用静态文件，因此您几乎可以从任何地方托管它。只需将整个site目录的内容上传到您托管网站的任何位置即可。有关一些常见主机的具体说明，请参阅 部署文档页面。


# 获取帮助

请参阅 [用户指南](https://www.mkdocs.org/user-guide/) 以获取有关 MkDocs 所有功能的更完整文档。

要获得有关 MkDocs 的帮助，请使用 [GitHub Discussions](https://github.com/mkdocs/mkdocs/discussions) 或 [GitHub Issues](https://github.com/mkdocs/mkdocs/issues)。