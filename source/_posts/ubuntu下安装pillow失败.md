---
title: ubuntu下安装pillow失败
date: 2016-10-21 15:06:00
tags:
---


ubuntu 16.04下
运行命令

    pip install pillow
    
然后失败了：

解决办法，先安装：

    sudo apt-get install libjpeg8 libjpeg62-dev libfreetype6 libfreetype6-dev

然后重新运行

    pip install pillow

	
最后欢迎大家观看我关于 django + xadmin的教程：

	http://coding.imooc.com/class/evaluation/78.html