[TOC]



# Nginx

## Nginx 能做什么

- 反向代理（核心指令 proxy_pass）
- 负载均衡（核心指令 upstream）
- HTTP 服务器
- 正向代理

## *架构

参考：http://www.aosabook.org/en/nginx.html

## location 匹配顺序

### 语法规则

> location [=|~|~*|^~] /uri/ { … }

| 模式                | 含义                                                         | 优先级         |
| ------------------- | :----------------------------------------------------------- | -------------- |
| **有修饰符**        |                                                              | 高于无修饰符   |
| location = /uri     | 精准匹配，只有 URI 完全相同才会执行 location 中的操作        | 最高           |
| location ^~ /uri    | 前缀匹配                                                     | 仅次于精准匹配 |
| location ~ pattern  | 区分大小写的正则匹配                                         | 仅次于前缀匹配 |
| location ~* pattern | 不区分大小写的正则匹配                                       | 仅次于前缀匹配 |
|                     | 两种正则匹配的优先级相同，按最先匹配原则，优先匹配位置最先、顺序最前的正则匹配 |                |
| **无修饰符**        |                                                              | 低于有修饰符   |
| location /uri       | 前缀匹配（按最长匹配原则，优先匹配 URI 最长的前缀匹配）      | 在正则匹配之后 |
| location /          | 通用匹配（最典型的前缀匹配），任何未匹配到其它 location 的请求都会匹配到，相当于switch 语句中的 default<br />最容易混淆的是 `location = /` 和 `location /` ，肯定是 `location = /` 精准匹配最先匹配 | 最低           |



前缀匹配时，Nginx 不对 url 做编码，因此请求为 `/static/20%/aa`，可以被规则 `^~ /static/ /aa` 匹配到（注意中间是空格）

多个 location 配置的情况下匹配顺序为（参考资料而来，还未实际验证，试试就知道了，不必拘泥，仅供参考）:

- 首先精确匹配 `=`
- 其次前缀匹配 `^~`
- 其次是按配置文件中顺序的正则匹配
- 然后匹配不带任何修饰的前缀匹配
- 最后是交给 `/` 通用匹配
- 当有匹配成功时候，停止匹配，按当前匹配规则处理请求

注意：前缀匹配时，如果有包含关系时，按最大匹配原则进行匹配。比如在前缀匹配：location /dir01 与 location /dir01/dir02，如有请求 http://localhost/dir01/dir02/file 将最终匹配到 location /dir01/dir02

### 大神实例：

例子，有如下匹配规则：

```nginx
location = / {
   echo "规则A";
}
location = /login {
   echo "规则B";
}
location ^~ /static/ {
   echo "规则C";
}
location ^~ /static/files {
    echo "规则X";
}
location ~ \.(gif|jpg|png|js|css)$ {
   echo "规则D";
}
location ~* \.png$ {
   echo "规则E";
}
location /img {
    echo "规则Y";
}
location / {
   echo "规则F";
}
```

那么产生的效果如下：

- 访问根目录 `/`，比如 `http://localhost/` 将匹配 `规则A`
- 访问 `http://localhost/login` 将匹配 `规则B`，`http://localhost/register` 则匹配 `规则F`
- 访问 `http://localhost/static/a.html` 将匹配 `规则C`
- 访问 `http://localhost/static/files/a.exe` 将匹配 `规则X`，虽然 `规则C` 也能匹配到，但因为**最大匹配原则**，最终选中了 `规则X`。你可以测试下，去掉规则 X ，则当前 URL 会匹配上 `规则C`。
- 访问 `http://localhost/a.gif`, `http://localhost/b.jpg` 将匹配 `规则D` 和 `规则 E` ，但是 `规则 D`顺序优先，`规则 E` 不起作用，而 `http://localhost/static/c.png` 则优先匹配到 `规则 C`
- 访问 `http://localhost/a.PNG` 则匹配 `规则 E` ，而不会匹配 `规则 D` ，因为 `规则 E` 不区分大小写。
- 访问 `http://localhost/img/a.gif` 会匹配上 `规则D`,虽然 `规则Y` 也可以匹配上，但是因为正则匹配优先，而忽略了 `规则Y`。
- 访问 `http://localhost/img/a.tiff` 会匹配上 `规则Y`。

访问 `http://localhost/category/id/1111` 则最终匹配到规则 F ，因为以上规则都不匹配，这个时候应该是 Nginx 转发请求给后端应用服务器，比如 FastCGI（php），tomcat（jsp），Nginx 作为反向代理服务器存在。

所以实际使用中，笔者觉得至少有三个匹配规则定义，如下：

