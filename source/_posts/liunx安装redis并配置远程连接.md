---
title: liunx安装redis
date: 2022-06-25 21:22:22
tags: liunx
categories: redis
top_img: /images/redis.jpg
cover: /images/redis1.gif
---

### 🎉 前言
{% note default flat %}
Redis 是完全开源免费的，遵守 BSD 协议，是一个灵活的高性能 key-value 数据结构存储，可以用来作为数据库、缓存和消息队列。
{% endnote %}
## 环境🍦
- redis版本：6.2.0
- liunx版本：centos 7.4
## 环境搭建 🚀
1. &ensp;<font size=3>**进入/usr/local 目录下,下载`redis`**</font>
```bash
wget http://download.redis.io/releases/redis-6.2.0.tar.gz
```
2. &ensp;<font size=3>**解压**</font>
```bash
tar -zxvf redis-6.2.0.tar.gz
```
3. &ensp;<font size=3>**进入解压目录**</font>
```bash
cd redis-6.2.0
```
4. &ensp;<font size=3>**编译安装**</font>
```bash
make
```
6. &ensp;<font size=3>**编译之后开始安装 🚀**</font>
```bash
make install
```
7. &ensp;<font size=3>**修改redis配置文件**</font>
```bash
vim /usr/local/redis-6.2.0/redis.conf
将daemonize no 改成daemonize yes		// 配置redis为后台启动
bind 127.0.0.1		// redis 默认只允许本机访问,注掉
bind 0.0.0.0		// 表示允许远程连接
requirepass 123456   // 设置redis密码

```

8. &ensp;<font size=3>**启动`redis`并指定配置文件**</font>
```bash
./src/redis-server /usr/local/redis-6.2.0/redis.conf
```
9. &ensp;<font size=3>**客户端登录**</font>
```bash
cd /usr/local/redis-6.2.0/bin
./redis-cli -h localhost -p 6379
```
10. &ensp;<font size=3>**开放`端口`**</font>
```bash
firewall-cmd --zone=public --add-port=6379/tcp --permanent
firewall-cmd --reload
```
11. &ensp;<font size=3>**关闭redis**</font>
```bash
redis-cli shutdown
redis-cli -a 密码 shutdown		// 设置了密码的话用这条命令关闭redis
```

# 补充

{% tip info faa-horizontal animated-hover %}*如果是阿里云服务器需配置安全组，将6379端口开放即可，切记！* {% endtip %}


