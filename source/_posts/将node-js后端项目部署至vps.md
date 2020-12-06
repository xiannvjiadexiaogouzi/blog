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

<!-- more -->

### node.js
```bash
# 更新apt-get
sudo apt-get update

# 安装node
sudo apt-get install nodejs

# 安装npm
sudo apt-get install npm

# 全局安装n（npm版本管理器）
sudo npm install n -g

# 安装最新稳定版node
n stable

# 查看版本
node -v
npm -v
# 返回版本号表示安装成功
```

### mongodb

#### 安装
[官方指南](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-debian/)
我的 vps 的系统为debian 8, 注意在指南中对应自己的系统！

- 为mongodb管理包导入公钥
```bash
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4
```

- 为mongodb建立版本文件夹 **对应 debian 8 系统*
```bash
echo "deb http://repo.mongodb.org/apt/debian jessie/mongodb-org/4.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list
```

- 更新本地`apt-get`
```bash
sudo apt-get update
```

- 安装最新版mongodb
```bash
sudo apt-get install -y mongodb-org
```

默认安装的文件路径：
- 数据文件 `/var/lib/mongodb`
- log文件 `/var/log/mongodb`

#### 运行
安装完成后，需要运行一下来确定是否已经安装完成

- 启动mongodb
```bash
sudo service mongod start
```

- 查看mongodb的log文件，确认是否开启
```bash
cat /var/log/mongodb/mongod.log
# 如果里面最后出现：
[initandlisten] waiting for connections on port 27017
# 表示mongodb已经开启
```

- 停止mongodb
```bash
sudo service mongod stop
```

- 重启mongodb
```bash
sudo service mongod restart
```

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
- 安装POD
`npm install -g pod`

#### 初始化
- 初始化项目目录
`pod`
**如果是初次初始化，它会询问你需要初始化的项目路径，如果没有特殊要求，直接会车，就会初始化当前目录*
初始化后，会在当前目录生成 apps 和 repo 两个文件夹：
  - apps 里会有 myapp 文件，用来放项目；
  - repo 里有 myapp.git, 用来repo

#### clone & deploy 项目文件
本步骤是从 vps 回到本地进行

- 克隆 myapp 文件夹到本地
`git clone ssh://用户名@公网IP/黄色字体路径(repo)...myapp.git`

- 将所有项目文件复制进myapp
为该文件夹添加 deploy
`it remote add deploy ssh://用户名@公网IP/黄色字体路径(repo)...myapp.git`

- 创建 `.gitignore` 文件
```
node_modules
.vscode/
.DS_Store
dump.rdb
```
- 通过git上传文件
```
git add .
git commit -m "first commit"
git push deploy master
```

之后回到 vps 上查看项目文件里的文件是否上传成功

#### pm2 启动项目
回到 vps 的项目文件夹

***我这边时使用 git 用户来管理的，所以我是切换到了 git 用户来进行以下操作的，即如果切换回其他用户，`pm2 list`也是看不到有node服务在跑的；***

切换 git 用户
`su git`

这里使用 pm2 来启动文件，因为我这边发现使用了 pod 启动之后，无法正确开启项目，虽然可以看到项目运行，但似乎并没有实际运行，而且无法停止；最后我重启了服务器才解决

*所以这边多说一句，要用 pm2 来启动程序*
[*想用 pod 来管理的请看这里*](#pod启动)

- 启动项目
在项目文件内
`pm2 start $app.js`

- 查看项目
`pm2 list`
![](pm2-list.png)

[常用命令](https://www.jianshu.com/p/e709b71f12da)
[pm2简易使用手册](https://juejin.im/post/5be406705188256dbb5176f9)

这样 node 服务就跑在了服务器上的3000端口

#### pod启动
- 启动项目
`pod start myapp`
**官方说这里面的启动文件一定要是app.js, 其实应该是package.json 中的 main 的文件名要和主文件名一致，反正都用app.js或者index.js就不会出错*

- 查看项目
`pod list`
你应该看到一个启动项目的详细列表
![](https://camo.githubusercontent.com/5775ccc7a9f2d9fa2227362fdf35227412a6fda0/687474703a2f2f692e696d6775722e636f6d2f70646132314b592e706e67)


## 配置nginx
现在我希望
- 从我的域名 [www.domain.com]()(瞎写的) 访问，打开我的blog；
- 从server的ip打开是访问我的node服务;

其实就是在nginx新建一个server，然后后使得当访问ip的时候转发到后端的node服务

### 修改 blog server
在`/etc/nginx/sites-available`中找到对应的blog的配置文件，`vi $blog`
```
server {
        listen 80; # 注释掉后面的default_server, 因为现在需要多项目共用80端口
        #default_server;
        #listen [::]:80 default_server;

        root /home/.../hexo; # blog 文件根目录
        
        # Add index.php to the list if you are using PHP                
        index index.html index.htm index.nginx-debian.html;

        server_name domain.com www.domain.com;  # 客户端访问的域名

        location / {                                                    
                # First attempt to serve request as file, then          
                # as directory, then fall back to displaying a 404.     
                try_files $uri $uri/ =404;     
        }
        ...                
}
```
修改完，`esc`退出编辑，`:wq`保存修改

删除原来sites-enabled中的软连接
`rm ../sites-enabled/$blog`

重新创建软链接
`ln -s /etc/nginx/sites-available/$blog /etc/nginx/sites-enabled/$blog`

### 创建 node 的server
回到sites-availble文件夹中

拷贝一份新的server文件
`cp default myapp`

打开新文件`myapp`
`vi myapp`

修改如下
```
server {
        listen 80;

        server_name XX.XXX.XXX.XXX; # server_name 改成server ip 或者其他你有的域名

        location / {                                                    
                # First attempt to serve request as file, then          
                # as directory, then fall back to displaying a 404.     
                try_files $uri $uri/ =404;                              
                proxy_pass  http://127.0.0.1:3000/; # 填写你要转发的地址，node是跑在服务器上3000端口的
                # 这步一定要在最后加 / ，这样 nginx 会自动转发你的 uri 到node下面，才可以访问 node 后面的子路由
                proxy_set_header Host  $http_host;                      
                proxy_set_header X-Real-IP  $remote_addr;               
                proxy_set_header X-Forwarded-For  $proxy_add_x_forwarded_for;
                proxy_set_header X-Nginx-proxy true;                    
                proxy_redirect off;              
                proxy_set_header   Connection "";
                #proxy_cache one;                
                proxy_cache_key sfs$request_uri$scheme;          
                proxy_http_version 1.1;          
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
        }
        ...
}
```

创建软链接
`ln -s /etc/nginx/sites-available/$myapp /etc/nginx/sites-enabled/$myapp`

全部创建完毕后，重启 nginx 服务
`nginx -s relaod`

### 最后
现在，当你用域名访问的时候就是打开blog，而用服务器ip访问就是node服务了；

**其实就是用node服务开了反向代理，然后当访问ip时，从 ip:8080 端口转发到了服务器的 node 所在 3000 端口**

前端小白一枚，如有更好的方法或者不对的地方请指正，谢谢
