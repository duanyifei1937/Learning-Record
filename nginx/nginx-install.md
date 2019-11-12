# Nginx Install

## Install
``` bash
sudo yum -y install yum-utils \
&& sudo yum-config-manager --add-repo https://openresty.org/yum/cn/centos/OpenResty.repo \
&& sudo yum -y install openresty \
&& sudo ln -s /usr/local/openresty/nginx /usr/local/ \
&& sudo rm -rf /usr/local/nginx/logs \
&& sudo mkdir -p /data/logs/nginx/ \
&& sudo ln -s /data/logs/nginx/ /usr/local/nginx/logs
```

## Logrotate
> 切割日志

``` yaml
cat /etc/logrotate.d/nginx
/data/logs/nginx/*log {
create 0644 nginx nginx
daily
rotate 5
missingok
notifempty
compress
sharedscripts
postrotate
/bin/kill -USR1 `cat /run/nginx.pid 2>/dev/null` 2>/dev/null || true
endscript
}
```

## Logstash Filebeat Collect
start:
`/data/deploy/logstash-6.1.1/bin/logstash -f /data/deploy/logstash-6.1.1/nginx_logstash.conf`

## Deploy Step
单节点测试；
修改LB测试；
