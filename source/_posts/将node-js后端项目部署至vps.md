---
title: 将node.js后端项目部署至vps
date: 2019-07-12 16:40:52
tags:
- vps
- node
- express
- mongodb
- 部署
- nginx
- pm2
- pod
---
## 概要
需要将后端项目部署至自己的vps中
项目技术栈： express + mongodb
端口
- **express** 3000
- **mongodb** 默认端口 27017

## 环境部署
**以下都是我自己的步骤，每个人的步骤略有不同*

需要在vps上安装的环境
- node.js
- mongodb

### node.js
更新apt-get
`sudo apt-get update`

安装node
`sudo apt-get install nodejs`

安装npm
`sudo apt-get install npm`

全局安装n（npm版本管理器）
`sudo npm install n -g`

安装最新稳定版node
`n stable`

查看版本
`node -v`
`npm -v`

返回版本号表示安装成功

### mongodb

#### 安装
[官方指南](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-debian/)
我的 vps 的系统为debian 8, 注意在指南中对应自己的系统！

为mongodb管理包导入公钥
`sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4`

为mongodb建立版本文件夹 **对应 debian 8 系统*
`echo "deb http://repo.mongodb.org/apt/debian jessie/mongodb-org/4.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list`

更新本地apt-get
`sudo apt-get update`

安装最新版mongodb
`sudo apt-get install -y mongodb-org`

默认安装的文件路径：
- 数据文件 `/var/lib/mongodb`
- log文件 `/var/log/mongodb`

#### 运行
安装完成后，需要运行一下来确定是否已经安装完成

启动mongodb
`sudo service mongod start`

查看mongodb的log文件，确认是否开启
`cat /var/log/mongodb/mongod.log`
如果里面最后出现：
`[initandlisten] waiting for connections on port 27017`
表示mongodb已经开启

停止mongodb
`sudo service mongod stop`

重启mongodb
`sudo service mongod restart`

## 上传项目
本来上传我在想是要用古早的ftp上传吗？后来我发现了pod，尤大早些年写了自用的一个repo工具

### POD
[POD](https://github.com/yyx990803/pod)
这是尤大大写得一个用来上传的一个工具，本质上还是用了git hook，然后再用了 pm2 进行项目管理

#### 准备
vps 所需环境
- git
- pm2
**因为这些都很简单，此处就略过了*

#### 安装
安装POD
`npm install -g pod`

#### 初始化
初始化项目目录
`pod`
**如果是初次初始化，它会询问你需要初始化的项目路径，如果没有特殊要求，直接会车，就会初始化当前目录*
初始化后，会在当前目录生成 apps 和 repo 两个文件夹：
- apps 里会有 myapp 文件，用来放项目；
- repo 里有 myapp.git, 用来repo

#### clone & deploy 项目文件
本步骤是从 vps 回到本地进行

克隆 myapp 文件夹到本地
`git clone ssh://用户名@公网IP/黄色字体路径(repo)...myapp.git`

将所有项目文件复制进myapp
为该文件夹添加 deploy
`it remote add deploy ssh://用户名@公网IP/黄色字体路径(repo)...myapp.git`

创建 `.gitignore` 文件
```
node_modules
.vscode/
.DS_Store
dump.rdb
```
通过git上传文件
```
git add .
git commit -m "first commit"
git push deploy master
```

之后回到 vps 上查看项目文件里的文件是否上传成功

#### 启动项目
回到 vps 的项目文件夹
启动项目
`pod start myapp`
**官方说这里面的启动文件一定要是app.js, 其实应该是package.json 中的 main 的文件名要和主文件名一致，反正都用app.js或者index.js就不会出错*

查看项目
`pod list`
![](https://camo.githubusercontent.com/5775ccc7a9f2d9fa2227362fdf35227412a6fda0/687474703a2f2f692e696d6775722e636f6d2f70646132314b592e706e67)
你应该看到一个启动项目的详细列表

## 配置nginx
。。。待续。。。

前端小白一枚，如有更好的方法或者不对的地方请指正，谢谢
