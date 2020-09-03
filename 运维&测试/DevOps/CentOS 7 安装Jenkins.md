原文链接：https://segmentfault.com/a/1190000004639325

`Jenkins`是开源的,使用`Java`编写的持续集成的工具，在Centos上可以通过`yum`命令行直接安装。记录下安装的过程，方便以后查找。需要先安装`Java`,如果已经`Java`可以跳过该步骤。

## 1. 安装Java

看到当前系统`Java`版本的命令:

> java -version

如果显示`Java`版本号，说明已经正确安装，如果显示没有该命令，需要安装`Java`：

> sudo yum install java

该命令如果检测到`Java`不存在可以直接安装`Java`,如果已存在则可以升级`Java`。

## 2. 安装Jenkins

首先要先添加`Jenkins`源:

> sudo wget -O /etc/yum.repos.d/jenkins.repo [http://jenkins-ci.org/redhat/...](http://jenkins-ci.org/redhat/jenkins.repo)
> sudo rpm --import [http://pkg.jenkins-ci.org/red...](http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key)

添加完成之后直接使用`yum`命令安装`Jenkins`:

> yum install jenkins -y

另外，可以**直接下载rpm安装**。

rpm下载地址：https://pkg.jenkins.io/redhat-stable/

清华大学镜像源：https://mirrors.tuna.tsinghua.edu.cn/jenkins/

```bash
wget https://pkg.jenkins.io/redhat/jenkins-2.156-1.1.noarch.rpm
rpm -ivh jenkins-2.156-1.1.noarch.rpm
```

## 3. 启动Jenkins

使用命令启动`Jenkins`:

> sudo service jenkins start

```
Starting Jenkins                                           [  OK  ]
```

在浏览器中输入：[http://<服务器ip>:8080/](http:// 就可以进入`Jenkins`界面直接使用了 。
停止`Jenkins`服务的命令为：

> sudo service jenkins stop

## 4. 相关配置

`enkins`安装目录：

```
 /var/lib/jenkins/
```

Jenkins配置文件地址：

```
/etc/sysconfig/jenkins
```

这就是`Jenkins`的配置文件，可以在这里查看`Jenkins`默认的配置。

> cat jenkins

这里介绍下三个比较重要的配置：

- `JENKINS_HOME`
- `JENKINS_USER`
- `JENKINS_PORT`

`JENKINS_HOME`是Jenkins的主目录，Jenkins工作的目录都放在这里,Jenkins储存文件的地址,Jenkins的插件，生成的文件都在这个目录下。

```shell
## Path:        Development/Jenkins
## Description: Jenkins Continuous Integration Server
## Type:        string
## Default:     "/var/lib/jenkins"
## ServiceRestart: jenkins
#
# Directory where Jenkins store its configuration and working
# files (checkouts, build reports, artifacts, ...).
#
JENKINS_HOME="/var/lib/jenkins"
```

`JENKINS_USER `是Jenkins的用户，拥有$JENKINS_HOME和/var/log/jenkins的权限。

```shell
## Type:        string
## Default:     "jenkins"
## ServiceRestart: jenkins
#
# Unix user account that runs the Jenkins daemon
# Be careful when you change this, as you need to update
# permissions of $JENKINS_HOME and /var/log/jenkins.
#
JENKINS_USER="jenkins"
```

如果为了不因为权限出现各种问题，这里直接使用root用户。

```bash
vim /etc/sysconfig/jenkins

#修改配置
$JENKINS_USER="root"
```
修改目录权限

```
chown -R root:root /var/lib/jenkins
chown -R root:root /var/cache/jenkins
chown -R root:root /var/log/jenkins
```
重启
```
service jenkins restart
ps -ef | grep jenkins
```
`JENKINS_PORT `是Jenkins的端口，默认端口是8080。

```shell
## Type:        integer(0:65535)  
## Default:     8080
## ServiceRestart: jenkins
#
# Port Jenkins is listening on.
# Set to -1 to disable
#
JENKINS_PORT="8080"
```

## 4. 插件下载加速

以下的配置Json其实在Jenkins的工作目录中。

```bash
$ cd {你的Jenkins工作目录}/updates  #进入更新配置位置
```
**第一种方式：使用vim**。

```bash
$ vim default.json   #这个Json文件与上边的配置文件是相同的
```
这里wiki和github的文档不用改，我们就可以成功修改这个配置。

使用vim的命令，如下，替换所有插件下载的url。

```bash
:1,$s/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g
```
替换连接测试url。
```bash
:1,$s/http:\/\/www.google.com/https:\/\/www.baidu.com/g
```
> 进入vim先输入：然后再粘贴上边的：后边的命令，注意不要写两个冒号！

修改完成保存退出`:wq`。

**第二种方式：使用sed**。

```bash
$ sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json && sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json
```
> 这是直接修改的配置文件，如果前边Jenkins用sudo启动的话，那么这里的两个`sed`前均需要加上`sudo`。

重启Jenkins，安装插件试试，简直超速！！

## 5. 常见问题

### 5.1 [Jenkins]Error:403 No valid crumb was included in the request

配置 `jenkins` 的时候，一直报这个错，是因为 `jenkins` 默认安全设置里面开启了 `防止款站点请求伪造`。

方法：

> 取消勾选这一项，就可以成功集成了。

位置： `Jenkins > 全局安全配置 > CSRF Protection`

![2020-06-01-APcqST](https://image.ldbmcs.com/2020-06-01-APcqST.jpg)

