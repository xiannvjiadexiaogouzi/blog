---
title: 【跳坑指南】在vps上部署hexo博客的那些事
date: 2019-05-07 14:54:31
tags: 
- blog
- vps
- hexo
- git
- nginx
---
**说在前面**
本地使用macOS，vps系统为debian 8；
虽说是跳坑指南，但是这篇文章本意也仅仅自用，总结一下自己遇到的问题和解决方法，仅供参考，如有其他问题请百度(google)

**整体思路：**
本地调试完的hexo项目文件通过git推送给vps，然后直接部署在vps上使用域名来访问，这样一个属于自己的blog就可以完成了

<!-- more -->

![](hexo-vps.png)

## 本地环境
### Git
Git是一个免费的开源分布式版本控制v系统，旨在快速，高效地处理从小型到大型项目的所有事务。这原是广泛用在代码的版本控制，在hexo建站里面的主要作用是推送渲染好的静态网页文件到部署的仓库。
#### 下载Git与安装
因为本来我就有，所以这里就不多介绍；
网上其他大佬也有很多教程
### node.js
#### 下载Node.js与安装
同上，本地环境也有；
### Hexo
Hexo是一个快速、简洁且高效的静态博客框架。安装简单，虽然网上教程一大堆，建议参考官方文档。重要！官方文档有中文 😃。

**官方文档**
https://hexo.io/zh-cn/

#### 安装hexo
首先确保本地安装好git和node.js，在终端中输入
`npm install -g hexo-cli`

