---
title: 把github贡献日历部署到博客上
date: 2022-5-29 18:53:00
tags:
  - 分享
categories: Hexo
comments: true
cover: https://pic.imgdb.cn/item/629321e10947543129228d44.png
abbrlink: 6
# sticky: 2 #置顶
---

# 前言

<font size=3>{% note default flat %}此教程转载自Reverse大佬(https://blog.c12th.cn/archives/6.html){% endnote %}</font>

---

<img src="https://ghchart.rshah.org/409ba5/12thstan" alt="12thstan's Github Chart" />

**<p align = "center">效果展示</p>**

---

# 教程
## 直接部署

- 这里用到了 [2016rshah 大佬](https://github.com/2016rshah/githubchart-api) 的方法。

```md
  <img src="https://ghchart.rshah.org/github用户名" alt="github用户名's Github chart" />
```
- [这个方法最绝的地方是，将你自己的贡献日历变成了图片，只需要使用一行 HTML 语句，就可以在任何地方使用贡献日历。](https://blog.csdn.net/weixin_44580076/article/details/118737615)

<br/>

- 如我在当前页面下部署

<img src="https://ghchart.rshah.org/12thstan" alt="12thstan's Github chart" />

```md
  <img src="https://ghchart.rshah.org/12thstan" alt="12thstan's Github chart" />
```

---

## 自定义 颜色
- 自定义 **颜色** **#000000** *十六进制颜色代码* **<font size="3" color="red">注意格式不带#</font>**

```md
  <img src="https://ghchart.rshah.org/000000/github用户名" alt="github用户名's Github Chart" />
```

<br/>

- <font size="3" color="blue">蓝色</font> **#0000FF**

<img src="https://ghchart.rshah.org/0000FF/12thstan" alt="12thstan's Github Chart" />

```md
  <img src="https://ghchart.rshah.org/0000FF/12thstan" alt="12thstan's Github Chart" />
```

- <font size="3" color="red">红色</font> **#ff0000**

<img src="https://ghchart.rshah.org/ff0000/12thstan" alt="12thstan's Github Chart" />

```md
  <img src="https://ghchart.rshah.org/ff0000/12thstan" alt="12thstan's Github Chart" />
```

- <font size="3" color="green">绿色</font> **#00785D**

<img src="https://ghchart.rshah.org/00785D/12thstan" alt="12thstan's Github Chart" />

```md
  <img src="https://ghchart.rshah.org/00785D/12thstan" alt="12thstan's Github Chart" />
```

---
## 部署到 Next 主题
- **这部分是关于 Next 主题的设置，请** <font size="3" color="blue">点击链接</font> **移步到 Next 主题**

{% link 把github贡献日历部署到博客上(二), https://next.c12th.cn/archives/3.html, https://pic.imgdb.cn/item/624aa2d5239250f7c5147553.ico %}

---

# 补充

{% tip info faa-horizontal animated-hover %}如不能使用 **HTML语句** 可看另一种写法 *[把github贡献日历部署到博客上(二)](https://jsimple.c12th.cn/posts/3/)*{% endtip %}

{% tip cogs faa-horizontal animated-hover %}第一篇 *[把github贡献日历部署到博客上(一)](https://blog.c12th.cn/archives/5.html)* {% endtip %}

---