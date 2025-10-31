---
title: 解决pip下载python包很慢的问题
date: 2022-05-28 12:39:10
tags: pip
categories: python
top_img: /images/pip.webp
cover: /images/pip.webp
---
# 前言
pip是python的包管理工具，它可以让我们安装python的包，并且可以自动更新，但是pip是从pypi中下载文件的，pypi是python官方的第三方库的仓库， 用的是国外的服务器，下载速度自然很慢。


## 解决方案
我们只需要在pip下载时将下载源切换为国内源即可（module替换为你要下载的包名）
### 安装
```bash
pip install -i https://pypi.douban.com/simple module # 使用豆瓣源
 
pip install -i http://mirrors.aliyun.com/pypi/simple/ module  # 阿里云 
 
pip install -i  https://pypi.mirrors.ustc.edu.cn/simple/ module  # 中国科技大学 
 
pip install -i  http://pypi.douban.com/simple/ module  # 豆瓣(douban) 
 
pip install -i  https://pypi.tuna.tsinghua.edu.cn/simple/ module  # 清华大学 
 
pip install -i  http://pypi.mirrors.ustc.edu.cn/simple/ module  # 中国科学技术大学
```

如果你对软件测试有兴趣，可以添加微信一起交流。
<img src="/images/wechat.jpg" width="50%" height="50%" />
