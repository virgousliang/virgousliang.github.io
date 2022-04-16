# docker快速安装nginx以及实现反向代理(上)

# 1、创建Nginx镜像

## 1、下载nginx镜像

```java
docker pull nginx
```

> 不指定版本默认拉取最新的 latest

## 2、创建nginx容器

```
docker run --name nginx -p 80:80 -d nginx
```

> --name  :指的是创建容器后的名字
>
> -d : 指的是对应镜像名 

## 3、改变nginx配置

想要通过nginx实现负载均衡的话就需要对`nginx.conf`配置文件进行配置，那么在`docker`里面怎么对配置文件完成修改呢? 步骤如下：

### 3.1进入容器

```java
docker exec -it nginx /bin/bash
```

> --it 后面跟的为nginx容器名

**大家在linux上安装docker目录结构均一致，下面默认以我本机操作讲解：**

配置文件在容器内`/etc/nginx/`路径下的`nginx.conf`和`conf.d`文件夹内的`default.conf`文件。见下图：

![](C:\Users\chengl\Desktop\QQ截图20220121170733.png)

这个时候我们需要通过`vim`修改指定文件。但`docker`里面是没有安装的

### 3.2docker内安装vim

在当前目录下执行两条命令即可。

```
apt-get update
apt-get install vim
```

这里是可以配加速的。感兴趣的读者可行摸索，接下来等待安装 即可

### 3.3修改config实现反向代理

调用命令如下，进入配置文件

```
vim nginx.cong
```

文件内容大致 如下：

```yaml
user nginx;
worker_processes auto;

error_log /var/log/nginx/error.log notice;
pid    /var/run/nginx.pid;

events {
  worker_connections 1024;
}

http {
  include   /etc/nginx/mime.types;
  default_type application/octet-stream;

  log_format main '$remote_addr - $remote_user [$time_local] "$request" '
           '$status $body_bytes_sent "$http_referer" '
           '"$http_user_agent" "$http_x_forwarded_for"';

  access_log /var/log/nginx/access.log main;

  sendfile    on;
  #tcp_nopush  on;

  keepalive_timeout 65;

  #gzip on;

 #include /etc/nginx/conf.d/*.conf;
  server {
    location / {
      proxy_pass http://192.168.118.130:80/;
      proxy_redirect default;
   }
  }
}
```

添加反向代理内容如下：

```xml
server {
    location / {
      proxy_pass http://192.168.118.130:80/;
      proxy_redirect default;
   }
  }
```

> 做几点说明：
>
>  #include /etc/nginx/conf.d/*.conf 改行文件是没注释的，需要人工注释，只有注释掉才会不走欢迎页面
>
> 直接会请求到代理地址
>
> 原因：因为/docker nginx/conf/conf.d中的default.conf文件夹内有这个location /配置，在nginx.conf中通过include使用了这个文件，优先匹配。

将配置文件保存后退出。

### 3.4重启nginx容器

```
docker restart nginx
```

> 这里restart后面跟即可以跟nginx容器id也可以跟容器name 这里博主跟的是容器name



