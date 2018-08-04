# 1. zabix简介

## 1.1. 监控对象

1. 硬件与网络

网络：流量，端口等  通过        snmp
硬件：硬件cpu，磁盘，温顿       IPMI

2. 操作系统

集群、虚拟化、操作系统          使用agent来监控

3. 应用层

数据库：mysql，oracle等        使用agent来监控
应用软件：Nginx，Apacch

## 1.2. 监控模式
被动模式: 服务端主动去轮询
主动模式: agent主动打开Tcp连接
第三者模式：中心zabbix，管理多台zabbix。

# 2. zabix安装

在安装之前，一定要记得先关闭防火墙 和 suselinux。
默认的网页登入帐号密码: Admin,zabbix
http://ip/zabbix

## 2.1. 安装zabbix

``` python
1. 安装yum源  
rpm -ivh http://repo.zabbix.com/zabbix/3.2/rhel/7/x86_64/zabbix-release-3.2-1.el7.noarch.rpm
2. 安装相关的服务
yum install zabbix-server-mysql zabbix-web-mysql -y
yum install zabbix-agent  # 客户端可选
```

## 2.2. 安装数据库并授权

``` python
[root@lh ~]# yum install -y mariadb mariadb-server
[root@lh ~]# systemctl enable mariadb
[root@lh ~]# systemctl start mariadb
shell> mysql
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> grant all privileges on zabbix.* to zabbix@localhost identified by '<password>';
mysql>  flush privileges;
mysql> quit;

"生成用户"
[root@lh ~]# cd /usr/share/doc/zabbix-server-mysql-3.2.11/
[root@lh zabbix-server-mysql-3.2.11]# zcat create.sql.gz | mysql -uroot zabbix

"修改zabbixserver配置文件的数据库连接密码"
[root@lh zabbix-server-mysql-3.2.11]# vim /etc/zabbix/zabbix_server.conf
### Option: DBPassword
#       Database password. Ignored for SQLite.
#       Comment this line if no password is used.
#
# Mandatory: no
# Default:
DBPassword=hui

"修改httpd的时区"
 vim /etc/httpd/conf.d/zabbix.conf
     <IfModule mod_php5.c>
        php_value max_execution_time 300
        php_value memory_limit 128M
        php_value post_max_size 16M
        php_value upload_max_filesize 2M
        php_value max_input_time 300
        php_value always_populate_raw_post_data -1
        php_value date.timezone Asia/Shanghai  # 修改这行
    </IfModule>
```
完成之后就直接网页安装就ok了。

## 2.3. 客户端的zabbix安装

先决条件” 也需要zabbix的yum源.  
记得防火墙等需要关闭。
``` python

"1. 安装"
[root@lh ~]# rpm -ivh http://repo.zabbix.com/zabbix/3.2/rhel/7/x86_64/zabbix-release-3.2-1.el7.noarch.rpm
yum install zabbix-agent

"2. 修改配置文件"
vim /etc/zabbix/zabbix_agentd.conf

Server=192.168.0.223
ServerActive=192.168.0.223
Hostname=wb123

"3. 重启"
[root@lh ~]# systemctl restart zabbix-agent
[root@lh ~]# ss -tnlp | grep 10050
```

## 简单的测试

``` python
# 获取客户端的系统名字
zabbix_get -s 192.168.0.88 -k system.uname
# 获取客户端的负载
zabbix_get -s 192.168.0.88 -k system.cpu.load[all,avg15]
# 获取流量
zabbix_get -s 192.168.0.88 -k net.if.total[ens33]

```
# Item

自定义
``` python
more /etc/zabbix/zabbix_agentd.d/userparameter_mysql.conf 
Userparameter=mysql.reestions,mysqladmin -uroot status | cut -f4 -d':' | cut -f1 -d'S'

使用cut 进行分割提取。
```
