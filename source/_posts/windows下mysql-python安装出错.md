---
title: windows下mysql-python安装出错
date: 2016-10-17 09:36:33
tags: 
- python
- mysql-python 
- mysql
---

windows安装python的mysql驱动是so easy的：

    pip install mysql-python

但是毫不意外的出错了：

     _mysql.c(42) : fatal error C1083: Cannot open include file: 'config-win.h': No such file or directory
     酱紫的：
     
![image](http://of66as8gb.bkt.clouddn.com/blog/20161017/104042119.png)
     
     
一般安装运行上面的神器命令都会出错，即使在linux下也是， 直接上解决办法：

    到 http://www.lfd.uci.edu/~gohlke/pythonlibs/ 下载二进制安装包
    网站打开大概是酱紫的：
    
![image](http://of66as8gb.bkt.clouddn.com/blog/20161017/103442345.png)

通过ctrl+f 搜索 "mysql-python", 然后就酱紫了：
    
![image](http://of66as8gb.bkt.clouddn.com/blog/20161017/103934947.png)

下载64位版本

    然后运行安装：pip install MySQL_python-1.2.5-cp27-none-win_amd64.whl

然后稍等一下
    
![image](http://of66as8gb.bkt.clouddn.com/blog/20161017/104709908.png)
    
oh yeah， 成功啦， 我门就可以快速的开始我门下一步的开发工作了
 

最后欢迎大家观看我关于 django + xadmin的教程：

	http://coding.imooc.com/class/evaluation/78.html