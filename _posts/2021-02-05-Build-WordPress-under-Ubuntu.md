---
layout: post
title: "WordPress建站记录"
subtitle: ''
date: 2020-04-24 23:33:00
author: "突击核"
header-style: text
hidden: false
tags:
  - Blog
  - WordPress
  - ubuntu
---
# Ubuntu下搭建Jekyll博客完整过程记录
**使用环境**：Ubuntu 18.04 x86_64   
**目标**：在Ubuntu服务器上搭建Jekyll，使用Nginx代理并开启https，保持与GitPage同步更新    
## 准备工作：

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

6. 修改`pwa`目录下的json文件和图标为自己的；修改`sw.js`里引用的图像为你自己想用的

7. 在`_posts/`下存放markdown格式的文章即可在博客中显示，注意文件名必须是形如`2020-02-05-君の名は`的形式，最后的文章路径会是xxx.io/2020/02/05/君の名は的形势，这里的时间其实是以下面yaml的时间为准，文章名字用英文的话可以链接的书写，也可以是其他字符。

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
首先使用前面创建的用户执行`git clone`博客到你打算运行的目录，然后进入该目录，执行`bundle exec jekyll serve --host=0.0.0.0`

按提示执行`gem install jekyll-paginate`后再次运行上面的命令
访问`http://IP:4000`查看效果

接着安装好Nginx、申请好SSL证书开始部署https访问

这里我使用lnmp完成，目标域名为`blog.39hope.com`，创建好vhost后修改配置文件，使所有访问使用https访问并转发到4000短款，配置文件如下

```
server {
  listen         80;
  server_name    blog.39hope.com;
  rewrite        ^   https://$server_name$request_uri? permanent;
}
server {  
  listen                 443 ssl http2;
  server_name            blog.39hope.com;
  ssl                    on;
  ssl_certificate        /usr/local/nginx/****/fullchain.cer;
  ssl_certificate_key    /usr/local/nginx/****/blog.39hope.com.key;
  location / {
      proxy_set_header   X-Real-IP        $remote_addr;
      proxy_set_header   Host             $http_host;
      proxy_set_header   X-Frame-Options  DENY;
      proxy_pass         http://127.0.0.1:4000;
  }
}
```
完成后回到博客所在目录执行`screen -S blog`

然后执行`bundle exec jekyll serve`

最后使用域名访问，可以正常访问则配置成功。退出ssh使jekyll保持在后台运行
## 使用webhook自动更新
然后配置自动更新，再新建一个vhost，用于访问webhook的url,这里以webhook.php举例，首先编辑php.ini,搜索disable_functions删掉shell_exec。编辑`php-fpm.conf`，将user和group修改为前面准备的用户。完成后执行`service php-fpm restart`重启php。lnmp的安装目录位为`/usr/local/php/etc`

编写一个脚本用于更新制定目录，我的脚本如下

```php
<!--webhook.php-->
<?php
/**
* 记得在此处追加webhook验证，可不敢裸奔啊
*
*/

//执行shell命令并把返回信息写进日志
shell_exec("cd /home/wwwroot/blog.39hope.com  && date >> log.txt  &&  /home/wwwroot/blog.39hope.com/update.sh");
?>

```

```shell
#!/bin/bash
cd astralhope.github.io/
git pull >> /home/wwwroot/blog.39hope.com/log.txt
#bundle exec jekyll serve &
```

完成后每次更新git push到GitHub后服务器上就会自动更新

##存在的问题
1. 系统重启后需要手动执行`screen -S blog`和`bundle exec jekyll serve`启动服务
2. 公式、流程图的支持还在探索阶段，支持情况可以查看[Markdown语法](https://blog.39hope.com/2020/06/04/Markdown-grammar/)中哪些已可正常显示验证。

解决办法参考：
1. [ubuntu开机以指定用户身份执行脚本](https://www.jianshu.com/p/4f7416ae1cd7)

2. [jekyll-mermaid](https://rubygems.org/gems/jekyll-mermaid/versions/1.0.0)
