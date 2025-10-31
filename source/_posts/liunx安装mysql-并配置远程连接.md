---
title: liunx安装mysql并配置远程连接
date: 2022-06-19 21:22:22
tags: liunx
categories: mysql
top_img: /images/mysql.jpg
cover: /images/mysql.jpg
---


# 前言
{% note default flat %}
MySQL 是一款安全、跨平台、高效的，并与 PHP、Java 等主流编程语言紧密结合的数据库系统。该数据库系统是由瑞典的 MySQL AB 公司开发、发布并支持，由 MySQL 的初始开发人员 David Axmark 和 Michael Monty Widenius 于 1995 年建立的。
{% endnote %}
***
## 环境搭建
&ensp;<font size=3>**首先需要确认下是否已经安装了mysql**</font>
 ```bash
rpm -qa | grep mysql
```
&ensp;<font size=3>**这时候没有任何输出,则代表没有安装MySQL**</font>
&ensp;<font size=3>**如果查到了相关文件，则需要删除mysql，然后重新安装，如下图所示：**</font>
 <img src="/images/mysql_jpg1.jpg" />
### 删除mysql
&ensp;<font size=3>**使用 rpm -e 文件名 的命令删除该文件，以上面文件为例，执行以下命令：**</font>
 ```bash
 rpm -e mysql-community-release-el7-5.noarch
 ```
&ensp;<font size=3>**重复使用 rpm -e 命令删除文件，直到所有文件完全删除**</font>
&ensp;<font size=3>**如果提示依赖包错误，则使用以下命令尝试：**</font>
 ```bash
 rpm -ev MySQL-client-5.5.25a-1.rhel5 --nodeps
 ```
&ensp;<font size=3>**使用yum卸载安装的mysql**</font>
 ```bash
 yum remove mysql mysql-server mysql-libs mysql-server
 ```
###删除mysql目录
&ensp;<font size=3>**全局搜索名称包含 mysql 的所有文件**</font>
 ```bash
 find / -name "*mysql*"
 ```
&ensp;<font size=3>**使用 rm -rf 命令删除所有搜索出来的mysql文件。**</font>
 ```bash
 rm -rf /usr/lib/mysql
 ```
&ensp;<font size=3>**最后再检查一下：**</font>
 ```bash
 rpm -qa|grep mysql
 find / -name "*mysql*"
 ```
&ensp;<font size=3>**如果有的话继续执行删除，直到删干净为止！**</font>
### 安装mysql
&ensp;<font size=3>**Linux平台上推荐使用RPM包来安装mysql。**</font>
 ```bash
 wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
 ```
&ensp;<font size=3>**rpm安装：**</font>
 ```bash
 rpm -ivh mysql-community-release-el7-5.noarch.rpm
 ```
&ensp;<font size=3>**yum安装mysql-server：**</font>
 ```bash
 yum install mysql-server
 ```
&ensp;<font size=3>**等待安装完成后,重启MySQL服务: **</font>
 ```bash
 service mysqld restart
 ```
&ensp;<font size=3>**查看MySQL服务状态：**</font>
 ```bash
 service mysqld status
 ```
### 远程访问
&ensp;<font size=3>**远程访问MySQL服务**</font>
&ensp;<font size=3>**首先，远程访问需要设置一个允许远程访问的用户**</font>
&ensp;<font size=3>**新安装的MySQL是没有密码的,输入命令进入数据库：**</font>
 ```bash
  mysql -uroot
 ```

 ```bash
 grant all privileges on *.* to 'admin'@'%';
 ```
&ensp;<font size=3>**授予{% span red, admin %}用户所有访问权限,*{% span red, . %}* 代表任意数据库的任意表,{% span red,% %}代表任意ip地址(这里也可以直接授予root用户这种权限)**</font>
&ensp;<font size=3>**授权完成后刷新一下 : **</font>
 ```bash
 flush privileges;
 ```
&ensp;<font size=3>**修改密码 : **</font>
```bash
use mysql;
set password for 'admin'@'%'=password('123456');
```
&ensp;<font size=3>**常用命令： **</font>
```bash
service mysqld start # 启动mysql服务
service mysqld stop # 停止mysql服务
service mysqld restart # 重启mysql服务
service mysqld status # 查看mysql服务状态
```

{% tip info faa-horizontal animated-hover %}*此时，mysql就配置完成了，可尝试使用navicat测试连接* {% endtip %}

# 补充

{% tip info faa-horizontal animated-hover %}*如果是阿里云服务器需配置安全组，将3306端口开放即可，切记！* {% endtip %}
{% tip info faa-horizontal animated-hover %}*如果上述方法都尝试了，还是连接不上，可查看防火墙端口是否开放。具体方法自行百度，不在赘述* {% endtip %}

---
