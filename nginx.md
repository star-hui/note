https://www.bilibili.com/video/av22616373/?p=1
https://www.kancloud.cn/curder/nginx/96674

# 1. nginx的安装 与 信号量

## 1.1. nginx的安装
下载地址：nginx官网，下载stable版本
wget 下载到服务器上，再解压

下载地址: nginx的官 https://nginx.org/en/download.html 选择stable版本  

### 1.1.1. 编译安装三部曲
1. config 生产install文件  
其中会依赖gcc编译器  
当缺少 xxxx library 就是安装 yum install xxx-devel、  

``` python
./configure --prefix=/usr/local/nginx
```

     yum install -y zlib-devel  
     yum install -y pcre-devel
2. make 编译成成二进制文件  
    直接命令行下 make

3. make install  
    其实就是把生产的二进制文件 copy 到指定的文件夹  

安装完成：在安装目录下的sbin的文件下：./nginx  启动进程

启动错误：  
端口号被占用  
``` python
netstat -ant | grep 80
kill -9 进程号  # 杀掉占用的端口
```

## nginx进程模型

nginx 一启动就会有 master 与 worker进程。

``` python
[root@lh ~]# ps aux | grep nginx
root      17644  0.0  0.0  20544   608 ?        Ss   13:21   0:00 nginx: master process ./nginx  #  主进程
nobody    17645  0.0  0.0  20988  1576 ?        S    13:21   0:00 nginx: worker process          #  工作进程
```

在工作方式上，Nginx分为单工作进程和多工作进程两种模式。  
    在单工作进程模式下，除主进程外，还有一个工作进程，工作进程是单线程的；  
    在多工作进程模式下，每个工作进程包含多个线程。Nginx默认为单工作进程模式。

Nginx在启动后，会有一个master进程和多个worker进程。

1. master进程

主要用来管理worker进程，包含：接收来自外界的信号，向各worker进程发送信号，监控worker进程的运行状态，当worker进程退出后(异常情况下)，会自动重新启动新的worker进程。

master进程充当整个进程组与用户的交互接口，同时对进程进行监护。它不需要处理网络事件，不负责业务的执行，只会通过管理worker进程来实现重启服务、平滑升级、更换日志文件、配置文件实时生效等功能。

我们要控制nginx，只需要通过kill向master进程发送信号就行了。比如kill -HUP pid，则是告诉nginx，从容地重启nginx，我们一般用这个信号来重启nginx，或重新加载配置，因为是从容地重启，因此服务是不中断的。master进程在接收到HUP信号后是怎么做的呢？首先master进程在接到信号后，会先重新加载配置文件，然后再启动新的worker进程，并向所有老的worker进程发送信号，告诉他们可以光荣退休了。新的worker在启动后，就开始接收新的请求，而老的worker在收到来自master的信号后，就不再接收新的请求，并且在当前进程中的所有未处理完的请求处理完成后，再退出。当然，直接给master进程发送信号，这是比较老的操作方式，nginx在0.8版本之后，引入了一系列命令行参数，来方便我们管理。比如，./nginx -s reload，就是来重启nginx，./nginx -s stop，就是来停止nginx的运行。如何做到的呢？我们还是拿reload来说，我们看到，执行命令时，我们是启动一个新的nginx进程，而新的nginx进程在解析到reload参数后，就知道我们的目的是控制nginx来重新加载配置文件了，它会向master进程发送信号，然后接下来的动作，就和我们直接向master进程发送信号一样了。

2. worker进程：
而基本的网络事件，则是放在worker进程中来处理了。多个worker进程之间是对等的，他们同等竞争来自客户端的请求，各进程互相之间是独立的。一个请求，只可能在一个worker进程中处理，一个worker进程，不可能处理其它进程的请求。worker进程的个数是可以设置的，一般我们会设置与机器cpu核数一致，这里面的原因与nginx的进程模型以及事件处理模型是分不开的。
worker进程之间是平等的，每个进程，处理请求的机会也是一样的。当我们提供80端口的http服务时，一个连接请求过来，每个进程都有可能处理这个连接，怎么做到的呢？首先，每个worker进程都是从master进程fork过来，在master进程里面，先建立好需要listen的socket（listenfd）之后，然后再fork出多个worker进程。所有worker进程的listenfd会在新连接到来时变得可读，为保证只有一个进程处理该连接，所有worker进程在注册listenfd读事件前抢accept_mutex，抢到互斥锁的那个进程注册listenfd读事件，在读事件里调用accept接受该连接。当一个worker进程在accept这个连接之后，就开始读取请求，解析请求，处理请求，产生数据后，再返回给客户端，最后才断开连接，这样一个完整的请求就是这样的了。我们可以看到，一个请求，完全由worker进程来处理，而且只在一个worker进程中处理。worker进程之间是平等的，每个进程，处理请求的机会也是一样的。当我们提供80端口的http服务时，一个连接请求过来，每个进程都有可能处理这个连接，怎么做到的呢？首先，每个worker进程都是从master进程fork过来，在master进程里面，先建立好需要listen的socket（listenfd）之后，然后再fork出多个worker进程。所有worker进程的listenfd会在新连接到来时变得可读，为保证只有一个进程处理该连接，所有worker进程在注册listenfd读事件前抢accept_mutex，抢到互斥锁的那个进程注册listenfd读事件，在读事件里调用accept接受该连接。当一个worker进程在accept这个连接之后，就开始读取请求，解析请求，处理请求，产生数据后，再返回给客户端，最后才断开连接，这样一个完整的请求就是这样的了。我们可以看到，一个请求，完全由worker进程来处理，而且只在一个worker进程中处理。