```nginx
# 直接匹配网站根，通过域名访问网站首页比较频繁，使用这个会加速处理，官网如是说。
# 这里是直接转发给后端应用服务器了，也可以是一个静态首页
# 第一个必选规则
location = / {
    proxy_pass http://tomcat:8080/index
}

# 第二个必选规则是处理静态文件请求，这是 nginx 作为 http 服务器的强项
# 有两种配置模式，目录匹配或后缀匹配，任选其一或搭配使用
location ^~ /static/ {
    root /webroot/static/;
}
location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {
    root /webroot/res/;
}

# 第三个规则就是通用规则，用来转发动态请求到后端应用服务器
# 非静态文件请求就默认是动态请求，自己根据实际把握
# 毕竟目前的一些框架的流行，带.php、.jsp后缀的情况很少了
location / {
    proxy_pass http://tomcat:8080/
}
```

参考：

1. https://moonbingbing.gitbooks.io/openresty-best-practices/content/ngx/nginx_local_pcre.html
2. http://nginx.org/en/docs/http/ngx_http_core_module.html#location

## server_name 匹配顺序

server_name 指令是用来设置虚拟服务器的域名或 IP 地址。如果配置为域名，需要到系统本地 hosts 中配置 IP 地址。如果反向代理的负载是本地服务的话，可以直接用 localhost 或 127.0.0.1。

server_name 的匹配顺序如下：

1. 准确匹配

   ```nginx
   server {
        listen       80;
        server_name  domain.com  www.domain.com;
        ...
   }
   ```

   

2. 匹配以通配符 * 开始的 server_name

   ```nginx
   server {
        listen       80;
        server_name  *.domain.com;
        ...
   }
   ```

   

3. 匹配以通配符 * 结束的 server_name

   ```nginx
   server {
        listen       80;
        server_name  www.*;
        ...
   }
   ```

   

4. 匹配正则表达式

   ```nginx
   server {
        listen       80;
        server_name  ~^(?.+)\.domain\.com$;
        ...
   }
   ```

参考：http://nginx.org/en/docs/http/server_names.html

## 指令和变量

参考：

http://nginx.org/en/docs/dirindex.html

http://nginx.org/en/docs/varindex.html

## 正向代理

默认 Nginx 不支持 HTTP 的 CONNECT 方法，所以不支持 HTTPS 正向代理，只支持 HTTP 正向代理。

```nginx
server {
    #正向代理不能使用 server_name

    #指定DNS服务器IP地址
    resolver 114.114.114.114; 
    
    #设置代理端口，不一定是80
    listen 80;
    
    location / {
        proxy_pass http://$host$request_uri; #设定代理服务器的协议和地址
    }
}

```

