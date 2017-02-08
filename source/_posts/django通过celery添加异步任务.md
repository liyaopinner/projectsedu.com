---
title: django通过celery添加异步任务
date: 2017-02-08 10:41:06
tags:
---


### 异步任务的重要性

   大家在做web项目的时候经常会遇到一些耗时的操作， 比如： 发送邮件、发送短信、生成pdf。这些操作在某些情况下需要立即返回结果给用户，但是可以在后台异步执行。

   比如用户邮箱注册的时候， 在发送邮件的时候可以先把"已经发送激活邮件到邮箱"返回给用户， 同时把邮件发送任务提交到异步处理线程中。
  
   现在介绍一款python写的专门用于处理异步任务的框架--celery。当然celery能完成的功能远不止异步任务， 还有一个很常用的功能--定时任务
    
   celery的功能还包括：定义工作流、监控、任务流控制、资源泄露保护以及自定义用户组件等。
    
### celery介绍

    说明：
        最新版本的celery支持的python版本必须大于2.7.6，
        如果是python2.7.6及以下版本的时候import celery是会报错滴。
    
   celery是通过将代码序列然后传输到中间通信组件，这些组件可以采用任何方式实现， 这里最常用的两种是rabbitmq和redis， 然后celery的后台线程不停的从rabbitmq或者redis中读取这些任务并执行然后返回结果到这些组件，这样就实现了一个异步的功能。
        
   Celery 用redis或者rabbitmq做消息通信，这里redis或者rabbitmq被称为中间人（Broker）Celery 系统可包含多个线程和中间人，以此获得高可用性和横向扩展能力。

   Celery 虽然是用 Python 编写的，但协议可以用任何语言实现。迄今，已有 Ruby 实现的 RCelery 、node.js 实现的 node-celery 以及一个 PHP 客户端 ，语言互通也可以通过 using webhooks 实现。

### django 介绍
    
   django作为python最主流也是资格最老的的web开发系统，是一个全栈的开发框架，几乎web开发系统中会用到的所有功能django都有，即使没有也可以在网站找到对应的开源解决方案，在stackoverflow上的问答也是最多的。基本上学习懂了django以后学习其他如flask、tornado都会觉得手到擒来。
    
   本文中我们就介绍一下如何将celery集成到django中来完成django耗时任务的异步执行和定时任务计划。
    
   我们将采用redis来做为中间人

###celery 安装和使用
   >  celery安装
    
    pip install -U celery[redis]
    
   该命令会安装celery以及redis开发相关所有的依赖包。安装完成我们可以看到：