## 信号控制信号量

TERM, INT   fast shutdown  
QUIT        graceful shutdown  
HUP         changing configuration, keeping up with a changed time zone (only for FreeBSD and Linux), starting new worker processes with a new configuration, graceful shutdown of old worker processes
USR1        re-opening log files
USR2        upgrading an executable file
WINCH       graceful shutdown of worker processes

# 2. 配置文件介绍

``` python
"""
全局区:
"""
#user  nobody;  # 启动的帐户
worker_processes  1;  # 有一个工作的子进程，可以自行修改，一般设置为cpu*核数

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

'''
events区域：一般配置nginx进程与连接的特需病房
'''
events {
    worker_connections  1024;  # 一个work进程允许最大的连接数
}

'''
http区域：我们主要的配置区域
'''
http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
    server {
        listen 80;
        server_name z.cn;
        location / {
            root    zcom;
            index   hui.index;
        }
    }
    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index ab.html index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```

## 2.1. 三种虚拟主机

对于网工的同学，可能对ip与域名的两种方式不能理解，nginx是解析http请求包里面的数据地址。 而不是看dns转换后的ip。

### 2.1.1. 域名的虚拟主机

监听域名，根据域名反射到不同的路径资源，最少配置三项:

1. 监听端口
2. 域名
3. 资源的路径与文件名

``` python
server{
    listen 80;  # 监听端口
    server_name z.com;  # 监听的域名
    localcation / {  # 请求过来，返回指定的  路径 与 资源
        root    z.com;  # 指定这个虚拟主机的工作目录，可以使用绝对路径 与 相对路径。 这里使用的是相对路径(相对于nginx的主目录)
        index   index.html;  # 当没有带后缀名的时候，默认访问这个页面
    }
}
```

## 2.2. 端口虚拟主机

``` python
 42     server {
 43         listen          2222;
 44         server_name     z.cn;
 45         location / {
 46             root        /var/www;  # 这里使用的是绝对路径
 47             index       login.html;
 48         }
 49     }
```

## 2.3. ip的虚拟主机

``` python
 42     server {
 43         listen          2222;
 44         server_name     192.168.9.88;
 45         location / {
 46             root        /var/www;  # 这里使用的是绝对路径
 47             index       login.html;
 48         }
 49     }
```

# 3. 日志管理

## 3.1. 日志的配置

nginx可以对不同的server做不同的日志管理。  
默认在启动的时候,使用了系统自定义的main格式，存放在logs/access.log下。  
我们可以为每个server存放不同的日志  
> access_log logs/access.log main;  
> 参数一：配置名  
> 参数二：指定访问日志的文件是logs/access.log  
> 参数三：使用的日志格式"main"格式。同时我们也可以通过log_format配置其他格式的日志记录方式。  

记得把全局的log_format打开。 这个main是日志格式的别名，我们也可以自己定义
``` python
 42     server {
 43         listen          2222;
 44         server_name     192.168.9.88;
 45         location / {
 46             root        /var/www;  # 这里使用的是绝对路径
 47             index       login.html;
 48         }
            access_log logs/2222.access.log main; # 加上自己server的日志
 49     }
```

## 3.2. 日志的切割

日志如果不切割，量会非常大。 按照每天进行分割，这样文件就会很小。也便于管理。

所需要的技术shell脚本与定时任务

``` python
#!/bin/bash

DATE=$(date -d yesterday +%Y%m%d)
LOG_PATH=/usr/local/nginx/logs/
LOG_NAME=access.log
BASE_PATH=/var/log/
SAVE_LOG_NAME=${DATE}.${LOG_NAME}

mv ${LOG_PATH}${LOG_NAME} ${BASE_PATH}${SAVE_LOG_NAME}

kill -USR1 $(cat /usr/local/nginx/logs/nginx.pid)
```