如果想要 HTTPS 正向代理，需要添加 [ngx_http_proxy_connect_module](https://github.com/chobits/ngx_http_proxy_connect_module) 模块（需要编译）。

```nginx
server {
     listen  443;

     # dns resolver used by forward proxying
     resolver  114.114.114.114;

     # forward proxy for CONNECT request
     proxy_connect;
     proxy_connect_allow            443;
     proxy_connect_connect_timeout  10s;
     proxy_connect_read_timeout     10s;
     proxy_connect_send_timeout     10s;

     # forward proxy for non-CONNECT request
     location / {
         proxy_pass http://$host;
         proxy_set_header Host $host;
     }
 }
```

注意，HTTP/2 中已经不支持 CONNECT 方法，所以只支持 HTTP/1.x 的 HTTPS 正向代理。

参考：

[nginx正向代理配置详解](https://cloud.tencent.com/developer/article/1521322)

https://zhuanlan.zhihu.com/p/70459013

http://tengine.taobao.org/document/proxy_connect.html

## 反向代理与负载均衡

### HTTP 反向代理和负载均衡

Nginx HTTP 反向代理使用的模块是 ngx_http_proxy_module，核心指令是 proxy_pass；负载均衡使用的模块是 ngx_http_upstream_module，核心指令是 upstream。示例如下：

```nginx
http {
    #定义负载均衡服务器（默认使用轮询策略）
    upstream backend {
        server 127.0.0.1:8070;
        server 127.0.0.1:8080;
        server 127.0.0.1:8090;
    }

    #定义虚拟服务器，监听80端口，并将80端口的所有流量传递给上游upstream
    #注意upstream名称要和proxy_pass的名称匹配。
    server {
        listen 80;
        location / {
            #反向代理指令
            proxy_pass http://backend;
            #一般会加下面的指令设置HTTP请求头，也可以不加
            proxy_set_header Host      $host:$server_port;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

参考：

https://nginx.org/en/docs/http/load_balancing.html

https://nginx.org/en/docs/http/ngx_http_proxy_module.html

https://nginx.org/en/docs/http/ngx_http_upstream_module.html

### TCP、UDP反向代理和负载均衡

Nginx 自 1.9.0 版本起支持 TCP 反向代理和负载均衡，自 1.9.13 版本起支持 UDP 反向代理和负载均衡。

与 HTTP 不同，TCP 和 UDP 反向代理使用的模块是 ngx_stream_proxy_module，负载均衡使用的模块是 ngx_stream_upstream_module。



通过 63790 端口来 TCP 反向代理 Redis 的 6379 端口示例：

```nginx
stream {
  server {
    listen 63790;
    #请将下面的redis替换为实际的redis服务IP地址
    proxy_pass redis:6379;
  }
}
```



参考：

https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html

https://nginx.org/en/docs/stream/ngx_stream_upstream_module.html

### 负载均衡策略

Nginx 负载均衡是通过 upstream 模块来实现的，内置了三种负载策略。

- 轮循（round-robin）（默认） 

Nginx 根据请求次数，将每个请求均匀分配到每台服务器。如果设置了权重的话，会按照权重均匀分配给每台服务器。

- 最少连接（least-connected）

Nginx 会统计哪些服务器的连接数最少，然后将请求优先分配给连接数最少的服务器。

- IP Hash

第一次请求时，Nginx 会将客户端IP地址的哈希值绑定集群中的某台服务器，后续该客户端的所有请求都会转发给集群中绑定的那台服务器去处理。

参考：https://nginx.org/en/docs/http/load_balancing.html

## WebSocket 代理

```nginx
location /chat/ {
    proxy_pass http://backend;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

参考：https://nginx.org/en/docs/http/websocket.html

## 配置 HTTPS 服务器

```nginx
server {
    listen              80;
    #必须定义一个ssl监听端口
    listen              443 ssl;
    server_name         localhost;
    #必须指定ssl证书位置
    ssl_certificate     cert.pem;
    #必须指定ssl证书私钥
    ssl_certificate_key cert.key;
    ....
}
```



参考：https://nginx.org/en/docs/http/configuring_https_servers.html

## 80端口重定向到443端口

80 端口重定向到 443 端口，也就是 http 访问自动跳转到 https。

```nginx
#以下为核心配置
server {
    listen 80;
    server_name www.example.com;
    rewrite ^(.*)$ https://${server_name}$1 permanent; 
}
#以上为核心配置

server {
    listen 443;
    server_name www.example.com;
    root /home/wwwroot;
    ssl on;
    ssl_certificate /etc/nginx/certs/server.crt;
    ssl_certificate_key /etc/nginx/certs/server.key;
    ....
}
```



参考：https://blog.csdn.net/m0_37886429/article/details/72271983

## 日志格式

推荐使用 JSON 日志格式，清晰明了，也方便日志采集分析

参考：https://github.com/openresty/docker-openresty/blob/master/nginx.conf

## nginx-rtmp-module

下载地址：https://gitee.com/mirrors/nginx-rtmp-module

安装方式：

```bash
yum install -y pcre-devel.x86_64 openssl-devel.x86_64
./configure --add-module=/root/nginx-rtmp-module --prefix=/usr/local/nginx --with-debug
make & make install
```

**直播服务器配置**

```nginx
rtmp {
    server {
        listen 1935;

        application live {
            live on;
        }
    }
}
```

推流和拉流：

```bash
ffmpeg -i video.mp4 -r 25 -b 4M -f flv rtmp://{{server}}:1935/live/5
// 执行完上面的命令后，VLC播放器打开 rtmp://{{server}}:1935/live/5
```

**HLS配置**

```nginx
rtmp {
    server {
        listen 1935;

        application hls {
            live on;
            hls on;
            hls_path /var/hls;
        }
    }
}
http {
    server {
        listen      8080;

        location /hls {
            # Serve HLS fragments
            types {
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }
            root /var;
            add_header Cache-Control no-cache;
        }
    }
}
```

推流和拉流：

```bash
// 推流格式必须是 H264/AAC 编码
ffmpeg -i video.mp4 -vcodec libx264 -acodec aac -f flv rtmp://{{server}}:1935/hls/5
// 执行完上面的命令后，VLC播放器打开 http://{{server}}:8080/hls/5.m3u8
```

***视频录制配置**

```nginx
rtmp {
    server {
        listen 1935;
        
        application record {

            live on;

            # 录制不了，提示权限不够
            # 推流会自动开启录制
            record all;
            record_path /tmp/rec;
            record_unique on;
        }
    }
}
```

**视频点播配置**

```nginx
rtmp {
    server {
        listen 1935;

        # 视频点播（Video On Demand）
        # VLC播放器打开 rtmp://{{server}}:1935/vod/{{video.mp4}}
        application vod {
            play /tmp/vod;
        }
    }
}
```

**视频中继**

```nginx
rtmp {
    server {
        listen 1935;

        # 视频中继
        # VLC播放器打开 rtmp://{{server}}:1935/tv
        application tv {
            live on;

            # Pull all streams from remote machine
            # and play locally
            # 这里使用《CCTV-1综合》的RTMP流来测试
            pull rtmp://58.200.131.2:1935/livetv/cctv1;
        }
    }
}
```



参考：https://github.com/arut/nginx-rtmp-module

# *OpenResty

# *Kong