# Nginx Upstream Healch Check 
## 背景
  * 解决无损上线及Nginx-upstream后端层故障；

## 目前配置及存在的问题
```
server {
	listen 80;
	server_name internal.abc.net;

	location /dw {
        proxy_pass http://internal-ai-dw;
    }
}

upstream internal-ai-dw {
        server b.b.b.b:8020;
        server a.a.a.a:8020;
}
```
* 上游失效状态：
  * timeout
  * connect refuse
  * 5xx (500/502/503/504)
* 默认Nginx只针对前两种,无法根据后端状态码进行切换；

## 完善后端健康检查；

### module1: ngx_http_proxy_module + ngx_http_upstream_module(自带)
#### 配置及解析
```
upstream name {
        server 10.1.1.110:8080 max_fails=1 fail_timeout=10s;
        server 10.1.1.122:8080 max_fails=1 fail_timeout=10s;
}

http {
	proxy_next_upstream http_502 http_504 http_404 error timeout invalid_header;
}

# 10s内出现出现proxy_next_upstream定义的错误max_fails次，将该后端视为不健康；
# proxy_next_upstream定义:
Syntax:		proxy_next_upstream error | timeout | invalid_header | http_500 | http_502 | http_503 | http_504 | http_403 | http_404 | http_429 | non_idempotent | off ...;
Default:	proxy_next_upstream error timeout;
Context:	http, server, location
```
#### 优缺点
* 如果后端有不健康节点，负载均衡器依然会先把该请求转发给该不健康节点，然后再转发给别的节点，这样就会浪费一次转发。
* 可是，如果当后端应用重启时，重启操作需要很久才能完成的时候就会有可能拖死整个负载均衡器。此时，由于无法准确判断节点健康状态，导致请求hang住，出现假死状态，最终整个负载均衡器上的所有节点都无法正常响应请求。
* 并且ngx_http_upstream_module模块中的server指令中的max_fails参数设置值，也会和ngx_http_proxy_module 模块中的的proxy_next_upstream指令设置起冲突。比如如果将max_fails设置为0，则代表不对后端服务器进行健康检查，这样还会使fail_timeout参数失效（即不起作用）。
* 此时，其实我们可以通过调节ngx_http_proxy_module 模块中的 proxy_connect_timeout 指令、proxy_read_timeout指令，通过将他们的值调低来发现不健康节点，进而将请求往健康节点转移。
* 不推荐的方式。

``` bash
proxy_connect_timeout 60s;	ngx 与 proxy_server 连接超时时间，不能超过75s;
proxy_read_timeout 60s;		与proxy_server读超时时间，决定ngx 等待多长时间去获得 proxy_server 请求响应不是获取整个response,而是两次reading操作的时间.
proxy_send_timeout 60s;		ngx 给 proxy_server 发送请求的超时时间；不是整个发送时间，而是两次write时间；超时后仍然没有收到数据，连接关闭。
ngx & proxy 建立连接   -- connect_timeout
ngx --> proxy 发送请求 -- send_timeout
ngx <-- proxy 等待响应 -- read_timeout
```

### module2: upstream_check_module
> 更专业的模块，来专门提供负载均衡器内节点的健康检查的。淘宝tengine

#### 添加第三方模块:
> 需要重新编译

