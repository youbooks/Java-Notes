原文链接：https://juejin.im/post/5dabec98e51d4524c06047aa

---

> rabbitMQ是一款开源的高性能消息中间件，最近项目要使用，于是使用docker搭建，快速方便。

## 1. 获取镜像

1. 使用`docker search rabbitMq`命令获取镜像列表。

	![2020-06-09-GSb5J7](https://image.ldbmcs.com/2020-06-09-GSb5J7.jpg)

2. 使用`docker pull docker.io/rabbitmq:3.8-management` 拉取镜像

	![2020-06-09-A9cP1P](https://image.ldbmcs.com/2020-06-09-A9cP1P.jpg)

我们选择了STARS数最多的官方镜像，此处需要注意，默认rabbitmq镜像是不带web端管理插件的，所以指定了镜像tag为3.8-management，表示下载包含web管理插件版本镜像，其它Tag版本可以访问DockerHub查询。

## 2. 创建RabbitMQ容器

1. 使用`docker images`获取查看rabbitMQ镜像ID，我的是`4b23cfb64730`

	![2020-06-09-0TR4NC](https://image.ldbmcs.com/2020-06-09-0TR4NC.jpg)

2. 执行`docker run --name rabbitmq -d -p 15672:15672 -p 5672:5672 4b23cfb64730`命令创建rabbitMq容器，关于其中的参数含义如下：

   - --name指定了容器名称
   - -d 指定容器以后台守护进程方式运行
   - -p指定容器内部端口号与宿主机之间的映射，rabbitMq默认要使用15672为其web端界面访问时端口，5672为数据通信端口

   命令执行完毕后，docker会使用ID为 `4b23cfb64730`的镜像创建容器，创建完成后返回容器ID为`3ae75edc48e2416292db6bcae7b1054091cb....(太长省略)`

   执行`docker ps`可以查看正在运行的容器，我们能看到rabbitMq已经运行。
   
   ![2020-06-09-g3XUgk](https://image.ldbmcs.com/2020-06-09-g3XUgk.jpg)

3. 查看容器日志 使用`docker logs -f 容器ID`命令可以查看容器日志，我们执行`docker logs -f 3ae`命令查看rabbitMq在启动过程中日志，3ae是容器ID的简写——容器ID太长，使用时其写前几位即可。

	![2020-06-09-DZzp9O](https://image.ldbmcs.com/2020-06-09-DZzp9O.jpg)

   从日志可以看出，rabbitMq默认创建了guest用户，并且赋予administrator角色权限，同时服务监听5672端口TCP连接和15672端口的HTTP连接，至此说明安装成功。

## 3. 访问RabbitMq

### 3.1 访问web界面

在浏览器 输入你的`主机Ip:15672`回车即可访问rabbitMq的Web端管理界面，默认用户名和密码都是`guest`，如图出现如下界面代表已经成功了。

![2020-06-09-7AoSqQ](https://image.ldbmcs.com/2020-06-09-7AoSqQ.jpg)

### 3.2 新添加一个账户

默认的`guest` 账户有访问限制，默认只能通过本地网络(如 localhost) 访问，远程网络访问受限，所以在使用时我们一般另外添加用户，例如我们添加一个root用户：

1. 执行`docker exec -i -t 3ae bin/bash`进入到rabbitMq容器内部。

   ```bash
   [root@localhost docker]# docker exec -i -t 3a bin/bash
   root@3ae75edc48e2:/# 
   ```

2. 执行`rabbitmqctl add_user root 123456` 添加用户，用户名为root,密码为123456。

   ```bash
   root@3ae75edc48e2:/# rabbitmqctl add_user root 123456 
   Adding user "root" ...
   ```

3. 执行`abbitmqctl set_permissions -p / root ".*" ".*" ".*"` 赋予root用户所有权限。

   ```bash
   root@3ae75edc48e2:/# rabbitmqctl set_permissions -p / root ".*" ".*" ".*"
   Setting permissions for user "root" in vhost "/" ...
   ```

4. 执行`rabbitmqctl set_user_tags root administrator`赋予root用户administrator角色。

   ```bash
   root@3ae75edc48e2:/# rabbitmqctl set_user_tags root administrator
   Setting tags for user "root" to [adminstrator] ...
   ```

5. 执行`rabbitmqctl list_users`查看所有用户即可看到root用户已经添加成功。

   ```bash
   root@3ae75edc48e2:/# rabbitmqctl list_users
   Listing users ...
   user	tags
   guest	[administrator]
   root	[administrator]
   ```

执行`exit`命令，从容器内部退出即可。这时我们使用root账户登录web界面也是可以的。到此，rabbitMq的安装就结束了，接下里就实际代码开发。

