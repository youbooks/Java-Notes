# Docker开启Remote API 访问 2375端口

> 转载：[Docker开启Remote API 访问 2375端口](https://www.cnblogs.com/hongdada/p/11512901.html)

## 1. Docker常见端口

我看到的常见docker端口包括：

> 2375：未加密的docker socket,远程root无密码访问主机
> 2376：tls加密套接字,很可能这是您的CI服务器4243端口作为https 443端口的修改
> 2377：群集模式套接字,适用于群集管理器,不适用于docker客户端
> 5000：docker注册服务
> 4789和7946：覆盖网络

## 2. 开启配置

### 2.1 方法一

首先是怎么配置远程访问的API：

```bash
sudo vim /etc/default/docker
```

加入下面一行:

```bash
DOCKER_OPTS="-H tcp://0.0.0.0:2375"
```

重启docker即可：

```bash
sudo systemctl restart docker
```

> PS:这是网上给的配置方法，也是这种简单配置让Docker Daemon把服务暴露在tcp的2375端口上，这样就可以在网络上操作Docker了。Docker本身没有身份认证的功能，只要网络上能访问到服务端口，就可以操作Docker。

### 2.2 方法二

在`/usr/lib/systemd/system/docker.service`，配置远程访问。

主要是在[Service]这个部分，加上下面两个参数：

```bash
# vim /usr/lib/systemd/system/docker.service
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
```

重启:

```bash
systemctl daemon-reload
systemctl restart docker
```

### 2.3 方法三

下面修改`daemon.json`的配置：

```bash
vim /etc/docker/daemon.json

{
  "hosts": ["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"]
}
```

查看docker进程：

```bash
[root@slaver2 ~]# ps -ef|grep docker
root      44221      1  1 18:16 ?        00:00:06 /usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
```

Docker守护进程打开一个HTTP Socket,这样才能实现远程通信。

## 3. 简单使用

`-H`为连接目标主机docker服务

查看docker版本：

```bash
[root@slaver2 /]# docker -H tcp://18.16.202.95:2375 version
Client: Docker Engine - Community
 Version:           19.03.0
 API version:       1.40
 Go version:        go1.12.5
 Git commit:        aeac9490dc
 Built:             Wed Jul 17 18:15:40 2019
 OS/Arch:           linux/amd64
 Experimental:      false
Cannot connect to the Docker daemon at tcp://18.16.202.95:2375. Is the docker daemon running?
```

查看镜像包：

```bash
[root@slaver2 ~]# docker -H tcp://18.16.202.95:2375 images
REPOSITORY                                 TAG                 IMAGE ID            CREATED             SIZE
zookeeper                                  3.5.5               3487af26dee9        4 weeks ago         225MB
k8s.gcr.io/kube-apiserver                  v1.15.1             68c3eb07bfc3        8 weeks ago         207MB
k8s.gcr.io/kube-scheduler                  v1.15.1             b0b3c4c404da        8 weeks ago         81.1MB
k8s.gcr.io/kube-proxy                      v1.15.1             89a062da739d        8 weeks ago         82.4MB
k8s.gcr.io/kube-controller-manager         v1.15.1             d75082f1d121        8 weeks ago         159MB
quay.io/coreos/flannel                     v0.11.0-amd64       ff281650a721        7 months ago        52.6MB
k8s.gcr.io/coredns                         1.3.1               eb516548c180        8 months ago        40.3MB
k8s.gcr.io/etcd                            3.3.10              2c4adeb21b4f        9 months ago        258MB
quay.io/jetstack/cert-manager-controller   v0.5.2              2e4d862afebb        9 months ago        47.3MB
confluentinc/cp-kafka                      5.0.1               5467234daea9        10 months ago       557MB
k8s.gcr.io/pause                           3.1                 da86e6ba6ca1        21 months ago       742kB
radial/busyboxplus                         curl                71fa7369f437        4 years ago         4.23MB
```

## 4. 参考

1. [Docker Remote API的安全配置](https://p0sec.net/index.php/archives/115/)

2. [Docker 2375 端口入侵服务器](http://www.dockerinfo.net/1416.html)

3. [远程连接docker daemon，Docker Remote API](https://deepzz.com/post/dockerd-and-docker-remote-api.html)

4. [远程访问 Docker Daemon](https://blog.csdn.net/cao0507/article/details/83043485)

