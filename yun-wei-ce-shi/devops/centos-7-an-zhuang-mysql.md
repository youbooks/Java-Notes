# CentOS 7 安装 MySQL

## 1. 系统环境

CentOS7.5 64位

## 2. 开始部署

### 2.1 添加mysql yum源

在[CentOS](https://www.centos.bz/tag/centos/)上直接使用`yum install mysql`安装，最后安装上的会是[MariaDB](https://www.centos.bz/tag/mariadb/)，所以要先添加[mysql](https://www.centos.bz/tag/mysql-2/) `yum`源。

```text
rpm -Uvh https://repo.mysql.com//mysql80-community-release-el7-2.noarch.rpm
```

### 2.2 安装（如果要安装最新版，可直接开始安装）

查看`yum`源中所有Mysql版本。

```text
yum repolist all | grep mysql
```

此时的最新版本是mysql8.0，把它禁用掉。

```text
yum -y install yum-utils
yum-config-manager --disable mysql80-community
```

[mysql5.7](https://www.centos.bz/tag/mysql5-7/)是我要安装的版本，启用mysql5.7。

```text
yum-config-manager --enable mysql57-community
```

检查刚才的配置是否生效。

```text
yum repolist enabled | grep mysql
```

开始安装。

```text
yum install -y mysql-community-server
```

### 2.3 启动服务

```text
service mysqld start
```

启动完成之后检查mysql状态。

```text
service mysqld status
```

查看临时密码。

```text
grep 'temporary password' /var/log/mysqld.log
```

登录。

```text
mysql -uroot -p
```

## 3. 其他

### 3.1 修改密码

```text
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
或者
SET password='new_password';
```

会提示密码不满足策略要求。

```text
mysql root@localhost:(none)> show VARIABLES like "%password%"
+---------------------------------------+---------+
| Variable_name                         | Value   |
|---------------------------------------+---------|
| default_password_lifetime             | 0       |
| disconnect_on_expired_password        | ON      |
| log_builtin_as_identified_by_password | OFF     |
| mysql_native_password_proxy_users     | OFF     |
| old_passwords                         | 0       |
| report_password                       |         |
| sha256_password_proxy_users           | OFF     |
| validate_password_dictionary_file     |         |
| validate_password_length              | 8       |
| validate_password_mixed_case_count    | 1       |
| validate_password_number_count        | 1       |
| validate_password_policy              | MEDIUM  |
| validate_password_special_char_count  | 1       |
+---------------------------------------+---------+
```

更改密码必须满足：数字、小写字母、大写字母 、特殊字符、长度至少8位。

### 3.2 远程连接

```text
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
```

### 3.3 binlog

```text
vim /etc/my.cnf
#添加
log-bin=mysql-bin
server-id=1
```

## 4. MySQL常用命令

> Systemd是一个系统管理守护进程、工具和库的集合，用于取代System V初始进程。主要负责控制systemd系统和服务管理器。

启动mysql服务

```text
systemctl start mysqld.service
// 启动同时查看启动日志
sudo systemctl start mysql --no-block; sudo journalctl -xefu mysql
```

停止mysql服务

```text
systemctl stop mysqld.service
```

重启mysql服务

```text
systemctl restart mysqld.service
```

查看mysql服务当前状态

```text
systemctl status mysqld.service
```

设置mysql服务开机自启动

```text
systemctl enable mysqld.service
```

停止mysql服务开机自启动

```text
systemctl disable mysqld.service
```