```
1. 查看当前版本：
$ nginx -v
nginx version: nginx/1.12.2

2. 当前已经安装module:
$ nginx -V
--prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi --http-scgi-temp-path=/var/lib/nginx/tmp/scgi --pid-path=/run/nginx.pid --lock-path=/run/lock/subsys/nginx --user=nginx --group=nginx --with-file-aio --with-ipv6 --with-http_auth_request_module --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_addition_module --with-http_xslt_module=dynamic --with-http_image_filter_module=dynamic --with-http_geoip_module=dynamic --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_slice_module --with-http_stub_status_module --with-http_perl_module=dynamic --with-mail=dynamic --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream=dynamic --with-stream_ssl_module --with-google_perftools_module --with-debug --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -m64 -mtune=generic' --with-ld-opt='-Wl,-z,relro -specs=/usr/lib/rpm/redhat/redhat-hardened-ld -Wl,-E'

3. 下载对应源码包：
wget -P /data/soft http://nginx.org/download/nginx-1.12.2.tar.gz
tar -xaf /data/soft/nginx-1.12.2.tar.gz -C /data/soft
cd /data/soft/nginx-1.12.2

4. 下载module:
$ sudo git clone https://github.com/yaoweibin/nginx_upstream_check_module.git
$ cd nginx_upstream_check_module/

5. patch：
$ sudo yum -y install pcre pcre-devel openssl openssl-devel gcc-c++ autoconf automake zlib-devel libxml2 libxml2-dev libxslt-devel gd-devel perl-devel perl-ExtUtils-Embed GeoIP GeoIP-devel GeoIP-data gperftools-devel
$ cd /data/soft/nginx-1.12.2
$ patch -p1 < /data/soft/nginx-1.12.2/nginx_upstream_check_module/check_1.12.1+.patch
$ ./configure --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi --http-scgi-temp-path=/var/lib/nginx/tmp/scgi --pid-path=/run/nginx.pid --lock-path=/run/lock/subsys/nginx --user=nginx --group=nginx --with-file-aio --with-ipv6 --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_addition_module --with-http_xslt_module=dynamic --with-http_image_filter_module=dynamic --with-http_geoip_module=dynamic --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_slice_module --with-http_stub_status_module --with-http_perl_module=dynamic --with-mail=dynamic --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream=dynamic --with-stream_ssl_module --with-google_perftools_module --with-debug --add-module=/data/soft/nginx-1.12.2/nginx_upstream_check_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -m64 -mtune=generic' --with-ld-opt='-Wl,-z,relro -specs=/usr/lib/rpm/redhat/redhat-hardened-ld -Wl,-E'
$ make (不能make install 不然产生覆盖)；
$ cp /usr/sbin/nginx /data/soft/nginx-1.12.2/nginx_bak
$ 源码目录生成objs/nginx  覆盖之前nginx执行程序； 
$   * cp -rfp objs/nginx ../nginx
$ nginx -t
$ kill -USR2 9647
# nginx -s reload 不生效，需要kill -USR2
```
* 升级重启：
  * 平滑升级的过程，Nginx服务器接受到USR2信号后，首先将旧的nginx.pid文件添加后缀.oldbin，变为nginx.pid.oldbin文件，然后执行新版本的Nginx服务器的二进制的文件启动服务，这个时候需要提前将编译好的新版本的二进制实现复制到sbin文件夹中。如果新的服务启动成功，系统中将有新旧两个Nginx服务共同提供Web服务。之后，需要向旧的Nginx服务器进程发送WINCH信号，使旧的Nginx服务平滑停止，并删除nginx.pid.oldbin文件。在发送WINCH信号之前，如果发现有什么错误，可以随时停止新的Nginx服务。

* 快速停止Nginx服务：快速停止是指立即停止当前Nginx服务正在处理的所有网络请求，马上丢弃连接，停止工作
``` yaml
[root@c7node1 ~]# kill -TERM `cat /run/nginx.pid`
[root@c7node1 ~]# kill -INT `cat /run/nginx.pid`
```

* 平缓停止Nginx服务：平缓停止是指允许Nginx服务将当前正在处理的网络请求处理完成，但不再接受新的请求，之后关闭连接，停止工作
``` yaml
[root@c7node1 ~]# kill -QUIT `cat /run/nginx.pid`
```

* 平缓重启Nginx服务：Nginx服务进程接受到信号后，首先读取新的Nginx配置文件，如果配置语法正确，则启动新的Nginx服务，然后平缓关闭旧的服务进程，如果新的Nginx配置文件有问题，将显示错误，仍然使用旧的Nginx进程提供服务
``` yaml
[root@c7node1 ~]# kill -HUP `cat /run/nginx.pid`
```

* 日志切割：重新打开日志文件，常用于日志切割
``` yaml
[root@c7node1 ~]# kill -USR1 `cat /run/nginx.pid`
```

* 平缓升级Nginx服务：使用新版本的Nginx文件启动服务，之后平缓停止原有的Nginx进程
``` yaml
[root@c7node1 ~]# kill -USR2 `cat /run/nginx.pid`
```

* 平缓停止worker process：用于Nginx服务平缓升级
``` yaml
[root@c7node1 ~]# kill -WINCH `cat /run/nginx.pid`
```

* 信号指令 & 平滑重启

| TERM, INT | 快速关闭 | 
| --- | --- |
| QUIT | 从容关闭  |
| HUP | 重载配置用新的配置开始新的工作进程从容关闭旧的工作进程 |
| USR1 | 重新打开日志文件 |
| USR2 | 平滑升级可执行程序 |
| WINCH | 从容关闭工作进程|

