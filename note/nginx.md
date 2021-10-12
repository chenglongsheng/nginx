[TOC]

# 一、nginx介绍



## 1、nginx概述

​		Nginx ("engine x") 是一个高性能的 HTTP 和反向代理服务器,特点是占有内存少，并发能力强，事实上 nginx 的并发能力确实在同类型的网页服务器中表现较好，中国大陆使用 nginx网站用户有：百度、京东、新浪、网易、腾讯、淘宝等。



## 2、Nginx 作为 web 服务器

[Nginx 可以作为静态页面的 web 服务器，同时还支持 CGI 协议的动态语言，比如 perl、php等。但是不支持 java。Java 程序只能通过与 tomcat 配合完成。Nginx 专为性能优化而开发，性能是其最重要的考量,实现上非常注重效率 ，能经受高负载的考验,有报告表明能支持高达 50,000 个并发连接数。](https://lnmp.org/nginx.html)



## 3、正向代理

nginx 不仅可以做反向代理，实现负载均衡。还能用作正向代理来进行上网等功能。

正向代理：如果把局域网外的 Internet 想象成一个巨大的资源库，则局域网中的客户端要访问 Internet，则需要通过代理服务器来访问，这种代理服务就称为正向代理。



## 4、反向代理

反向代理，其实客户端对代理是无感知的，因为客户端不需要任何配置就可以访问，我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，在返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器 IP 地址。



## 5、负载均衡

​		客户端发送多个请求到服务器，服务器处理请求，有一些可能要与数据库进行交互，服务器处理完毕后，再将结果返回给客户端。

​		这种架构模式对于早期的系统相对单一，并发请求相对较少的情况下是比较适合的，成本也低。但是随着信息数量的不断增长，访问量和数据量的飞速增长，以及系统业务的复杂度增加，这种架构会造成服务器相应客户端的请求日益缓慢，并发量特别大的时候，还容易造成服务器直接崩溃。很明显这是由于服务器性能的瓶颈造成的问题，那么如何解决这种情况呢？

​		我们首先想到的可能是升级服务器的配置，比如提高 CPU 执行频率，加大内存等提高机器的物理性能来解决此问题，但是我们知道[摩尔定律](https://www.cnblogs.com/ysocean/p/7641540.html)的日益失效，硬件的性能提升已经不能满足日益提升的需求了。最明显的一个例子，天猫双十一当天，某个热销商品的瞬时访问量是极其庞大的，那么类似上面的系统架构，将机器都增加到现有的顶级物理配置，都是不能够满足需求的。那么怎么办呢？

​		上面的分析我们去掉了增加服务器物理配置来解决问题的办法，也就是说纵向解决问题的办法行不通了，那么横向增加服务器的数量呢？这时候**集群**的概念产生了，*单个服务器解决不了，我们增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的情况改为将请求分发到多个服务器上，将负载分发到不同的服务器*，也就是我们所说的**负载均衡**。



## 6、动静分离

为了加快网站的解析速度，可以把动态页面和静态页面由不同的服务器来解析，加快解析速度。降低原来单个服务器的压力。



# 二、安装nginx



1.官网下载nginx

http://nginx.org/

**需要的依赖*pcre*、*openssl*、*zlib*、*nginx***

2.一键安装

```shell
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
```

3.安装nginx

解压nginx-xx-.tar.gz

```shell
tar -xvf nginx-xx-.tar.gz
```

进入解压缩目录，执行

```shell
./configure
```

解析安装

```shell
make && make install
```



***防火墙开放端口号***

查看开放端口号

```shell
firewall-cmd --list-all
```

设置开放端口号（80端口）

```shell
firewall-cmd --add-service=http -permanent
```

```shell
firewall-cmd --add-port=80/tcp --permanent
```

重启防火墙

```shell
firewall-cmd -reload
```



# 三、nginx常用命令



需在目录中使用

```shell
cd /usr/local/nginx/sbin/
```

启动

```shell
./nginx
```

关闭

```shell
./nginx -s stop
```

重载

```shell
./nginx -s reload
```

查看nginx进程

```shell
ps -ef | grep nginx
```

查看版本号

```shell
./nginx -v
```



# 四、nginx配置文件



## 1、位置

​		nginx 安装目录下，其默认的配置文件都放在这个目录的 conf 目录下，而主配置文件nginx.conf 也在其中，后续对 nginx 的使用基本上都是对此配置文件进行相应的修改。

```shell
cd /usr/local/nginx/conf/
```



## 1、组成

1. 全局

   ​		影响nginx服务器整体运行的配置指令，主要包括配置运行nginx服务器的用户（组）、允许生成的worker process数，进程PID存放路径、日志存放路径和类型以及配置文件的引入等。

   ​		比如：

   ```co
   worker_process 1;
   ```

   ​		这是 Nginx 服务器并发处理服务的关键配置，worker_processes 值越大，可以支持的并发处理量也越多，但是会受到硬件、软件等设备的制约.

2. events

   ​		events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process 下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 word process 可以同时支持的最大连接数等。

   ```
   events {
       worker_connections 1024;
   }
   ```

   ​		上述例子就表示每个 work process 支持的最大连接数为 1024。

   ​		这部分的配置对 Nginx 的性能影响较大，在实际中应该灵活配置。

3. http

   这算是 Nginx 服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里。

   需要注意的是：http 块也可以包括 **http 全局块**、**server 块**。

   **①、http 全局块**

   http 全局块配置的指令包括文件引入、MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。

   **②、server 块**

   这块和虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了节省互联网服务器硬件成本。

   

   每个 http 块可以包括多个 server 块，而每个 server 块就相当于一个虚拟主机。

   而每个 server 块也分为全局 server 块，以及可以同时包含多个 location 块。

   **1、全局 server 块**

   最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或 IP 配置。

   **2、location 块**

   一个 server 块可以配置多个 location 块。

   这块的主要作用是基于Nginx服务器接收到的请求字符串（例如 server_name/url-string），对虚拟主机名称（也可以是 IP 别名）之外的字符串（例如 前面的 /url-string）进行匹配，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里进行。



# 五、配置实例



## 1、反向代理

### 1、实例一

**1、实现**：浏览器访问www.123.com，得到的页面是Linux上tomcat服务器的8080端口



**2、Linux上安装tomcat**

```shell
cd /usr/src
```

把文件通过工具传到Linux上，解压（XXX为版本号）

```shell
tar -xvf apache-tomcat-XXX
```

启动tomcat

```shell
cd apache-tomcat-XXX/bin
```

```shell
./start.sh
```

查看日志

```shell
cd ..
```

```shell
cd logs/
```

```shell
tail -f catalina.out
```

开放防火墙端口号

```shell
firewall-cmd --add-port=8080/tcp --permanent
```

```shell
firewall-cmd --reload
```

查看已经开放端口号

```shell
firewall-cmd --list-all
```

本地浏览器访问虚拟机IP:8080



**3、具体配置**

（1）在windows上修改hosts文件（目录C:\Windows\System32\drivers\etc）

```
# Copyright (c) 1993-2009 Microsoft Corp.
#
# This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
#
# This file contains the mappings of IP addresses to host names. Each
# entry should be kept on an individual line. The IP address should
# be placed in the first column followed by the corresponding host name.
# The IP address and the host name should be separated by at least one
# space.
#
# Additionally, comments (such as these) may be inserted on individual
# lines or following the machine name denoted by a '#' symbol.
#
# For example:
#
#      102.54.94.97     rhino.acme.com          # source server
#       38.25.63.10     x.acme.com              # x client host

# localhost name resolution is handled within DNS itself.
#	127.0.0.1       localhost
#	::1             localhost
192.168.40.128 www.123.com
```

**加上`192.168.40.128 www.123.com`**

即使用www.123.com域名访问IP192.168.40.128



（2）修改nginx配置

```shell
cd /usr/local/nginx/conf
```

```shell
vi nginx.conf
```

修改server_name为`server_name  192.168.40.128;`

location中加上代理路径`proxy_pass http://127.0.0.1:8080;`

**测试**windows浏览器访问www.123.com:80

```
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


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
        listen       80;
        server_name  192.168.40.128;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            proxy_pass http://127.0.0.1:8080;
            index  index.html index.htm;
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



### 2、实列二

**1.实现**

访问http://192.168.40.128:9001/edu/ 直接跳转127.0.0.1:8080

访问http://192.168.40.128:9001/vod/ 直接跳转127.0.0.1:8081



**2.准备两个tomcat服务，8080和8081**

在/usr/src/中新建两个文件夹tomcat8080和tomcat8081

```shell
mkdir tomcat8080 tomcat8081
```

把apache-tomcat-xxx.tar.gz分别复制一份在两个文件夹中，并解压

```shell
tar -xvf apache-tomcat-xxx.tar.gz
```

**关闭之前启动的tomcat**

启动两个服务器

```shell
cd /usr/src/tomcat8080/apache-tomcat-xxx/bin
./start.sh
```

```
cd /usr/src/tomcat8081/apache-tomcat-xxx/bin
./start.sh
```

在浏览器中分别测试两个服务

虚拟机IP:8080 

虚拟机IP:8081



**3.增加测试文件**

使用远程管理工具在/usr/src/tomcat8080/apache-tomcat-xxx/webapps中新建文件夹edu，并在里面新建文件edu.html，以及编辑内容`<h1>edu!!!</h1>`

使用远程管理工具在/usr/src/tomcat8081/apache-tomcat-xxx/webapps中新建文件夹vod，并在里面新建文件vod.html，以及编辑内容`<h1>vod!!!</h1>`

在浏览器中测试（192.168.40.128为虚拟机IP）

http://192.168.40.128:8080/edu/edu.html

http://192.168.40.128:8081/vod/vod.html



**4.配置nginx.conf**

加入以下配置

    server {
            listen       9001;
            server_name  192.168.40.128;
    	location ~ /edu/ {
            proxy_pass http://127.0.0.1:8080;
        }
        location ~ /vod/ {
            proxy_pass http://127.0.0.1:8081;
        }
    }

在这个服务中监听9001端口，服务名称是192.168.40.128

 ~ /edu/是正则表达，出现edu就访问http://127.0.0.1:8080;

**开发对外访问端口号8081，9001**

重启nginx

测试效果

![](https://note-1010.oss-cn-beijing.aliyuncs.com/img/屏幕截图 2021-10-10 213942.png)

![](https://note-1010.oss-cn-beijing.aliyuncs.com/img/屏幕截图 2021-10-10 214014.png)

**location 指令说明**

1、=：用于不含正则表达式的uri前，要求请求字符串与uri严格匹配，如果匹配成功，就停止继续搜索并立刻处理该请求。

2、~：用于表示uri包含正则表达式，并区分大小写。

3、~*：用于表达uri包含正则表达式，不区分大小写。

4、^~：用于不含正则表达式的uri前，要求nginx服务器找到标识uri和请求字符串匹配度是最高的location后，立刻使用此location处理请求，而不再使用location块中的正则uri和请求字符串做匹配。

**注意：如果 uri 包含正则表达式，则必须要有 ~ 或者 ~* 标识。**



## 2、负载均衡



**1、分配策略**

（1）轮询（默认）

​		每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器宕机，能够自动剔除。



（2）weight

​		weight代表权，默认1，权重越高分配的客户端越多。

​		指定轮询几率，weight和访问比例成正比，用于后端服务器性能不均的情况，例如：

```
upstream server_pool { 
	server 192.168.5.21 weight=10; 
	server 192.168.5.22 weight=10; 
}
```



（3）ip_hash

​		每个请求按ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以session的问题。例如：

```
upstream server_pool {
	ip_hash;
	server 192.168.40.128:80;
	server 192.168.40.129:80;
}
```



（4）fair（第三方）

按后端服务器的响应时间来分配请求，响应时间短的优先分配。

```
upstream server_pool { 
	server 192.168.5.21:80; 
	server 192.168.5.22:80; 
	fair; 
}
```



**2、实现效果**

（1）在浏览器地址栏中输入：http://192.168.40.128/edu/a.html，负载均衡效果，平均8080和8081端口中。



**3、准备工作**

（1）准备两台tomcat服务器，一台8080一台8081

（2）在两台tomcat里面webapps目录中创建edu文件夹，并在里面创建a.html，分别标识8080和8081，用于测试。

（3）在nginx配置文件中加入以下配置：

![](https://note-1010.oss-cn-beijing.aliyuncs.com/img/屏幕截图 2021-10-11 115611.png)

在http块中增加

```
upstream myserver {
	server	192.168.40.128:8080;
	server	192.168.40.128:8081;
}
```

修改server_name的值为192.168.40.128（虚拟机ip）

在location中新增代理路径

```
proxy_pass http://myserver;
```

![](https://note-1010.oss-cn-beijing.aliyuncs.com/img/屏幕截图 2021-10-11 125634.png)



**4、启动**

（1）启动两台tomcat

（2）启动nginx

**5、测试**

打开http://192.168.40.128/edu/a.html

每次刷新显示8080或者8081



## 3、动静分离

**1、介绍**

​		nginx 动静分离简单来说就是把动态跟静态请求分开，不能理解成只是单纯的把动态页面和静态页面物理分离。严格意义上说应该是动态请求跟静态请求分开，可以理解成使用 Nginx 处理静态页面，Tomcat 处理动态页面。动静分离从目前实现角度来讲大致分为两种，一种是纯粹把静态文件独立成单独的域名，放在独立的服务器上，也是目前主流推崇的方案；另外一种方法就是动态跟静态文件混合在一起发布，通过 nginx 来分开。

​		**通过 location 指定不同的后缀名实现不同的请求转发。通过 expires 参数设置，可以使浏览器缓存过期时间，减少与服务器之前的请求和流量。具体 Expires 定义：是给一个资源设定一个过期时间，也就是说无需去服务端验证，直接通过浏览器自身确认是否过期即可，所以不会产生额外的流量。此种方法非常适合不经常变动的资源。（如果经常更新的文件，不建议使用 Expires 来缓存），我这里设置 3d，表示在这 3 天之内访问这个 URL，发送一个请求，比对服务器该文件最后更新时间没有变化，则不会从服务器抓取，返回状态码304，如果有修改，则直接从服务器重新下载，返回状态码 200。**

![动静分离](https://note-1010.oss-cn-beijing.aliyuncs.com/img/屏幕截图 2021-10-11 162533.png)



**2、准备**

准备静态资源

在根目录下新建data文件夹，并在里面新建www和image两个文件夹

在里面放点测试文件



**3、配置文件**

在server中创建两个location

```
location /www/ {
	root /data/;
	index index.html index.html
}
```

```
location /image/ {
	root /data/;
	autoindex on;
}
```

![](https://note-1010.oss-cn-beijing.aliyuncs.com/img/屏幕截图 2021-10-11 164647.png)



**4、测试**

浏览器中打开

http://192.168.40.128/www/a.html

http://192.168.40.128/image/a.bmp



## 4、高可用集群

**1、什么是高可用**

![](C:/Users/CLS/Pictures/note/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202021-10-12%20142108.png)



-  需要两台nginx服务器
- 需要keepalived
- 需要虚拟ip



**2、配置**

准备两台虚拟机

在两台虚拟机中安装nginx和keepalived

```shell
yum install keepalived -y
```

记得关闭防火墙

修改keepalived配置

```
cd /etc/keepalived
```

在配置文件中全部替换以下内容 **注释部分重点修改**

```
global_defs { 
	notification_email { 
         acassen@firewall.loc 
         failover@firewall.loc 
         sysadmin@firewall.loc 
     } 
     notification_email_from Alexandre.Cassen@firewall.loc 
     smtp_server 192.168.40.132 
     smtp_connect_timeout 30 
     router_id LVS_DEVEL 
} 
vrrp_script chk_http_port { 
     script "/usr/local/src/nginx_check.sh" 
     interval 2 #（检测脚本执行的间隔） 
     weight 2 
} 
vrrp_instance VI_1 { 
     state BACKUP # 备份服务器上将 MASTER 改为 BACKUP 
     interface ens33 //网卡 
     virtual_router_id 51 # 主、备机的 virtual_router_id 必须相同 
     priority 100 # 主、备机取不同的优先级，主机值较大，备份机值较小 
     advert_int 1 
     authentication { 
     auth_type PASS 
     auth_pass 1111 
     } 
     virtual_ipaddress { 
     192.168.40.50 // VRRP H 虚拟地址 
	}
}
```

在/usr/local/src中放入nginx_check.sh脚本文件，检测nginx是否还活着

其内容为

```shell
#!/bin/bash 
A=`ps -C nginx –no-header |wc -l` 
if [ $A -eq 0 ];then 
 /usr/local/nginx/sbin/nginx 
 sleep 2 
 if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then 
 killall keepalived 
 #当主服务器宕机，杀死所有主服务器的nginx进程，启用从服务器
 fi 
fi
```



**3、测试**

启动 keepalived 和 nginx

```shell
systemctl start keepalived.service
```

![](https://note-1010.oss-cn-beijing.aliyuncs.com/img/屏幕截图 2021-10-12 193933.png)

服务器绑定到虚拟ip

（1）访问192.168.40.50

（2）停止主服务器再访问192.168.40.50，从服务器会绑定到虚拟ip



# 五、nginx原理



## 1、worker和master

![](https://note-1010.oss-cn-beijing.aliyuncs.com/img/屏幕截图 2021-10-12 195228.png)



## 2、工作原理

**争抢**

![](https://note-1010.oss-cn-beijing.aliyuncs.com/img/屏幕截图 2021-10-12 195248.png)



## 3、一个master和多个worker

（1）可以热部署`nginx -s reload`

（2）每个worker都是独立的进程，不造成服务中断

（3）worker数=cpu数

![](https://note-1010.oss-cn-beijing.aliyuncs.com/img/屏幕截图 2021-10-12 195034.png)

**好处**

​		首先，对于每个worker进程来说，独立的进程，不需要加锁，所以省略了锁带来的开销，同时在编程以及问题查找时，也会方便很多。其次，采用独立的进程可以让相互之间不会影响，一个进程退出后，其它进程还在工作，服务不会中断，master进程则很快启动新的worker进程。当然，worker进程的异常退出，肯定是程序有bug了，异常退出，会导致当前worker上的所有请求失败，不过不会影响到所有请求，所以降低了风险。

**设置worker数量**

​		nginx同redis类似都采用了io多路复用机制，每个worker都是一个独立的进程，但每个进程里只有一个主线程，通过异步非阻塞的方式来处理请求，即使是成千上万个请求也不在话下。每个worker的线程可以把一个cpu的性能发挥到极致。所以worker数和服务器的cpu数相等是最为合适的。少了浪费cpu，多了损耗cpu.





*发送请求，占用worker多少连接数？*

答：2或4。只调用静态资源是两个，加上调用tomcat连接数据库操作数据，4个



*nginx有一个master，有四个worker，每个worker支持最大的连接数据1024，支持的**最大并发数**是多少？*

答：静态访问是**worker_connecttion * worker_processes / 2**，

​		HTTP作为反向代理是**worker_connecttion * worker_processes / 4**

