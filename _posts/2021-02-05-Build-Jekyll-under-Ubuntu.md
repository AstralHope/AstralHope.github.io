---
layout: post
title: "Build Jekyll under Ubuntu"
subtitle: '过程记录'
author: "Hux"
header-style: text
tags:
  - Blog
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
bundle exec jekyll serve
bundle exec jekyll serve --host=0.0.0.0
```
访问`http://IP:4000`，访问到初始页面即完成初始设置
## GitHub选取想要的博客主题进行修改
我这里选择[黄玄](https://github.com/Huxpro/huxpro.github.io)的主题进行修改。

