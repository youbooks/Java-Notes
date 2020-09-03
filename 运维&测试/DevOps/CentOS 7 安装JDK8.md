## 1. 开发环境

腾讯云服务器 CentOS 7.2 64位

## 2. 检查系统是否已自带open-jdk

```shell
rpm -qa | grep java 或者 rpm -qa | grep jdk
```

**ps:**如果没有显示则说明尚未安装jdk，若是已经安装可以通过命令`rpm -qa | grep java | xargs rpm -e --nodeps`批量卸载所有带java的文件

## 3. 安装openjdk(无需配置path环境变量)

### 3.1 检索yum源中的jdk包

- `yum list java*`
- `yum list java-1.8*(指定搜索1.8版本)`

通过以上命令可以检索出当前yum源中openjdk版本为1.8.0。

![](https://image.ldbmcs.com/2019-09-18-090801.jpg)

### 3.2 安装1.8版本的open-jdk

这里我们需要安装java-1.8版本的所有文件，安装命令：

```
yum install -y java-1.8.0-openjdk*
```

### 3.3 检查是否安装成功

`java -version`

![](https://image.ldbmcs.com/2019-09-18-090841.jpg)

出现以上类似信息则说明安装成功了。