#### 创建hexo项目（建站）
```
hexo init <folder> // 初始化项目文件夹；<folder>为自定义hexo项目名称
cd <folder> // 打开项目文件夹
npm install // 安装依赖
```
完成上述步骤，查看自己的项目文件夹内，目录应为：
```
.
├── node_modules //依赖包
├── scaffolds // 模版
├── source // 文章资源
|   ├── _drafts //草稿
|   └── _posts //发布文章
├── themes //主题
|   └── landscape //默认主题
├── package.json // npm配置文件（可以不用管它）
└── _config.yml //hexo配置文件
```
**配置hexo**
既然生成了博客了，那自然需要自己来配置一下自己的博客的信息，打开_config.yml文件进行配置；
具体配置参考[官方文档](https://hexo.io/zh-cn/docs/configuration)

**下载主题**
主题我挑了很久，最后选择综合症的我只有选了好多人推荐的next主题
[next官方文档](https://github.com/theme-next/hexo-theme-next/blob/master/docs/zh-CN/README.md)

**下载**
根据文档给的方式使用`git clone`
```shell
git clone https://github.com/theme-next/hexo-theme-next themes/next
```

试了几次，我一直报错 
`'RPC failed; curl 18 transfer closed with outstanding read data remaining'`

解决方案：
`git config http.postBuffer 524288000`
当然啦，依旧报错，并没有解决我的问题
参考：https://www.cnblogs.com/zjfjava/p/10392150.html

所以我将git clone里的https地址换成了ssh地址，完美解决，虽然网速依然很慢，但是不报错啊！
`git clone git@github.com/theme-next/hexo-theme-next themes/next`

**主题配置**
直接在项目文档里`clone`或者单独`clone`后将`next`文件夹复制进项目的`themes`文件夹里
```shell
.
├── themes //主题
    ├── landscape //默认主题
    └── next //新clone的next主题
```
并在hexo配置文件中更改主题
```json
// ...
theme: next //更改主题
// ...
```
其他配置暂时就不说了，以后会再次具体写

## vps环境
### vps
如何购买vps不在本文详细叙述，不论在阿里云还是腾讯云还是海外譬如搬瓦工之类购买，都是差不多的，vps大同小异，不过好像是国内的服务器配置域名是需要实名认证的，但是海外的就不需要；

重新说明我的vps安装的系统是debian 8，如果有和我一样的小伙伴那么我的接下里的配置就很有参考价值了，当然其他版本的系统原理和流程其实也是大同小异；

再多说一句，系统不同，安装包的命令也是不同的：
`rpm`/`yum`适用于Redhat、CentOS、Suse等平台；
`apt-get`/`dpkg`适用于Debian、Ubuntu等平台；
请大家自己寻找对应的安装包，我这边用的是`apt-get`;

### 部署原理
本地hexo文本编辑后，使用git远程部署到vps的git仓库。`hexo d`只deploy本地打包后的`public`文件夹；
在vps的git仓库里使用git hooks，当git仓库收到最新的push时，将git仓库接受到的内容复制到VPS上的网站根目录内；相当于完成了手动将`public`文件夹复制到vps的网站根目录内

**连接vps**
我觉得如果看到这篇文章了，用ssh连接vps应该是都会的吧，那我就不说了；不会的请百度(谷歌) 😂

😤 算了，我稍微提一下，打开终端输入
`ssh 用户名(一般是root)@vps的ip地址 -p 端口号`
按照提示输入密码，出现`[root@主机名~]#`，登陆成功 😀

### Git
#### 安装
```shell
# 安装git
apt install git

# 查看版本号，用以确认安装成功
git -version
```
 
#### 创建git用户
为什么要创建git用户？
老实说，如果是自己一个人用的vps，那么不用git用户其实也没啥，但是创建的意义就是用git用户来管理git和代码，而不用root，这样更安全；

```shell
# 创建git用户
adduser git
# 按照提示设置密码

# 赋予git用户sudo权限
chmod 740 /etc/sudoers #修改权限
vim /etc/sudoers #编辑该文件

# 😃 成功后，修改如下内容
# User privilege specification
root    ALL=(ALL:ALL) ALL
git     ALL=(ALL:ALL) ALL #添加此行内容

# 修改回文件的权限
chmod 440 /etc/sudoers
```

**切换git用户，配置ssh**
下载和上传代码的都是使用git这个用户，但我们又不想要每次都输入密码，怎么办？
所以我们使用ssh证书，本地生成公钥复制至服务端，每次登陆只要服务器自动核对公钥是否匹配即可

切换git用户
`su git`

创建.ssh文件夹
`mkdir ~/.ssh`

创建authorized_keys文件并编辑，将公钥复制进去即可
`vim ~/.ssh/authorized_keys`
保存退出

- 本地
先在本地电脑中获取公钥，再将公钥复制粘贴到authorized_keys
（本地）查看公钥
`cat ~/.ssh/id_rsa.pub | pbcopy`

修改公钥文件相应权限
`chmod 600 ~/.ssh/authorzied_keys` #只有拥有者有读写权限
`chmod 700 ~/.ssh` #只有拥有者有读、写、执行权限

返回本地客户端，测试是否可以连接上服务器(此时应该免密登陆成功) 😃
`ssh -v git@$ip -p $port`
如果不成功请重试以上配置ssh步骤

#### 配置git hooks
先确定自己想将项目的根目录放在哪里？
我的是这样的：
`/home/git/blog/blog.git #git仓库`
`/home/git/blog/hexo #项目根目录`

**初始化git仓库**
`git init --bare blog.git`
参数 --bare ，创建一个裸仓库，不包含工作区

**配置git hooks**
我们这里要使用的是 post-receive 的 hook，这个 hook 会在整个 git 操作过程完结以后被运行。[hook详情](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)

在blog.git/hooks目录下新建一个post-receive文件:
`vim post-receive`

文件中写入：
```
#!/bin/sh
git --work-tree=/home/git/blog/hexo --git-dir=/home/git/blog/blog.git checkout -f
```
`--work-tree` 对应项目部署根目录
`--git-dir` 对应版本库目录
保存退出

添加post-receive文件的可执行权限
`chmod +x post-receive`

blog.git的拥有者为git用户
`chown -R git:git blog.git`

禁用git用户的shell登录权限
`vim /etc/passwd`
找到
`git:x:1001:1001:,,,:/home/git:/bin/bash`
改为
`git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell`
保存退出
这样git用户无法使用shell登陆，但可以通过ssh正常使用git推送

参考：https://www.cnblogs.com/navysummer/p/9842065.html

多说一句
执行`ll`命令，确保以上`blog.git`、`.ssh`、`blog`目录的用户组权限为`git:git`
`ll /home/git`

如果不是，执行
`chown git:git -R /home/git/blog`

### nginx
#### 安装 nginx
安装 Ngnix
`apt-get install nginx`

查看版本号（确认安装成功）
`nginx -v`

#### 配置 nginx
关于nginx的配置，这里要说明一下
- `/etc/nginx/nginx.config`是nginx的默认配置文件；
- 如果需要使用nginx的虚拟机服务配置多个config文件，应该在`/etc/nginx/site-avalible`里修改，这是官方推荐的方法，然而此时还必须往`/etc/nginx/sites-enabled`里添加 `site-avalible`的软连接；
- 可以直接在`/etc/nginx/config.d`（用户自定义配置文件夹）里添加不同server的config文件，最后在`nginx.config`里 include 就可以了；

##### site-avalible
```
cd /etc/nginx/sites-available //切换目录
cp default default.bak //备份默认配置
vim default //修改配置
```
找到server,修改里面所有的location和root
```
server {
    # 默认80端口
    listen       80 default_server;
    listen       [::]:80 default_server;
    # 修改server_name为自己之前注册好的域名
    server_name  www.xksblog.top xksblog.top;
    # 修改网站根目录，在这里存放你的Hexo静态文件，请自行选择或创建目录
    root         /home/git/blog/hexo;

    location / {
        root    /home/git/blog/hexo; #部署的blog根目录
        index   index.html index.htm
    }

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}
```
这里我修改了default文件，如果需要添加其他文件，需要在site-enabled里增加相应软连接
`ln -s /etc/nginx/sites-available/dotcom /etc/nginx/sites-enabled/dotcom`


##### nginx.conf
配置文件路径
`/etc/nginx/nginx.conf`
我看了看，不同vps版本系统的文件路径不太一样，请大家自己寻找对应系统的教程或者自己在系统里寻找文件路径

打开nginx.conf文件所在目录
`cd /etc/nginx`
```
# 修改前先备份一下
cp nginx.conf nginx.conf.bak
# 修改配置文件
vim nginx.conf
```

修改配置文件中的http部分：
```
http {
    ...
    include /etc/nginx/conf.d/*.conf # 引入自定义sconfig文件
    include /etc/nginx/site-enabled/*.conf # site-enabled(site-avalible)的server的配置
    ...
}
```
保存退出，使用`nginx -t`查看配置是否有错误

启动ngnix
`service nginx start`
浏览器打开服务器public IP，检查ngnix是否启动成功

停止ngnix
`nginx -s stop`

重启nginx
`nginx -s reload`

查看运行状态
`systemctl status nginx`
显示running表示成功运行

参考：
[VPS搭建个人Hexo博客](https://segmentfault.com/a/1190000016106584)
[Hexo搭建个人博客并使用Git部署到VPS](https://www.jianshu.com/p/b926ecf1c6f6)
[Nginx 使用及配置](https://www.jianshu.com/p/849343f679aa)

## 本地deploy
现在距离成功还有一步之遥

### 安装deploy插件
`npm install hexo-deployer-git --save`

### hexo 的 deploy 配置
打开hexo配置文件_config.yml, 找到deploy
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
    type: git
    repo: git@VPS的IP:blog.git目录
    branch: master
```
一般到这边就可以直接`hexo d`了, 但是我报错了，发现默认端口22是无法连接上的，因为vps的默认端口不是22；
此时有两种解决方法
- 一种是修改vps的默认端口至22；
- 第二种就是修改一下deploy的配置，从指定端口连接

我选择第二种，方法如下：
`repo: ssh://git@ip:port/blog.git目录`

生成并推送部署
`hexo d -g`

此时用域名或者vps ip连接，自己的blog就完成了！😃

## PS

本人前端新手一枚，如有错误的话欢迎指正
喜欢的话麻烦 Github 给个 🌟 哦
