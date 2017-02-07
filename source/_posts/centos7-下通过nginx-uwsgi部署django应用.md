---
title: centos7 下通过nginx+uwsgi部署django应用
date: 2017-02-07 14:03:36
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
    source /usr/local/bin/virtualenvwrapper.sh
    
>     重新加载.bashrc文件
    source  ~/.bashrc
    
>     新建虚拟环境
    mkvirtualenv mxoline

>     进入虚拟环境 
    workon mxoline

>     安装pip包
    
    我们可以通过 pip freeze > requirements.txt 将本地的虚拟环境安装包相信信息导出来
    
    然后将requirements.txt文件上传到服务器之后运行：
    
    pip install -r requirements.txt
    安装依赖包
    
3. 安装uwsgi
    
    pip install uwsgi

4. 测试uwsgi
    
    uwsgi --http :8000 --module MxOnline.wsgi

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
    
这里需要注意 一定是直接用nginx命令启动， 不要用systemctl启动nginx不然会有权限问题

9. 通过配置文件启动uwsgi

    新建uwsgi.ini 配置文件， 内容如下：
        
        # mysite_uwsgi.ini file
        [uwsgi]
        
        # Django-related settings
        # the base directory (full path)
        chdir           = /home/bobby/Projects/MxOnline
        # Django's wsgi file
        module          = MxOnline.wsgi
        # the virtualenv (full path)
        
        # process-related settings
        # master
        master          = true
        # maximum number of worker processes
        processes       = 10
        # the socket (use the full path to be safe
        socket          = 127.0.0.1:8000
        # ... with appropriate permissions - may be needed
        # chmod-socket    = 664
        # clear environment on exit
        vacuum          = true
        virtualenv = /home/bobby/.virtualenvs/mxonline
    
    注：
        chdir： 表示需要操作的目录，也就是项目的目录
        module： wsgi文件的路径
        processes： 进程数
        virtualenv：虚拟环境的目录
            
    
    workon mxonline
    uwsgi -i 你的目录/Mxonline/conf/uwsgi.ini &

10 访问
    
    http://你的ip地址/
	
最后欢迎大家观看我关于 django + xadmin的教程：

	http://coding.imooc.com/class/evaluation/78.html
    
    
