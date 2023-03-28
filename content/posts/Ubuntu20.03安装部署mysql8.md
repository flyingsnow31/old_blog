---
title: "Ubuntu20.03安装部署mysql8"
date: 2023-03-28T11:55:05+08:00
draft: true
tags: ['MySQL', 'Ubuntu']
categories: 软件安装与配置
---



## 安装MySQL

### 更新apt源

```bash
$sudo apt update
```

### 安装MySQL服务器

```bash
$sudo apt install mysql-server
```

### 安装MySQL服务端

```bash
$sudo apt install mysql-client
```

### 配置MySQL

```bash
$sudo mysql_secure_installation

Securing the MySQL server deployment.

Enter password for user root:
The 'validate_password' component is installed on the server.
The subsequent steps will run with the existing configuration
of the component.
Using existing password for root.

Estimated strength of the password: 100
Change the password for root ? ((Press y|Y for Yes, any other key for No) : y

New password:

Re-enter new password:

Estimated strength of the password: 50
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y
 ... Failed! Error: Your password does not satisfy the current policy requirements

New password:

Re-enter new password:

Estimated strength of the password: 100
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Success.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
Success.

By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
 - Dropping test database...
Success.

 - Removing privileges on test database...
Success.

Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.

All done!
```

### 修改密码

```bash
$sudo mysql

mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password by 'mynewpassword';
Query OK, 0 rows affected (0.02 sec)
```

### 测试安装

```bash
$sudo systemctl status mysql.service
● mysql.service - MySQL Community Server
     Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-03-28 12:11:41 CST; 14min ago
   Main PID: 156257 (mysqld)
     Status: "Server is operational"
      Tasks: 40 (limit: 4650)
     Memory: 369.8M
     CGroup: /system.slice/mysql.service
             └─156257 /usr/sbin/mysqld

3月 28 12:11:39 aim-virtual-machine systemd[1]: Starting MySQL Community Server...
3月 28 12:11:41 aim-virtual-machine systemd[1]: Started MySQL Community Server.
```

## MySQL服务使用

### 启动MySQL数据库服务

```bash
$sudo service mysql start
或
$sudo systemctl start mysql.service
```



### 重启MySQL数据库服务

```bash
$sudo service mysql restart
或
$sudo systemctl restart mysql.service
```



### 停止MySQL数据库服务

```bash
$sudo service mysql stop
或
$sudo systemctl stop mysql.service
```



### 查看MySQL运行状态

```bash
$sudo service mysql status
或
$sudo systemctl status mysql.service
```



### 设置MySQL服务开机自启动

```bash
$sudo service mysql enable
或
$sudo systemctl enable mysql.service
```



### 停止MySQL服务开机自启动

```bash
$sudo service mysql disable
或
$sudo systemctl disable mysql.service
```



### 配置MySQL远程登录

有时候，为了开发方便，我们需要使用本地电脑远程访问和管理MySQL数据库。默认情况下，为了安全MySQL只允许本地登录，如果要开启远程连接，则需要修改MySQL的配置文件

```bash
$sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

打开配置文件，找到bind-address = 127.0.0.1这一行,改为bind-address = 0.0.0.0即可或简单一点注释掉也行

修改完成保存后，需要重启MySQL服务才会生效

