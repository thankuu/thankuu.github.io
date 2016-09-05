---
title: "搭建Django应用（一）"
tags:
    - python
    - Django

---
## python安装
作为一个Python Web框架，Django需要Python 。 它适用Python 2.7、3.2、3.3和3.4。osx自带了python2.7版本，如果需要安装更高版本的python，可以通过`brew`安装：

        $ brew install python3
        $ python3 -V
        Python 3.5.1   显示版本号表明安装成功


或者通过安装`pyenv`来在多版本中来回切换

        $ brew install pyenv
更多使用方法可以参考：http://seisman.info/python-pyenv.html

## Django安装
Django要通过pip安装，pip是python中的一个包管理工具，可以通过它安装python的软件包，mac系统自带的python已经安装好了pip，可以直接通过他进行安装：

    $pip install Django
如果安装了多版本，需要使用pip可以参考官方文档：https://docs.python.org/3/installing/
使用时带上python版本号执行：

    $pip3 install Django

如果之前安装了Django就需要卸载之前版本，再重新安装。

## 验证
安装完成以后可以通过执行python，运行以下命令来验证安装：

    >>> import django
    >>> print(django.get_version())
    1.8

看到返回版本号证明你的Django已经安装成功了。


