# Docker 入门教程

## 1. Docker简介

### 开源的容器引擎，将应用程序和基础设施层隔离，使用Docker可以更快的打包，测试和部署应用程序。https://www.docker.com/

### 版本

- Docker EE：企业版
- Docker CE：社区版

### 架构

- Docker daemon：Docker守护进程
- Client：Docker 客户端
- Images：Docker 镜像
- Container：Docker 容器
- Registry：集中存储与分发镜像的服务

### Docker VS 虚拟机

- 虚拟机利用Hypervisor虚拟化CPU，内存，IO设备等实现的，然后在其上运行完整的操作系统，在该系统上运行所需要的应用。资源隔离级别：OS级别
- 运行在Docker容器中的应用直接运行于宿主机的内核，容器共享宿主机的内核。

### 应用场景

- 简化配置
- DevOps
- 提高开发效率
- 隔离应用
- 整合服务器
- 调试能力
- 多租户
- 快速部署

## 2. Docker安装

### CentOS

- yum安装
- shell一键安装

### Ubuntu

- apt-get安装

### MacOS

- 安装包一键安装

### Windwos

- docker for windwos

## 3. Docker 镜像

### 镜像加速器

- 阿里云加速器

### 镜像常用命令

- 搜索镜像：docker search
- 下载镜像：docker pull
- 本地镜像列表：docker images
- 删除本地镜像：docker rmi
- 构建镜像：docker build
- 保存镜像：docker save
- 加载镜像：docker load

## 4. Docker 容器

### 容器常用命令

- 新建启动容器：docker run
- 本地容器列表：docker ps
- 停止容器：docker stop 容器id
- 强制停止容器：docker kill 容器id
- 启动已停止的容器：docker start 容器id
- 重启容器：docker restart 容器id
- 进入容器：docker attach 容器id，docker nsenter ，docker exec
- 删除容器：docker rm 容器id
- 导出容器：docker export
- 导入容器：docker import

### 容器互联

- 使用 --link 参数即可实现容器之间的互联。该参数的格式为: --link name:alias ，其中，name是容器的名称，alias则是这个连接的别名。

### 容器网络(略)

## 5. Dockerfile指令详解

### ADD 复制文件：从宿主机src目录复制文件到容器的指定目录中

### ARG 设置构建参数：类似于ENV，ARG设置的是构建时的环境变量，在容器运行时是不会存在这些变量的。

### CMD 容器启动命令

### COPY 复制文件，和ADD类似，COPY不支持URL和压缩包

### ENTRYPOINT 入口点：和CMD指令的目的一样，都是指定容器启动时执行的命令，可多次设置，但只有最后一个有效。

### ENV 设置环境变量

### EXPOSE 申明暴露的端口：申明在运行时容器提供服务的端口。

### FROM 指定基础镜像

### LABEL：为镜像添加元数据

### RUN 执行命令

### USER 设置用户

### VOLUME 指定挂载点

### WORKDIR 指定工作目录

## 6. Docker Registry

### 使用Docker Hub管理镜像(略)

### 使用Docker Registry管理镜像(略)

### 使用Nexus管理Docker镜像(略)

## 7. Docker 可视化管理工具

### DockerUI(ui for Docker)：https://github.com/kevana/ui-for-docker

### Portainer：https://github.com/portainer/portainer

### Kitematic：https://github.com/docker/kitematic

### Shipyard：https://github.com/shipyard/shipyard

### 各种可视化界面的比较：http://m.blog.csdn.net/qq273681448/article/details/75007828

## 8. Docker数据持久化

### 数据卷

- 数据卷是一个可供一个或多个容器使用的特殊目录，可以绕过UFS(Unix File System)。
  
  - 数据卷可以在容器之间共享和重用
  - 对数据卷的修改会立⻢生效
  - 对数据卷的更新，不会影响镜像
  - 数据卷默认会一直存在，即使容器被删除
  - 一个容器可以挂载多个数据卷

- 创建数据卷： docker run --name nginx-data -v /mydir nginx

- 删除数据卷： docker rm -v 容器ID
  
  - 即使容器被删除，宿主机中的目录也不会被删除。

### 数据卷容器

- 如果有数据需要在多个容器之间共享，此时可考虑使用数据卷容器。
- 创建数据卷容器： docker run --name nginx-volume -v /data nginx

## 9. 端口映射

### 随机映射：-P

### 指定端口映射：-p

- ip:hostPort:containerPort:映射到指定IP的指定端口
- ip::containerPort:映射到指定IP的随机端口
- hostPort:containerPort:映射到宿主机所有IP的指定端口
- containerPort:映射到宿主机所有IP的随机端口

### 查看端口映射

- docker ps
- docker port 容器ID

## 10. Docker Compose

### 简介

- Compose是一个用于定义和运行多容器Docker应用程序的工具，前身是Fig。它非常适合用在开发、测试、构建CI工作流等场景。

### 安装

- 安装Compose：shell，pip以及作为docker容器安装等方式。

- 安装Compose命令补全工具
  
  - Bash
  - Zsh

### 快速入门

- 基本步骤
  
  - 使用Dockerfile(或其他方式)定义应用程序环境，以便在任何地方重现该环境。
  - 在docker-compose.yml文件中定义组成应用程序的服务，以便各个服务在一个隔离的环境中一起运行。
  - 运行docker-compose up命令，启动并运行整个应用程序。

- 工程、服务、容器
  
  - Docker Compose运行目录下的所有文件组成一个工程，一个工程可能包含多个服务，每个服务定了容器运行的镜像，参数和依赖，一个服务包括多个容器实例。

### docker-compose.yml 常用命令

- build：Compose会利用它自动构建镜像。
- command：覆盖容器启动后默认执行的命令。
- dns：配置dns服务器。
- dns_search：配置dns的搜索域名。
- environment：环境变量设置。
- env_file：从文件中获取环境变量。
- expose：暴露端口，只将端口暴露给连接的服务，不暴露给宿主机。
- external_links：连接到docker-compose.yml外部的容器
- image：指定镜像名称或镜像id
- links：连接到其他服务的容器
- networks
- network_mode：设置网络模式
- ports：暴露端口信息，可使用HOST:CONTAINER的格式
- volumes：卷挂载路径设置。
- volumes_from：从另一个服务或容器挂载卷。

### docker-compose 常用命令

- build：构建或重新构建服务。服务构建后将以project_service的形式标记。
- help：查看指定命令的帮助文档。
- kill：通过发送kill命令停止指定服务的容器。
- logs：查看服务的日志输出。
- port：打印绑定的公共端口。
- ps：列出所有容器。
- pull：下载服务镜像
- rm：删除指定服务的容器。
- run：在一个服务商执行一个命令。
- scale：设置指定服务运行容器的个数
- start：启动指定福谁已存在的容器
- stop：停止已运行的容器
- up：构建，创建，重新创建，启动，连接服务的相关容器

### 网络设置

- Compose会为我们的应用创建一个网络，服务的每个容器都会加入该网络中。这样，容器就可被该网络中的其他容器访问，不仅如此，该容器还能以服务名称作为hostname被其他容器访问。
- 应用程序的网络名称基于Compose的工程名称，而项目名称基于docker-compose.yml所在目录的名称。如需修改工程名称，可使用--project-name标识或COMPOSE_PORJECT_NAME环境变量。

### 控制服务启动顺序

- wait-for-it
- dockerrize
- wait-for
