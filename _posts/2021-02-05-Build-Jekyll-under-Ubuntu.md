---
layout: post
title: "Ubuntu下搭建Jekyll博客完整过程记录"
subtitle: ''
date: 2021-02-08 20:00:00
author: "突击核"
header-style: text
hidden: false
tags:
  - Blog
  - Jekyll
  - ubuntu
---

# Ubuntu下搭建Jekyll博客完整过程记录
**使用环境**：Ubuntu 18.04 x86_64   
**目标**：在Ubuntu服务器上搭建Jekyll，使用Nginx代理并开启https，保持与GitPage同步更新    
## 准备工作：
安装好Nginx、申请好SSL证书

这里我使用lnmp完成，目标域名为`blog.39hope.com`

新建一个非root用户`adduser username`

赋予管理员权限`vim /etc/sudoers`，找到如下位置并添加一行
```shell
## Allow root to run any commands anywhere
root    ALL=(ALL)     ALL
username   ALL=(ALL)       ALL
```
完成后切换到该用户进行下列操作
```shell
su username
```
## 安装Jekyll
### 安装依赖
```shell
sudo apt-get install ruby-full build-essential zlib1g-dev
```
### 设置环境变量
```shell
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```
### 安装Jekyll和Bundler
```
gem install jekyll bundler
```
### 测试
新建一个站点测试是否可用,`--host=0.0.0.0`用于指定监听IP，默认只监听localhost
```shell
jekyll new my-awesome-site
cd my-awesome-site
<!--bundle exec jekyll serve-->
bundle exec jekyll serve --host=0.0.0.0
```
访问`http://IP:4000`，访问到初始页面即完成初始设置
## GitHub选取想要的博客主题进行修改
我这里选择[黄玄](https://github.com/Huxpro/huxpro.github.io)的主题进行修改。

首先**fork**到自己的GitHub仓库，主要修改注意的地方如下：

1. 将说明的链接换成自己的，Readme文件按自己喜好进行修改。因为我将博客部署到我自己的服务器上，所以不使用将域名直接解析到GitHub Pages的形式，删除掉仓库中的`CNAME`文件。

2. 根据[文档](https://github.com/Huxpro/huxpro.github.io/blob/master/_doc/README.zh.md)内容将标题等信息修改为你自己的；下方的微博、知乎等信息为你个人空间的域名后缀；侧栏仿着模版替换即可，同时换掉头像和图标`favicon.ico`

3. 注册多说[Duoshuo](http://duoshuo.com)或者[Disqus](http://disqus.com)，正确设置评论系统。按需设置百度统计和Google Analytics。

4. 我fork的版本里在底部增加了建站时间，联系方式里新增了bilibili

5. 在`_includes/about`下面修改自己版本的about me。在`index.html`中修改自己的副标题，添加自己的主页配图

6. 在`_posts/`下存放markdown格式的文章即可在博客中显示，注意文件名必须是形如`2020-02-05-君の名は`的形式，最后的文章路径会是xxx.io/2020/02/05/君の名は的形势，这里的时间其实是以下面yaml的时间为准，文章名字用英文的话可以链接的书写，也可以是其他字符。

yaml 头文件格式如下

```yaml
---
layout:     post
title:      "Hello 2015"
subtitle:   "Hello World, Hello Blog"
date:       2015-01-29 12:00:00
author:     "Hux"
header-style: text
header-img: "img/post-bg-2015.jpg"
hidden: false
tags:
    - Tag1
    - Tag2
---
```

完成后push到GitHub上继续在服务器上进行操作。
## 服务器上部署博客
这里继续使用上面创建的非root用户进行操作

首先`git clone`博客到你打算运行的目录，然后进入该目录，执行`bundle exec jekyll serve --host=0.0.0.0`

未完待续

