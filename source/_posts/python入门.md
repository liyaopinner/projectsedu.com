---
title: python入门
date: 2016-10-14 13:57:27
tags: python
categories: django
---


本文主要介绍在centos7 下通过docker安装sentry

### **1. docker**

**安装(方法一)**

1.确保yum packages 是最新的

    $ sudo yum update
    
2.添加yum repo

    $ sudo tee /etc/yum.repos.d/docker.repo <<-'EOF'
    [dockerrepo]
    name=Docker Repository
    baseurl=https://yum.dockerproject.org/repo/main/centos/7/
    enabled=1
    gpgcheck=1
    gpgkey=https://yum.dockerproject.org/gpg
    EOF
    
3.安装docker

    $ sudo yum install docker-engine
4.启动docker

    $ sudo service docker start
5.验证docker已经启动

    $ sudo docker run hello-world
    
    Unable to find image 'hello-world:latest' locally
    latest: Pulling from hello-world
    a8219747be10: Pull complete
    91c95931e552: Already exists
    hello-world:latest: The image you are pulling has been verified.      Important: image verification is a tech preview feature and should not be relied on to provide security.
    Digest: sha256:aa03e5d0d5553b4c3473e89c8619cf79df368babd1.7.1cf5daeb82aab55838d
    Status: Downloaded newer image for hello-world:latest
    Hello from Docker.
    This message shows that your installation appears to be working correctly.

    To generate this message, Docker took the following steps:
     1. The Docker client contacted the Docker daemon.
     2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
            (Assuming it was not already locally available.)
     3. The Docker daemon created a new container from that image which runs the
            executable that produces the output you are currently reading.
     4. The Docker daemon streamed that output to the Docker client, which sent it
            to your terminal.

    To try something more ambitious, you can run an Ubuntu container with:
     $ docker run -it ubuntu bash

    For more examples and ideas, visit:
     http://docs.docker.com/userguide/


**安装(方法二)**

1.确保yum packages 是最新的

    $ sudo yum update
2.运行docker 安装脚本
    
    $ curl -fsSL https://get.docker.com/ | sh
3.启动docker服务器

    $ sudo service docker start
4.验证docker已经启动

    $ sudo docker run hello-world


### **2**. sentry

sentry 依赖的组件比较多 包括 redis、 postgresql、 outbound email

在安装sentry前请确保 docker 版本大于1.10

1.安装git

    $ sudo yum install git
2.下载docker镜像并构建容器
    
    $ git clone  https://github.com/getsentry/onpremise.git
    $ cd onpremise
    $ sudo make build

注： 所有命令都要以sudo权限运行 否则会报错docker 未启动

3.用docker安装sentry依赖的组件

###### Redis

    docker run \
    --detach \
    --name sentry-redis \
    redis:3.2-alpine

###### PostgreSQL
    
    docker run \
    --detach \
    --name sentry-postgres \
    --env POSTGRES_PASSWORD=secret \
    --env POSTGRES_USER=sentry \
    postgres:9.5

###### Outbound Email
    
    docker run \
    --detach \
    --name sentry-smtp \
    tianon/exim4


注意：接下来所有命令都需要用到 Redis、 PostgreSQL、 Outbound Email中的环境变量，所有命令中需要将将三个镜像连接起来

    $ sudo docker run \
    --detach \
    --rm \
    --link sentry-redis:redis \
    --link sentry-postgres:postgres \
    --link sentry-smtp:smtp \
    --env SENTRY_SECRET_KEY=${SENTRY_SECRET_KEY} \
    ${REPOSITORY} \
    <command>
    
    其中 SENTRY_SECRET_KEY 可以自己生成

4.在PostgreSQL中生成sentry需要的表
    
    $ sudo docker run \
    --detach \
    --rm \
    --link sentry-redis:redis \
    --link sentry-postgres:postgres \
    --link sentry-smtp:smtp \
    --env SENTRY_SECRET_KEY=${SENTRY_SECRET_KEY} \
    -it sentry-onpremise upgrade
    
    在创建过程中会提示创建一个superuser， 根据提示自动输入邮箱和密码，该账户和密码很重要， 在sentry部署好以后需要用该账号登录， 请必须记住账号和密码

5.拉起sentry需要的后台服务

    $ sudo docker run \
    --detach \
    --rm \
    --link sentry-redis:redis \
    --link sentry-postgres:postgres \
    --link sentry-smtp:smtp \
    --env SENTRY_SECRET_KEY=${SENTRY_SECRET_KEY} \
    --name sentry-worker-01 \
    sentry-onpremise run worker
6.拉起sentry需要的cron后台服务

    $ sudo docker run \
    --detach \
    --rm \
    --link sentry-redis:redis \
    --link sentry-postgres:postgres \
    --link sentry-smtp:smtp \
    --env SENTRY_SECRET_KEY=${SENTRY_SECRET_KEY} \
    --name sentry-cron \
    sentry-onpremise run cron
7.最后拉起sentry的web服务
    
    $ sudo docker run \
    --detach \
    --rm \
    --link sentry-redis:redis \
    --link sentry-postgres:postgres \
    --link sentry-smtp:smtp \
    --env SENTRY_SECRET_KEY=${SENTRY_SECRET_KEY} \
    --name sentry-web-01 \
    --port 9000:9000 \
    sentry-onpremise \
    run web

8. 关闭注册
    
    修改 server.py 
    SENTRY_FEATURES['auth:register'] = True
    
    注：不同版本的docker可能会在--port 参数上有报错，如果出错可以尝试-p 或者--p 同时有些会提示 --d和-rm冲突，去掉--detach即可

最后在浏览器中访问 http://localhost:9000/

注意启动顺序 woker->cron->web, 如果不启动worker和cron可能会遇到报错如下：
![image](http://note.youdao.com/favicon.ico)






