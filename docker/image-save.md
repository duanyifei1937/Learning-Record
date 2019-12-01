# Image save

* docker image 迁移及保存

```
docker save contrail-analyticsdb-u14.04:4.0.0.0-3016 | gzip -c > contrail-analyticsdb-u14.04-4.0.0.0-3016.tar.gz

docker load < busybox.tar.gz
```

* 系统代理
    * polipo 制作web proxy
    * https://droidyue.com/blog/2016/04/04/set-shadowsocks-proxy-for-terminal/index.html