![image](http://okwbmb2ka.bkt.clouddn.com/%E5%AE%89%E8%A3%85celery%E5%90%8E%E7%9A%84%E6%88%AA%E5%9B%BE.png)
    这里我们可以看到安装了billiard、pytz、vine、amqp、redis、celery等
    
   >   redis-server安装
    
   既然是用redis做中间人，当然需要安装redis了、我们直接运行：
        
            sudo apt-getin install redis-server
        
   运行成功以后可以，redis-server直接就作为服务启动了， 我们可以通过：
            
            ps aux|grep redis
   命令来查看redis是否启动如下图：
![image](http://okwbmb2ka.bkt.clouddn.com/%E6%9F%A5%E7%9C%8Bredis%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%AF%E5%8A%A8%E6%83%85%E5%86%B5.png)

   这里我们可以看到redis已经在6379端口监听了
    
>  启动celery的worker
    
   前面介绍了celery的处理流程， 既然我们已经启动了redis， 当然我们需要启动一个随时监听异步处理函数的worker了。 这里我们直接启动celery的worker就行了
        
   首先我们来新建一个tasks.py 文件， 内容如下：
        
        from celery import Celery
    
        app = Celery('hello', broker='redis://localhost:6379/0')
    
        @app.task
        def hello():
            return 'hello world'
        
   然后我们允许下面的命令启动celery的worker
        
        celery -A tasks worker --loglevel=info
    
   注：

            这里tasks表示的是上面创建的文件的名字， 比如如果我们的py文件为tasks.py, 则直接 celery -A tasks worker --loglevel=info。 如果我们的py文件为celery-tasks这命令应该修改为：
            celery -A celery-tasks worker --loglevel=info
        
   启动后我们就可以看到celery已经启动了线程时刻监听redis中的异步函数，如下：
![image](http://okwbmb2ka.bkt.clouddn.com/celery%E5%90%AF%E5%8A%A8%E6%88%AA%E5%9B%BE.png)
        
   接下来我们分析一下上面的tasks.py文件：

        1. 首先直接初始化Celery对象， 并指明使用的redis的连接地址
        2. 直接用celery对象的task装饰任何我们需要异步的函数
    简单两步就完成了celery的异步函数
    
 >  直接执行异步函数
        
   这一步里面我们直接新建test.py文件， 内容如下：
   
            from tasks import add
            
            add.delay(1,2)
        
   注意这里对add函数的调用采用的是delay函数而不是直接采用add(1,2),因为这样调用就和普通函数调用没有区别了。所以这里一定要注意。运行test.py文件后我们可以看到celery的输出：
        
![image](http://okwbmb2ka.bkt.clouddn.com/celery%E6%8E%A5%E6%94%B6%E5%88%B0celery%E7%9A%84%E6%88%AA%E5%9B%BE.png)
    
   在最后面我们可以清楚的看到调用了add函数， add函数的执行结果会返回到redis中
    这里delay函数是将函数执行异步放入到redis中交给celery执行， 这样delay之后就会有个问题就是如果我们需要理解得到结果怎么办呢？
    
   我们可以直接调用：

        add.delay(1,2).get()
   这样就变成同步的了，等到返回结果才会去执行下一步
    
>  celery添加异步任务
    
  celery的使用非常简单   
    
  这里我们可以看到需要将一个函数变为异步函数非常简单， 只需要添加@app.task装饰器就够了。 是不是非常简单啊。
    
    
   1. 配置celery连接redis
    
        app.conf.result_backend = 'redis://localhost:6379/0'
    
   2. 配置任务执行结果保存地址
    
        app.conf.result_backend = 'redis://localhost:6379/0'
    
    前面我们讲到过celery是从中间人取出函数并执行，但是保存结果也需要保存到中间人， 这里实际上取任务的地方和保存结果的中间人实际上可以不一样， 所有这里就提供了中间结果执行的保存地址


### 集成celery到django中

   *这里以我的一门django搭建在线教育平台的课程为例来讲解，大家如果有兴趣可以去关注一下，课程[强力django+杀手级xadmin][2]*

   首先我们来看一下完整的系统结构图：
![image](http://okwbmb2ka.bkt.clouddn.com/%E7%B3%BB%E7%BB%9F%E7%BB%93%E6%9E%84.png)
    
   1.修改django项目的MxOnline/settings.py文件， 加上：
        
        ###配置Broker
        BROKER_URL = 'redis://127.0.0.1:6379/0'
        BROKER_TRANSPORT = 'redis'
    
   2.在MxOline下面新建celery.py文件
        
        from __future__ import absolute_import

        import os
        import django
        
        from celery import Celery
        from django.conf import settings
        
        os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'MxOnline.settings')
        django.setup()
        
        app = Celery('MxOnline')
        
        app.config_from_object('django.conf:settings')
        app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)
    
结构如图所示：

![image](http://okwbmb2ka.bkt.clouddn.com/%E6%B7%BB%E5%8A%A0celerypy%E6%96%87%E4%BB%B6%E5%90%8E.png)

   3.在对应的app下面新建tasks.py文件， 这里我们在users这个app下面新建， 如图所示：
        
![image](http://okwbmb2ka.bkt.clouddn.com/app%E4%B8%8B%E6%96%B0%E5%BB%BAtasks.png)
    
   文件源码如下：
    
    from MxOnline.celery import app

    @app.task
    def send_register_email(email, send_type="register"):
        email_record = EmailVerifyRecord()
        if send_type == "update_email":
            code = random_str(4)
        else:
            code = random_str(16)
        email_record.code = code
        email_record.email = email
        email_record.send_type = send_type
        email_record.save()
    
        email_title = ""
        email_body = ""
    
        if send_type == "register":
            email_title = "慕学在线网注册激活链接"
            email_body = "请点击下面的链接激活你的账号: http://www.imooc.com/active/{0}".format(code)
    
            send_status = send_mail(email_title, email_body, EMAIL_FROM, [email])
            if send_status:
                pass
        elif send_type == "forget":
            email_title = "慕学在线网注册密码重置链接"
            email_body = "请点击下面的链接重置密码: http://www.imooc.com/reset/{0}".format(code)
    
            send_status = send_mail(email_title, email_body, EMAIL_FROM, [email])
            if send_status:
                pass
        elif send_type == "update_email":
            email_title = "慕学在线邮箱修改验证码"
            email_body = "你的邮箱验证码为: {0}".format(code)
    
            send_status = send_mail(email_title, email_body, EMAIL_FROM, [email])
            if send_status:
                pass
    
    4. 编辑views.py文件完成邮件发送异步调用：
    
        #coding:utf-8
        from django.shortcuts import render
        from django.http import HttpResponse
        
        from .tasks import send_register_email
        
        def index(request):
            send_register_email.delay()
            return HttpResponse(u"邮件发送成功， 请查收")
    
    5. 进入MxOnline目录运行：
        celery -A demo worker -l debug
    
        以此来启动celery的worker服务

*关于django是如何实现邮件发送以及如何配置邮件的发送方配置，在课程 [强力django+杀手级xadmin][http://coding.imooc.com/class/78.html]中我会详细讲解，并包含django实现cookie和session的登录原理。*
    
至此，大功告成了！我们可以在我们定义的任何apps中添加tasks来定义需要的异步任务。


  [1]: http://coding.imooc.com/class/78.html
  [2]: http://coding.imooc.com/class/78.html
  [3]: http://coding.imooc.com/class/78.html