* 探测机制：

```
  * upstream cluster {
    server 192.168.0.1:80;
    server 192.168.0.2:80;
    check interval=500 rise=2 fall=2 timeout=1000 type=http;
    check_http_send "HEAD / HTTP/1.0\r\n\r\n";
    check_http_expect_alive http_2xx http_3xx http_4xx;
}

# 每500ms进行探测，失败两次或1s内连接超时认为失败；
# 2xx 3xx 4xx 认为成功
# 全部失败后重新探测
# 每次间隔发送三个head测试头，三个全部失败，标记一次，并发严格按照规则进行切走
# 并非使用真实请求做计数统计，发送head probe进行探测；
```

#### 配置及解析

```
http {
    upstream cluster {
        # simple round-robin
        server 192.168.0.1:80;
        server 192.168.0.2:80;
        check interval=5000 rise=1 fall=3 timeout=4000;
        #check interval=3000 rise=2 fall=5 timeout=1000 type=ssl_hello;
        #check interval=3000 rise=2 fall=5 timeout=1000 type=http;
        #check_http_send "HEAD / HTTP/1.0\r\n\r\n";
        #check_http_expect_alive http_2xx http_3xx;
    }
    server {
        listen 80;
        location / {
            proxy_pass http://cluster;
        }
        location /status {
            check_status;
            access_log   off;
            allow SOME.IP.ADD.RESS;
            deny all;
       }
    }
}
```

#### 模块测试：

```
# 配置：
upstream duan {
    server localhost:8081;
    server localhost:8082;
    check interval=10000 rise=2 fall=2 timeout=1000 type=http;
    check_http_send "HEAD / HTTP/1.0\r\n\r\n";
    check_http_expect_alive http_2xx http_3xx;
}

# error_log:
2018/04/03 11:12:30 [error] 12799#0: check protocol http error with peer: 127.0.0.1:8082
2018/04/03 11:12:37 [error] 12801#0: check protocol http error with peer: 127.0.0.1:8082
2018/04/03 11:12:38 [error] 12800#0: check protocol http error with peer: 127.0.0.1:8082
2018/04/03 11:12:40 [error] 12799#0: check protocol http error with peer: 127.0.0.1:8082
2018/04/03 11:12:47 [error] 12801#0: check protocol http error with peer: 127.0.0.1:8082
2018/04/03 11:12:48 [error] 12800#0: check protocol http error with peer: 127.0.0.1:8082
2018/04/03 11:12:50 [error] 12799#0: check protocol http error with peer: 127.0.0.1:8082

# 测试发送HEAD头与访问请求是独立的,单独发送探测信息；
```

### module3: http_healthcheck_module
> 不再维护；

## 线上添加后端状态码健康检查模块
> 推荐ngx_upstream_check_module

### 实现动态配置后端
> 通过一个简单的HTTP接口不需要重启nginx配置上游服务器组。http或stream服务器组必须位于共享内存。

-------

#### ngx_http_upstream_conf_module：

#### dynamic upstream:

* 实现功能：
  view the group configuration;
  view, modify, or remove a server;
  add a new server.

```
http {

    include conf/upstream.conf;

    server {
        listen   8080;

        location / {
            # The upstream here must be a nginx variable
            proxy_pass http://$host; 
        }
    }

    server {
        listen 8088;
        location / {
            return 200 "8088";
        }
    }

    server {
        listen 8089;
        location / {
            return 200 "8089";
        }
    }

    server {
        listen 8081;
        location / {
            dyups_interface;
        }
    }
}

upstream host1 {
    server 127.0.0.1:8088;
}

upstream host2 {
    server 127.0.0.1:8089;
}
```
##### dyups_module缺点
* no well work with upstream_check_module;(github错误)
  * 即使服务不可用，由于dyups_module优先级高于check_module,使得也可以调用；
* 更新数据不生成配置文件，保存于内存重启失效；
* 不能对upstream group内单独一台server操作，只能更新全部组；

#### lua-upstream-nginx-module
-------

参考文章：
[基于Nginx dyups模块的站点动态上下线并实现简单服务治理](http://blog.51cto.com/tenderrain/1966423)
[大众点评-Camel](http://leonindy.coding.me/camel_in_action/posts/ch1-overview/)
[Camel-git](https://github.com/leonindy/camel)