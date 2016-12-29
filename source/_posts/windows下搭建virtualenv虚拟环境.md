---
title: windows下搭建virtualenv、virtualenvwrapper虚拟环境
date: 2016-10-17 16:23:42
tags:
- virtualenv
- virtualenvwrapper-win
- virtualenvwrapper
---

操作系统： win7

### virtualenv

##### 1. 安装virtualenv

 
    pip install virtualenv

![image](http://of66as8gb.bkt.clouddn.com/blog/20161017/154500976.png)

##### 2. 新建虚拟环境

    
    virtualenv bobbyvir
    
![image](http://of66as8gb.bkt.clouddn.com/blog/20161017/154818931.png)

    注： 1. 虚拟环境位于当前命令的目录下 这里是 E:\Projects\projectsedu.com
         2. 虚拟环境名称为 bobbyvir
         
    
##### 3. 进入虚拟环境
    

    1) 进入虚拟环境目录： cd E:\Projects\projectsedu.com
    2) 进入脚本目录：     cd bobbyvir\Scripts
    2) 运行activate.bat:  activate.bat

![image](http://of66as8gb.bkt.clouddn.com/blog/20161017/155414943.png)

查看虚拟环境中默认安装的库
    
    pip list
    
![image](http://of66as8gb.bkt.clouddn.com/blog/20161017/155612296.png)

##### 4. 虚拟环境下安装开发库， 这里以requests库为参考

    
    pip install request
    
![image](http://of66as8gb.bkt.clouddn.com/blog/20161017/155832879.png)

##### 5. 退出virtualenv

    deactivate.bat
    
![image](http://of66as8gb.bkt.clouddn.com/blog/20161017/155941435.png)


### virtualenvwrapper

上面每次进入virtual我们都需要进入到virtualenv的目录下，一旦virtualenv过多，就蛋疼了，接下来隆重推荐virtualenvwrapper

##### 1. 安装virtualenvwrapper


    pip install virtualenvwrapper-win
    注： linux下运行pip install virtualenvwrapper

![image](http://of66as8gb.bkt.clouddn.com/blog/20161017/160742596.png)

设置WORK_HOME环境变量

![mark](http://of66as8gb.bkt.clouddn.com/blog/20161017/161216426.png)
    
##### 2. 新建虚拟环境

    mkvirtualenv bobbyvir

![mark](http://of66as8gb.bkt.clouddn.com/blog/20161017/161124020.png)

    注：因为前一步设置了WORK_HOME，所有虚拟环境将安装到 E:\virtualevn

##### 3. 查看安装的所有虚拟环境

    workon

![mark](http://of66as8gb.bkt.clouddn.com/blog/20161017/161419843.png)

    注： 这里不能查看到有virtualenv创建的虚拟环境，只能查看mkvirtualenv创建的虚拟环境

##### 4. 进入虚拟环境

    workon bobbyvir
    
![mark](http://of66as8gb.bkt.clouddn.com/blog/20161017/161536186.png)

##### 5. 退出虚拟环境

    deactivate

![mark](http://of66as8gb.bkt.clouddn.com/blog/20161017/161615326.png)

小伙伴有没有觉得so easy



最后欢迎大家观看我关于 django + xadmin的教程：

	http://coding.imooc.com/class/evaluation/78.html