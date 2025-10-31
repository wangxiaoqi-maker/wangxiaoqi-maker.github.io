---
title: python导出指定项目依赖包
date: 2022-06-03 18:14:25
tags: 导出依赖包
categories: python
top_img: /images/package.png
cover: /images/package.png
---
# 前言
{% note default simple %}
很多朋友都知道，利用pip也好，利用conda也好，我们可以使用pip freeze、conda export等语句来对当前的Python环境依赖进行导出备份，以方便在其他机器上还原环境，但此类环境依赖导出方法的局限在于，它会将当前环境下所有已安装的库信息进行导出，使得导出的结果繁杂臃肿。 而如果我们只想针对某个Python项目工程，将其内部真正导入的库信息进行导出，就可以用到pipreqs这个工具，使用pip install pipreqs进行安装之后，就可以以命令行的形式使用它
{% endnote %}
***
## pipreqs的使用
### 安装pipreqs
```bash
pip install pipreqs
```
&ensp;<font size=3>**安装完毕之后，就可以使用{% span blue, pipreqs %}命令来导出当前Python环境下的依赖包信息，进入项目根目录执行如下命令：**</font>
```bash
pipreqs ./
```
&ensp;<font size=3>**若成功，则会在项目根目录下生成一个{% span blue,requirements.txt %}文件。有时也会遇到一些{% span red, 错误 %}，具体如下：**</font>
```bash
Traceback (most recent call last):
  File "E:\python\3.8.0\lib\runpy.py", line 192, in _run_module_as_main
    return _run_code(code, main_globals, None,
  File "E:\python\3.8.0\lib\runpy.py", line 85, in _run_code
    exec(code, run_globals)
  File "E:\Python\3.8.0\Scripts\pipreqs.exe\__main__.py", line 7, in <module>
  File "E:\python\3.8.0\lib\site-packages\pipreqs\pipreqs.py", line 470, in main
    init(args)
  File "E\python\3.8.0\lib\site-packages\pipreqs\pipreqs.py", line 406, in init
    candidates = get_all_imports(input_path,
  File "E:\python\3.8.0\lib\site-packages\pipreqs\pipreqs.py", line 122, in get_all_imports
    contents = f.read()
UnicodeDecodeError: 'gbk' codec can't decode byte 0xa4 in position 135: illegal multibyte sequence
```
&ensp;<font size=3>**{% span green, 解决办法：%}**</font>
&ensp;<font size=3>**1. 这是一个编码解码时发生的错误，我们需要在生成依赖包信息时指定编码方式。具体方式如以下命令：**</font>
```bash
pipreqs ./ --encoding=utf8
```
## 导入依赖包
&ensp;<font size=3>**导入依赖包的方法：**</font>
&ensp;<font size=3>**1. 在项目根目录下执行如下命令：**</font>
```bash
pip install -r requirements.txt
```
&ensp;<font size=3>**也可指定使用其他源来加速安装依赖包，如豆瓣pypi，比如：**</font>
```bash
pip install -r requirements.txt -i https://pypi.douban.com/simple
```
