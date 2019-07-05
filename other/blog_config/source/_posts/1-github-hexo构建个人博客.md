---
title: 1.github+hexo构建个人博客
brief: 副标题
key: "github+hexo,个人博客"
date: 2019-06-13 14:12:45
tags:
	- 未分类
toc: true
comments: true
---

第一篇就写你吧……
![](/pic/unclassified/hexo.jpg)

<!-- more -->

# 安装Node.js
具体安装过程此处不赘述，见[官方文档](https://nodejs.org/en/)，安装完成后执行下面命令验证是否安装成功：
```
>node -v
v8.7.0
```

# 安装Git
这个过程就不赘述了，见[官方文档](https://git-scm.com/)，安装完成后执行下面命令验证是否安装成功：
```
>git --version
git version 2.12.2.windows.2
```

# 安装Hexo

## 安装
按顺序执行下面命令，进行安装：
```
npm install hexo-cli -g
hexo init blog
cd blog
npm install
hexo server
```
<font color=red>注</font>：blog是文件夹名字，根据需要自行修改。

按照上面步骤操作，没有报错，正常安装完成后，访问下面地址，能打开说明安装成功。  
[http://localhost:4000/](http://localhost:4000/)

blog目录中的文件及目录结构（只列了第一级目录）如下所示：
```
E:\blog
│
├─node_modules
├─public
├─scaffolds
├─source
├─themes
├─config.yml
├─db.json
├─package.json
└─package-lock.json
```

## 主题设置
默认的主题有点丑，换一个主题吧，以[hexo-theme-yilia](https://github.com/litten/hexo-theme-yilia.git)主题为例，[更多主题……](https://github.com/search?q=hexo+theme)

### 下载主题
操作步骤如下：
```
cd blog/themes
hexo clean
git clone https://github.com/litten/hexo-theme-yilia.git yilia
```

### 修改配置
修改blog目录下的_config.yml配置文件。
1. 将theme属性，设置为yilia。
2. 在文件最后添加如下配置，显示文章目录。
```
jsonContent:
  meta: false
  pages: false
  posts:
    title: true
    date: true
    path: true
    text: false
    raw: false
    content: false
    slug: false
    updated: false
    comments: false
    link: false
    permalink: false
    excerpt: false
    categories: false
    tags: true
```
生成目录：
```
npm i hexo-generator-json-content --save
```

### 验证主题
启动本地web服务器：
```
hexo server
```

访问[http://localhost:4000/](http://localhost:4000/)，查看新主题效果。

# 写文章
文章是markdown文档
1. 创建文章
执行命令：
```
hexo new "文章标题"
```
然后会在source/_posts路径下生成markdown文件。  
写文章就是按markdown语法编辑此文件。  

如果文章太长，可以在文章中任意你想截断的位置加上如下描述：
```
<!-- more -->
```
文章会在此处截断，文章列表页显示截断之前的内容，点击“查看全文”后显示全部内容。

# 发布博客
## 创建github账户并配置
1. 注册账号，[传送门……](https://github.com/)
2. 创建仓库
    Repository name必须是这种形式：{username}.github.io 
3. 配置hexo
    修改blog目录下的_config.yml配置文件中的deploy项为如下形式：
    ```
    deploy:
       type: git 
       repo: {上面创建的仓库地址，如：https://github.com/username/username.github.io.git}
       branch: master
       message: "git commit时的备注信息，自行调整"
    ```

## 发布
执行下面命令，发布博客，其实就是生成博客页面，然后上传到github仓库中。
```
hexo clean && hexo deploy
```
提交过程中会提示输入github用户名和密码。

# 访问博客

[传送门](https://forvoyager.github.io/)







