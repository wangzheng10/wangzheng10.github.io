---
title: windows使用hexo搭建github
date: 2021-12-12 23:12:20
tags:
toc: true
---

## 安装Nodejs
  首先进入[nodejs](https://nodejs.org/en/)官网，建议选择LTS recommended for most users版本，即被大多数用户推荐的长期支持版本。
  下载后直接双击安装包安装，一直点击下一步，安装完成后关闭。

---

## 安装hexo
  快捷键win + R，输入cmd命令，回车，进入cmd命令窗口。首先输入一下命令检查node和npm是否安装正常。

``` bash
$ node -v
$ npm -v
```
  如果正常显示版本号，则已正确安装，否则重新安装nodejs。

---
### 安装淘宝的cnpm管理器：
``` bash
$ npm install -g cnpm --registry=http://registry.npm.taobao.org
$ cnpm -v #查看cnpm版本
```
---
### 安装hexo框架：
``` bash
$ cnpm install -g hexo-cli
$ hexo -v  #查看hexo版本
```
---

## 生成本地博客
---
### 创建blog文件夹
``` bash
$ mkdir blog  #创建blog文件夹
$ cd blog  #进入blog目录
```
---
### 安装[git](https://git-scm.com/)
点击进入[git](https://git-scm.com/)官网，下载Windows版本，如果下载较慢，可以使用迅雷下载。
安装完成后，进入git bash命令窗口。

---
### 初始化博客
``` bash 
$ hexo init
```
---
### 启动本地博客服务
```bash
$ hexo s
```
本地访问地址：<https://localhost:4000/>  即可预览到初始的博客页面。

---

## 创建第一篇博客
---
首先新建md文件：
```bash 
$ hexo n "my first blog"
```
进入到博客存储路径下：
``` bash
$ cd source/_posts/
```
---
### 编辑博客
``` bash
$ vim my-first-blog.md
```
此处使用vim编辑器，在安装git时会自动安装vim编辑器，也可以使用记事本打开。
在里面编辑即可，具体markdown语法可上网查询使用。

---
### 生成博客
```bash
$ cd ../..  #退回到blog目录下
$ hexo clean  #清理缓存文件
$ hexo g  #生成博客
$ hexo s  #本地博客预览
```
刷新一下刚才的网址，即可发现多了一篇文章。

---

## 上传到远端服务器[github](https://github.com/)

首先注册一个github账号，点击进入[github](https://github.com/)官网。

---

登录github账号，新建一个仓库，点击New repository，命名仓库名称必须以你的账号名字做前缀，例如你的账户名字叫：xiaoming123，那么repository name必须是：xiaoming123.github.io。
后续可直接访问这个地址进入到自己的博客页面。
然后点击create repository，创建仓库。

---

### 安装部署插件
回到blog目录下，输入以下命令：
``` bash
$ cnpm install --save hexo-deployer-git
```

---

### 修改配置文件
``` bash
$ vim _config.yml
```
进入到文件底部，修改如下：
``` bash
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deployer:
  type: git
  repo: https://github.com/xiaoming123/xiaoming123.github.io.git
  branch: master
```
保存并退出Esc + :wq

---

### 添加git全局用户信息
``` bash
$ git config --global user.email "youremail"
$ git config --global user.name "yourname"
```

---

### 部署到github仓库中
``` bash
$ hexo d
```
此时访问<htttps://yourname.github.io/>，即可看到自己的本地博客已经上传到github服务器上。

至此github博客的搭建已经全部完成。

