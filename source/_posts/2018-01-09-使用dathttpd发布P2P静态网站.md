---
title: 使用dathttpd发布P2P静态网站
tags: []
categories: []
date: 2018-01-09 10:21:00
permalink:
---



***这篇文章主要介绍利用dathttpd来发布个人的到静态博客***

最近P2P的概念非常流行，博主本身对P2P网络也非常感兴趣，随着网络的发展，中心化的网站已经成为网络的累赘，另外各种服务对科技公司的要求越来越高，然后最近国内爆出泄露隐私、监控的几条新闻，基本涉及了国内的全部大型互联网公司。

有关P2P的概念就不多说了，感兴趣的可以自行谷歌、百度，或者关注博主的博客也可以，不过我不是专业人士，只能简单的介绍一下啦,作者的环境是，本地环境macOS 服务器环境centos 7 .类unix系统都差不多，windows系统请自行百度下载相应的软件进行搭建，下面介绍发布过程

<!-- more -->

### 1 利用hexo发布静态博客
搭建静态博客有多种方法，如hugo 纸小墨 等，自行百度

hexo的搭建依赖nodejs

#### 1.1 安装hexo

1.到这里下载[nodejs download](https://nodejs.org/zh-cn/)  下载好软件包后直接点击安装，推荐使用最新版本

如果显示无法安装第三方软件包，请使用终端解锁，命令如下：

```bash
$ sudo spctl --master-disable
```
2.***安装hexo***

安装前确保系统内有git，没有到话：

```bash
$ brew install git
```
hexo安装

```bash
$ npm install -g hexo-cli
```
如果报错，请查看错误信心，出现包含root | administrator的关键字信息，请使用sudo进行安装

#### 1.2 建立博客
选择一个文件夹，比如我在桌面上新建一个文件夹，命名hexo_blog
终端命令如下：

```bash
$ cd ~/User/$user/Desktop
$ mkdir hexo_blog    #创建本地目录
$ hexo init    #初始化博客
$ hexo s    #hexo server的缩写，启动本地服务，此时可以在浏览器里打开localhost:4000 来查看网站的样子
$ hexo g    #生成静态网页，可以发现在本地生成了一个public文件夹，就说网站所在地址
$ hexo d    # deploy,发布的意思
```

#### 2 利用BeakerBrowser发布网站到P2P网络

步骤如下：

1.打开Beaker菜单 -> New Site -> 写好title -> create site 

2.打开Library -> 在左上角选择你的文件夹 -> 点开share右侧下啦菜单 -> change folder -> Publish

3.网站地址点击share就出来了

类似下面

```bash
dat://3fbc2591d8f34dc3245bc7c8e682b778d9c06be31d819d4a44e541c3434d2b79
```
复制粘贴到Beaker里就能看到你的网站了
现在网站正常了，但一旦你的客户端机器关机，该网站就打不开了，第二这个地址hash看着也有点膈应，所以我们可以利用dathttpd来让你的网站一直保持在线，并且把网址改成正常的域名地址来访问

#### 3 利用dathttpd发布
1.打开服务器，输入一下命令

```bash
$ sudo yum update
$ sudo yum install libtool m4 automake libcap2-bin build-essentials    #安装依赖包
$ npm install -g dathttpd    #安装dathttpd
```

2.编辑配置文件

```bash
$ vim ~/.dathttpd.yml
ports:
  http: 80    #我的设置在9999,因为服务器80端口被nginx占用了
  https: 443
  metric: 8089
directory: ~/.dathttpd
letsencrypt:     #加密，会自动跳转https
  email: 'aa@bb.com'
  agreeTos: true
sites:
  tfwall.tk:     #我写的是dat.tfwall.tk
    url: dat://3fbc2591d8f34dc3245bc7c8e682b778d9c06be31d819d4a44e541c3434d2b79
    datonly: false
```
***配置段不要用tab键，直接用空格，听说会出问题，未测试***

正常这样就可以访问了，请用DNS进行转发顶级域名到服务器IP即可正常访问

3.我的机房是多服务器单IP，因此采用nginx转发
配置如下：

```bash
$ vim /etc/nginx/conf.d/reserve-proxy.conf
server{
    listen 80;
    server_name tfwall.tk;
    location /{
	proxy_redirect off;
	proxy_set_header Host $host;
	proxy_set_header X-Real-IP $remote_addr;
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	proxy_pass http://127.0.0.1:8888;
	#proxy_pass http://tfwall.tk;
    }
    access_log /var/log/nginx/tfwall_access.log;
}

server{
    listen 80;
    server_name dat.tfwall.tk;
    location /{
	proxy_redirect off;
	proxy_set_header Host $host;
	proxy_set_header X-Real-IP $remote_addr;
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	proxy_pass http://127.0.0.1:9999;
	#proxy_pass http://tfwall.tk;
    }
    access_log /var/log/nginx/tfwall_access.log;
}
```
重点在第二个server字段

4.配置好后，启动dathttpd吧

```bash
$ dathttpd
```
打开正常浏览器访问tfwall.tk来看看
或者打开dat.tfwall.tk看你的配置
或者在beakerbrowser浏览器里打开

```bash
dat://3fbc2591d8f34dc3245bc7c8e682b778d9c06be31d819d4a44e541c3434d2b79
```