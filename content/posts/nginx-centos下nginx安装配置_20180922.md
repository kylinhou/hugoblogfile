---
title: nginx安装配置
categories: ["性能测试"]
tags: ["nginx", "centos"]
date: 2016-08-24
author: "kylin"
grammar_cjkRuby: true
---

# centos下nginx安装配置

### 安装安装编译工具及库文件

```
yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel
```

<!--more-->

### 安装PCRE

```
yum -y install pcre-devel
```

### 安装nginx

#### 下载

下载 Nginx，下载地址：[http://nginx.org/download/nginx-1.6.2.tar.gz](http://nginx.org/download/nginx-1.6.2.tar.gz) 如果能使用wget

```
wget http://nginx.org/download/nginx-1.6.2.tar.gz
```

解压安装包

```
tar zxvf nginx-1.6.2.tar.gz
```

进入安装包目录

```
cd nginx-1.6.2
```

编译安装

```
[root@bogon nginx-1.6.2]# ./configure --prefix=/usr/local/webserver/nginx --with-http_stub_status_module --with-http_ssl_module 
[root@bogon nginx-1.6.2]# make
[root@bogon nginx-1.6.2]# make install
```

可以手动指定pcre的位置和版本：--with-pcre=/usr/local/src/pcre-8.35

> ./configure --prefix=/usr/local/webserver/nginx --with-http_stub_status_module --with-http_ssl_module  --with-pcre=/usr/local/src/pcre-8.35

#### nginx配置

创建 Nginx 运行使用的用户 nginx

```
[root@bogon conf]# /usr/sbin/groupadd nginx 
[root@bogon conf]# /usr/sbin/useradd -g nginx nginx
```

配置nginx.conf，将/usr/local/webserver/nginx/conf/nginx.conf替换为以下内容

```
worker_processes  auto;

worker_rlimit_nofile 65535;

error_log  logs/error.log error;
pid        logs/nginx.pid;


events {
    use epoll;
    worker_connections  1024;
}


http {
    server_tokens off;
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for" $host '
			'$upstream_addr $upstream_status $request_time $upstream_response_time';
    access_log  logs/access.log  main;

    sendfile        on;
    tcp_nopush     on;

    keepalive_timeout  65;
    tcp_nodelay   on;
 
    open_file_cache max=20000 inactive=120s;
    open_file_cache_valid 30s;
    open_file_cache_min_uses 2;
    open_file_cache_errors on;

    include extra/*.conf;
}
```

然后在

/usr/local/webserver/nginx/conf/目录下新建一个extra文件夹，里面放一个extra.conf，内容如下

```
upstream myServer {
	#这里是tomcat实例的地址，有几个写几个
    server ip:端口号 max_fails=2 fail_timeout=10s;
    server ip:端口号 max_fails=2 fail_timeout=10s;

    keepalive 4;
}
server {
    listen       80 default;
    server_name  xxxx;
   # proxy_redirect http://xxxx http://xxx；
   # proxy_redirect http://xxxx http://xxx;
    charset 	utf-8;
    access_log  logs/test-push.log  main;

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /etc/nginx/html;
    }

    location / {
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504 http_404;
        proxy_pass   http://myServer;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }
}
```

注意上面 location中的proxy_pass   http://myServer;必须与upstream myServer中的myServer保持一致

关于server中的server_name，发现在只有一个server域的时候可以随便配，或者不配，这样任意的访问都会匹配到这个server。但是多个server域的时候nginx肯定会去匹配具体的请求。具体的理解可以看这里：

> 一个大型的网站通常会有很多下属的站点，有各自的服务器提供相应的服务，在 nginx 中我们可以通过一个叫虚拟主机的概念来将这些不同的服务配置隔离，这就是上面配置中的 server 的含义。举例来说 google 旗下有翻译和学术两款产品我们就可以在 nginx 的配置文件中配置两个 server，server*name 分别为 translate.google.com 和 scholar.google.com，这样的话不同的 url 请求就会对应到 nginx 相应的设置，转发到不同的后端服务器上。这里的 server*name 是和客户端 http 请求中的 host 行进行匹配的。 
>
> default_server 的含义是指如果有其他 http 请求的 host 在 nginx 中不存在设置的话那么就用这个 server 的配置来处理。比如我们去访问 127.0.0.1 那么也会落到这个 server 来处理。 

#### 启动关闭nginx

```
## 检查配置文件是否正确
# /usr/local/webserver/nginx/sbin/nginx -t 
# ./sbin/nginx -V     # 可以看到编译选项

## 启动、关闭
# /usr/local/webserver/nginx/sbin/nginx        # 默认配置文件 conf/nginx.conf，-c 指定
# ./sbin/nginx -s stop
或 pkill nginx

## 重启，不会改变启动时指定的配置文件
# ./sbin/nginx -s reload
或 kill -HUP `cat /usr/local/nginx-1.6/logs/nginx.pid`
```

想看更多的介绍，可以看看这个https://segmentfault.com/a/1190000002797601

