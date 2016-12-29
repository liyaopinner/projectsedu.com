---
title: centos7 下通过nginx+uwsgi部署django应用
date: 2016-12-29 14:03:36
categories: 
- django
- 慕学在线网
---

##### uwsgi + nginx 搭建
    

1. 安装nginx


    https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-centos-7
    
2. 安装virtualenvwrapper
    

    yum install python-setuptools python-devel
    pip install virtualenvwrapper
    
>     编辑.bashrc文件
    
    export WORKON_HOME=$HOME/.virtualenvs
    export PROJECT_HOME=$HOME/Devel
    source /usr/local/bin/virtualenvwrapper.sh
    
>     重新加载.bashrc文件
    source  ~/.bashrc
    
>     新建虚拟环境
    mkvirtualenv mxoline

>     进入虚拟环境 
    workon mxoline

>     安装pip包
    pip install -r requirements.txt
    
3. 安装uwsgi
    
    pip install uwsgi

4. 测试uwsgi
    
    uwsgi --http :8000 --module Mxoline.wsgi

5. 配置nginx
    
    新建uc_nginx.conf

    
    # the upstream component nginx needs to connect to
    upstream django {
    # server unix:///path/to/your/mysite/mysite.sock; # for a file socket
    server 127.0.0.1:8000; # for a web port socket (we'll use this first)
    }
    # configuration of the server

    server {
    # the port your site will be served on
    listen      80;
    # the domain name it will serve for
    server_name 你的ip地址 ; # substitute your machine's IP address or FQDN
    charset     utf-8;

    # max upload size
    client_max_body_size 75M;   # adjust to taste

    # Django media
    location /media  {
        alias 你的目录/Mxonline/media;  # 指向django的media目录
    }

    location /static {
        alias 你的目录/Mxonline/static; # 指向django的static目录
    }

    # Finally, send all non-media requests to the Django server.
    location / {
        uwsgi_pass  django;
        include     uwsgi_params; # the uwsgi_params file you installed
    }
    }

6. 将该配置文件加入到nginx的启动配置文件中
    
    sudo ln -s 你的目录/Mxonline/conf/nginx/uc_nginx.conf /etc/nginx/conf.d/

7. 拉取所有需要的static file 到同一个目录
    
    在django的setting文件中，添加下面一行内容：
    
        STATIC_ROOT = os.path.join(BASE_DIR, "static/")
    运行命令
        python manage.py collectstatic

8. 运行nginx
    
    sudo /usr/sbin/nginx

9. 运行nginx
    
    
    workon mxonline
    uwsgi -i 你的目录/Mxonline/conf/uwsgi &

10 访问
    
    http://你的ip地址/
	
最后欢迎大家观看我关于 django + xadmin的教程：

	http://coding.imooc.com/class/evaluation/78.html
    
    
