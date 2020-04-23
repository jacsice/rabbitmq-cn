# 介绍

从docker拉取镜像

```docker pull rabbitmq:3-management```  带管理后台的镜像

启动镜像

`docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management` 

登录账号： guest  密码：guest

`docker log rabbitmq`  查看日志

默认端口是`5672`

