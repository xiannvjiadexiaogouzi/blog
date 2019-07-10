---
title: namesilo域名购买以及解析
date: 2019-07-09 23:26:11
tags:
- blog
- vps
- domain
- 域名
- ssl
- https
---
## 购买&解析域名

看了网上很多domain的推荐，最后在NameSilo上购买了域名人生当中的第一个域名（thisisluis.top），不得不说**namesilo**还是十分便宜的；
然后将域名解析到我的服务器ip（配置解析后有延迟，但我这边只过了几分钟就生效了），然后照着用CloudFlare免费上了https，记录一下过程

#### 域名(NameSilo)
为了便于记忆，人们采用域名的方式代替ip，由此产生了域名解析。
类型分为:A ,AAAA , CNAME , MX , TXT/SPF , SRV , CAA记录解析。

- **A**：
>A (Address) 记录是用来指定主机名（或域名）对应的IP地址记录。用户可以将该域名下的网站服务器指向到自己的web server上。同时也可以设置您域名的二级域名

- **CNAME**：
>别名记录。这种记录允许您将多个名字映射到另外一个域名。通常用于同时提供WWW和MAIL服务的计算机。
>例如，有一台计算机名为“host.mydomain.com”（A记录）。它同时提供WWW和MAIL服务，为了便于用户访问服务。可以为该计算机设置两个别名（CNAME）：WWW和MAIL。这两个别名的全称就是 www.mydomain.com 和 mail.mydomain.com。实际上他们都指向 host.mydomain.com。

[解析服务器](https://www.uud.me/site-notes/namesilo.html)

[配置域名](https://www.cnblogs.com/croso/p/5400890.html)


## SSL

现在主流浏览器都会鼓励网站添加HTTPS支持，如果还不与时俱进的话就会被它们在地址栏上显（羞）示（辱）“不安全”。 由于个人博客买收费证书还是比较奢侈的，还好有 Let’s Encrypt 为我们提供了免费的解决方案。

>**Let’s Encrypt** 是一家开源免费的证书机构，也是永久免费的SSL通配符证书。由Mozilla、Cisco、Akamai、IdenTrust、EFF等组织人员发起，具有较高的权威性，而且其颁发的证书可以自动续期，极大地方便了使用

我查了一下，按我的理解基本上分为两种方法
- 使用 [acme.sh](https://github.com/Neilpang/acme.sh) 或certbot这种第三方证书脚本工具来处理

- 使用cloudflare CDN 功能，即通过cloudflare的dns解析实现
**这次暂时选择了这种方式*

### acme.sh脚本

简单来说acme.sh实现了 acme 协议, 可以从 let‘s encrypt 生成免费的证书。

- 一个纯粹用Shell（Unix shell）语言编写的ACME协议客户端。
- 完整的ACME协议实施。 支持ACME v1和ACME v2，支持ACME v2通配符证书
- 简单，功能强大且易于使用，只需要3分钟就可以学习它。Let's Encrypt免费证书客户端最简单的shell脚本。
- 纯粹用Shell编写，不依赖于python或官方的Let's Encrypt客户端。
- 只需一个脚本即可自动颁发，续订和安装证书。不需要root/sudoer访问权限。
- 支持在Docker内使用，支持IPv6

[acme.sh项目](https://github.com/Neilpang/acme.sh) 

https://www.cnblogs.com/clsn/p/10040334.html

简单来说步骤如下
- 在服务器安装 acme.sh
- 生成证书
- copy 证书到 nginx/apache 或者其他服务
- 配置nginx/apache
- 更新证书
- 更新 acme.sh

https://www.liumapp.com/articles/2019/05/23/1558574698880.html#comments
这里面说到
>ACME提供两种方式，一种是不指定DNS运营厂商，我们自己去修改txt解析记录即可，另外一种是指定CloudFlare，也就是说您的域名DNS解析地址要由CloudFlare提供

>经过实际测试，前者获取到的证书在浏览器上会被标识为不安全，而后者却会被标识为安全

所以最后可能的办法是这样的
https://josta.me/blog/201708/domain_ssl/
[使用Cloudflare为博客增加Let's Encrypt免费SSL支持](https://josta.me/blog/201808/cloudflare_ssl/)



### cloudflare
Cloudflare以向客户提供网站安全管理、性能优化及相关的技术支持为主要业务。我们这里使用全站Cloudflare的免费SSL证书实现全站SSL加密（即https）访问

用户 <======> cloudflare ------> your server

**因为还可以使用 [cloudflare CDN](https://www.freehao123.com/cloudflare-cdn-ssl/) 功能，在线对网站的管理界面也非常方便，因为都是相当于在cloudflare端进行控制的，所以非常方便*


大致流程
- 注册cloudfare账号
- 绑定自己的域名
- 切换域名DNS解析服务地址 
**这步的时间略长，但我也就30min不到*
- Crypto 界面设置SSL加密方式
**如果服务器没有安装证书，就只能使用 flexible 加密方式*

[如何使用cloudflare加SSL](https://www.jianshu.com/p/24d3800f597a)



