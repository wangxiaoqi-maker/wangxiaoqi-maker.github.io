---
title: fiddler实现ios手机抓包
date: 2023-04-26 16:28:36
tags: fiddler
---
# 前言
{% note default flat %}
最近由于工作中涉及到app的测试，在测试中出现的问题不方便去定位，所以出一期配置ios手机抓包的文章记录下，由于fiddler没有手机端，所以我们需要对PC端的fiddler和手机端进行简单的配置即可实现手机端的抓包，具体步骤如下:
{% endnote %}
## 对PC端的fiddler进行配置

- fiddler默认只会抓取http的请求，所以我们要先允许捕获https请求
- 操作步骤：
  - 打开Fiddler，点击Tool-> Options-> 切换到HTTPS 栏。
  - 勾选上并Capture HTTPS CONNECTs（捕获 HTTPS 连接）和 Decrypt HTTPS traffic （HTTPS 请求解密），并安装证书（首次使用无证书，会弹出是否信任fiddler证书和安全提示，直接点击yes就行）之后点击ok保存即可

![](https://files.mdnice.com/user/43232/428f069b-cf45-4110-ab06-2717a604e3aa.png =50%x)

##  1.允许远程设备连接

1.  如果想要捕获手机上的通信数据，就需要手机连接上Fiddler代理，而Fiddler默认是不允许其他设备进行连接的，所以我们要配置先允许远程设备连接。
2.  操作步骤：
  - 打开 Fiddler，点击Tool-> Options-> 切换到Connections 栏；
  - 在 Connections 面板选中 Allow remote computers to connect 允许其他设备连接，点击ok保存即可
    ![](https://files.mdnice.com/user/43232/93f837a6-fd4b-4c46-aaf8-6e34e8634010.png =50%x)

## 2.查看PC端的ip地址和fiddler开放的端口号

- 在fiddler客户端的右上角，将鼠标悬浮在Online按钮上即可看到本机的ip地址，可以看到我电脑的ip地址为：192.168.1.6
  ![](https://files.mdnice.com/user/43232/36fd9bbc-1e5d-48f2-92df-85a32cc85361.png =50%x)
- 根据下图可看到fiddler开放的默认端口为8088（可自行更改）
  ![](https://files.mdnice.com/user/43232/adf82868-436b-47ef-93cc-9cb1c7dbaf6b.png =50%x)


## 3.手机端安装fiddler证书

- 打开手机浏览器，输入https://192.168.1.6:8888(ip是我们本机的ip，端口号fiddler开放的端口号)，跳转到fiddler证书下载页
- 点击【FiddlerRoot certificate】，弹出“此网址尝试下载一个配置描述文件，您要允许吗？”，点击【允许】。

![](https://files.mdnice.com/user/43232/1ea317b2-a5b6-40f4-91e0-519bde8a0639.jpg =50%x)

- 下载完成后，打开设置->通用->vpn与设备管理->配置描述文件，点击下载的配置文件，点击安装按钮

![](https://files.mdnice.com/user/43232/8468095b-cbf1-4862-93b7-7e9644ec0152.jpg =50%x)

- 证书安装完成后进入设置->通用->关于本机->证书信任设置->针对根证书启用完全信任，选择该证书即可

![](https://files.mdnice.com/user/43232/b7410ff7-0d49-4d76-bd82-9311aaee8007.jpg =50%x)

## 4.手机配置代理

- 进入【设置】，点击链接wifi右方的感叹号，点击配置代理，设置为手动，然后填写服务器：输入fiddler的电脑ip地址192.168.1.6和端口8888，点击存储，保存即可。

![](https://files.mdnice.com/user/43232/bb898238-f862-4145-a53e-f15d4812a110.jpg =50%x)

{% tip info faa-horizontal animated-hover %}*配置好PC端的Fiddler和手机安装证书以及设置代理后，就可以在手机上操作app，然后在Fiddler查看发送的请求和响应了，大家赶紧去试试吧。* {% endtip %}