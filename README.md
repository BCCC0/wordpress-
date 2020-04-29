# wordpress-migrate

在两个linux主机下的wordpress数据库自动迁移

## 问题描述

如果你有两个wordpress站点，如何自动实现同步。

## 使用方法
1. 安装expect
```shell
apt install expect
```
2. 把sender文件夹放在源主机上，把receiver放在目的主机上
3. 在sender文件夹下配置sender脚本
```shell
#!/bin/bash

./backup mysql_password mysql_user > mysql.sql
sed -i '1,2d' mysql.sql

./scpconnect login_password login_user remote_ip  remote_port remote_path
```

需要改变的参数有以下七个：

    mysql_password：源主机上MySQL的用户

    mysql_user：源主机MySQL的密码（与wordpress后台一致）

    login_password：ssh登录密码

    login_user：ssh登录用户

    remote_ip：ssh地址

    remote_port：ssh端口

    remote_path：目的主机上存放目录，假设是/root，那么会生成一个/root/mysql.sql的数据库文件

4. 在receiver文件夹下配置receiver脚本
```shell
#!/bin/bash

./restore mysql_password mysql_user path_to_mysql.sql
./changedomain mysql_password mysql_user oldurl newurl

```

需要改变的参数有五个：

    mysql_password：目的主机上MySQL的用户

    mysql_user：目的主机MySQL的密码（与wordpress后台一致）

    path_to_mysql.sql：生成文件mysql.sql的位置，如果源主机上remote_path配置是/root，此配置则为/root/mysql.sql

    oldurl:源主机wordpress后台站点域名

    newurl:目的主机wordpress后台站点域名
    
5. 设置crontab实现自动更新

**设置sender**

在sender主机上

```shell
crontab -e
```
用文本编辑器增加:
```shell
00 00 * * * bash path_to_sender_script
```
path_to_sender_script: send脚本的路径

**设置receiver**

在receiver主机上：

```shell
crontab -e
```
用文本编辑器增加:
```shell
05 00 * * * bash path_to_sender_script
```
path_to_send_script: receiver脚本的路径

这样配置可以在每天0点0分发送，0点5分安装数据库备份

## 补充

此项目上传数据库通过scp命令，可以自行配置其他方式，或是配置rsa免密码登录。亦可以利用诸如Syncthing之类的文件夹同步方式，自动同步wordpress媒体文件。

默认的数据库名称为wordpress，默认的表格前缀为wp_，有需要请自行改变。

可以利用changedomain脚本方便的改变数据库的域名，具体使用方法见上文receiver配置文件。

密码以明文方式存储在脚本当中，可考虑加密脚本文件文件。
