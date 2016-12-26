---
title: linux下安装mysql-python出错
date: 2016-10-20 09:59:54
tags: 慕学在线网
---

命令

    
    pip install mysql-python

然后出错了：

    
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/tmp/pip-build-YEYPJp/mysql-python/setup.py", line 17, in <module>
        metadata, options = get_config()
      File "setup_posix.py", line 43, in get_config
        libs = mysql_config("libs_r")
      File "setup_posix.py", line 25, in mysql_config
        raise EnvironmentError("%s not found" % (mysql_config.path,))
    EnvironmentError: mysql_config not found

解决方法：


    sudo apt-get install libmysqlclient-dev


然后重新安装就ok了