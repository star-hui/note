https://www.bilibili.com/video/av22616373/?p=1
https://www.kancloud.cn/curder/nginx/96674

# nginx的安装 与 信号量

## nginx的安装

下载地址: nginx的官 https://nginx.org/en/download.html 选择stable版本
# 编译安装




## 配置文件介绍

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
http区域：我么主要的配置区域
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

## 三种虚拟主机

对于网工的同学，可能对ip与域名的两种方式不能理解，nginx是解析http请求包里面的数据地址。 而不是看dns转换后的ip。

### 域名的虚拟主机

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

## 端口虚拟主机

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

## ip的虚拟主机

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

## 日志管理

### 日志的配置

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

### 日志的切割
日志如果不切割，量会非常大。 按照每天进行分割，这样文件就会很小。也便于管理。
