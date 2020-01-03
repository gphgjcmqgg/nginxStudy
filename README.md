# nginxStudy

## nginx基本概念

1. nginx是什么，做什么事情
  Nginx (engine x) 是一个高性能的HTTP和反向代理web服务器，其特点是占有内存少，并发能力强，事实上nginx的并发能力在同类型的网页服务器中表现较好，能够支持高达 50,000 个并发连接数的响应。
2. 反向代理
3. 负载均衡
4. 动静分离

## nginx安装、常用命令和配置文件

### nginx安装

1. 安装pcre依赖
把安装压缩文件放到linux系统中
解压压缩文件
进入解压之后目录，执行./configure
使用make && make install
安装之后，使用命令，查看版本号 pcre-config --version
2. 安装其他的依赖
yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel
3. 安装nginx
解压缩nginx-xx.tar.gz包
进入解压缩目录，执行./configure
make && make install
安装成功之后，在usr多出来一个文件夹local/nginx,在nginx有sbin有启动脚本
4. 查看开放的端口号
firewall-cmd --list-all
5. 设置开放的端口号
firewall-cmd --add-service=http-permanent
sudo firewall-cmd --add-port=80/tcp --permanent
6. 重启防火墙
firewall-cmd-reload

### nginx操作的常用命令

使用nginx操作命令前提条件：必须进入nginx的目录
/usr/local/nginx/sbin

1. 查看nginx的版本号
./nginx -v

2. 启动nginx
./nginx

3. 关闭nginx
./nginx -s stop

4. 重新加载nginx
./nginx -s reload

### nginx配置文件

1. nginx的配置文件的位置:
在/usr/local/nginx/conf  --> nginx.conf

### nginx的配置文件由三部分组成

1. 第一部分 全局块
从配置文件开始到events块之间的内容，主要会设置一些影响nginx服务器整体运行的配置指令
比如：worker_processes 值越大，可以支持的并发处理量也越多

2. 第二部分 events块
主要影响nginx服务器与用户的网络连接
比如： worker_connections  1024   支持最大连接数

3. 第三部分 http块
nginx服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里。
需要注意的是：http块也可以包括http全局块、server块。

## nginx配置实例 - 反向代理

1. 正向代理：在客户端（浏览器）配置代理服务器，通过代理服务器进行互联网访问
2. 反向代理：我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，再返回客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真是服务器IP地址。

### 反向代理具体实现

1. 实现效果

（1） 打开浏览器，在浏览器地址栏输入地址www.nihao.com,跳转到linux系统tomcat主页面中
2. 准备工作
（1）在linux系统安装tomcat，使用默认端口8080
 (2) 对外开放访问的端口8080
3. windows的host文件进行配置，配置域名映射的IP地址 www.nihao.com --> nginx地址

### 配置反向代理案例1

1. 找到C:\Windows\System32\drivers\etc 下的host文件，配置域名和ip地址映射
2. 在nginx进行请求转发的配置（反向代理）,找到/usr/local/nginx/conf下的nginx.conf

listen       80;
server_name  192.168.17.129;  //nginx的IP地址

location / {
    root   html;
    proxy_pass  http://127.0.0.1:8080;    // tomcat地址    返回nginx地址后转发到tomcat去
    index  index.html index.htm;
}
3. ./nginx  启动nginx
4. 输入网址访问

### 配置反向代理案例2

实现效果：
使用nginx反向代理，根据访问的路径跳转到不同端口的服务中
nginx监听端口为9001
访问http://192.168.17.129/edu/      直接跳转127.0.0.1:8080
访问http://192.168.17.129/vod/      直接跳转127.0.0.1:8081

1. 准备两个tomcat服务器，一个8080端口，一个8081端口
2. 找到C:\Windows\System32\drivers\etc 下的host文件，配置域名和ip地址映射
3. 在nginx进行请求转发的配置（反向代理）,找到/usr/local/nginx/conf下的nginx.conf

listen       9001;
server_name  192.168.17.129;  //nginx的IP地址

location ~ /edu/ {
    proxy_pass  http://127.0.0.1:8080;    // tomcat地址    返回nginx地址后转发到tomcat去8080
}
location ~ /vod/ {
    proxy_pass  http://127.0.0.1:8081;    // tomcat地址    返回nginx地址后转发到tomcat去8081
}
4. ./nginx  启动nginx
5. 输入网址访问

## nginx配置实例 - 负载均衡

单个服务器解决不了，我们增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的情况改为将请求分发到多个服务器上，将负载分发到不同的服务器，也就是我们所说的负载均衡。

### nginx配置实例 - 负载均衡实例1

1. 实现效果

（1）浏览器地址栏输入地址 http://192.168.17.129/edu/a.html，负载均衡效果，平均8080和8081端口中
2. 准备工作

（1） 准备两个tomcat服务器，一个8080端口，一个8081端口
（2） 在两台tomcat里面webapps目录中，创建名称是edu文件夹，在edu文件夹中创建页面a.html，用于测试。
3. 在nginx配置文件配置负载均衡
http {
    upstream myserver {
        server  192.168.17.129:8080;
        server  192.168.17.129:8081;
    }
    server {
        listen       80;
        server_name  192.168.17.129;

        location / {
            proxy_pass  http://myserver;
        }
    }
}
4. ./nginx  启动nginx
5. 输入网址访问

### nginx负载均衡 分配服务器策略

1. 轮询（默认）
2. weight策略：weight代表权重，默认是1，权重越高被分配的客户端越多
upstream myserver {
    server  192.168.17.129:8080 weight=5;
    server  192.168.17.129:8081 weight=10;
}
3. ip_hash：每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题
4. fair：按后端服务器的响应时间来分配请求，响应时间短的优先分配。

## nginx配置实例 - 动静分离

为了加快网站的解析速度，可以把动态页面和静态页面由不同的服务器来解析，加快解析速度。降低原来单个服务器的压力。

server {
    listen       80;
    server_name  192.168.17.129;

    location /www/ {
        root    /data/;
        index  index.html index.htm;
    }

    location /image/ {
        root    /data/;
        autoindex   on; // 列出文件夹中的内容
    }
}

## nginx配置高可用集群

准备工作

1. 需要两台nginx服务器
2. 需要keepalived       使用yum命令  yum install keepalived -y
安装之后，在etc里面生成目录keeplived，有文件keepalived.conf
3. 需要虚拟ip

## nginx原理
