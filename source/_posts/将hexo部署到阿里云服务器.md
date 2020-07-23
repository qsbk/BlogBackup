---
title: 将hexo部署到阿里云服务器
tags:
  - hexo
  - blog
  - 优化
abbrlink: 707f
date: 2020-05-03 23:19:48
---

访问速度和SEO一直是Hexo+GitHub搭建博客的痛点。其中访问速度来说，尽管图片等文件放到了阿里云oss，但一篇文章内容稍多访问速度就被拖到了20秒左右，这实在是没办法容忍的事情。起初我打算换成WordPress，折腾了几天，还是不好用而且对于我这种只是单纯的写博客来说，还是hexo更轻量好用一些，而且真的是大爱markdown。回到正题，看看成果：<!--more-->

![](https://qsbk-blog.oss-cn-beijing.aliyuncs.com/img/对比.PNG)

就我的体验而言，从20s -> 3s，也算是满足了我的基本需求。

## 搭建流程

先来了解下在`hexo d -g`时 hexo做了些什么，这样有助于我们理解接下来的流程。

第一步：hexo会将本地的md文件渲染为静态文件

第二步：调用Git推到服务器的仓库区

第三步：服务器通过git-Hooks同步到网站根目录



hexo本地环境配置及初始化这里不再赘述，其中服务器的环境需要安装`Git`,`Nginx`如果安装了BT面板可以直接创建一个站点不需要修改`Nginx`的配置文件

### 安装Git

```shell
yum install git
```

- 检查安装`git --version`

### 创建git用户

```shell
adduser git
```

vim打开sudo配置文件

```shell
chmod 740 /etc/sudoers
vim /etc/sudoers
```

找到以下内容

```shell
## Allow root to run any commands anywhere
root    ALL=(ALL)     ALL
```

在root用户后添加一行，赋予git用户权限，需要注意对齐

```shell
git     ALL=(ALL)     ALL
```

:切换命令模式输入wq保存并退出，记得改回sudo文件的权限` chmod 400 /etc/sudoers `

设置git用户密码

```shell
#需要root权限
sudo passwd git
```

### 配置ssh

切换git用户， 创建 ~/.ssh 文件夹

```shell
su git
mkdir ~/.ssh
```

打开本地电脑用户文件夹下的**.ssh**文件夹

如果你已经生成**ssh-keygen**里面会有两个文件**id_rsa.pub**和**id_rsa**

我们将本地的公钥也就是**id_rsa.pub**文件使用FTP或者BT面板上传到服务器git用户的**.ssh**文件夹中

将文件复制一份并配置权限

```shell
cd ~/.ssh
cp id_rsa.pub authorized_keys
chmod 600 ~/.ssh/authorzied_keys
chmod 700 ~/.ssh
```

- 使用` ssh -v git@SERVER `（server是你的服务器公网IP）测试是否能够连通

### 创建仓库

执行命令创建一个裸仓库

```shell
sudo git init --bare blog.git
```

> 使用–bare 参数，Git 就会创建一个裸仓库，裸仓库没有工作区，我们不会在裸仓库上进行操作，它只为共享而存在。

改变仓库拥有者为git

```shell
sudo chown -R git:git blog.git
```

### 配置钩子

 在 `blog.git/hooks` 目录下新建一个 `post-receive` 文件并编辑

```shell
vi ~/blog.git/hooks/post-receive
```

在`post-receive`文件中写入

```shell
#!/bin/bash
git --work-tree=/www/wwwroot/blog --git-dir=/home/git/blog.git checkout -f
```

其中`/www/wwwroot/blog`路径是你的网站根目录`/home/git/blog.git`是你的仓库目录

赋予`post-receive`执行权限` chmod +x ~/blog.git/hooks/post-receive `

**至此服务端配置结束~**

### 配置_config.yml

放在阿里的服务器国内访问会很快，但是国外就无法访问了，或速度很慢。作为一个国际化的coder（假装会有外国人看），所以我保留了GitHub的仓库。

```yaml
deploy:
  type: 'git'
  repository: 
    github: 你的GitHub仓库SSh
    aliyun: git@你的服务器公网IP:/home/git/blog.git
  branch: master
```

这样`hexo d -g`会同时push到两个仓库中。需要注意的是如果`hexo d -g`命令执行成功，服务器的网站根目录却没有生成任何静态文件，那么需要手动执行一遍

```shell
git --work-tree=/www/wwwroot/blog --git-dir=/home/git/blog.git checkout -f
```

最后将境外的线路解析到GitHub，将国内解析到阿里，这样我们就是一个~~国际化的coder~~了&emsp;大误



___end___