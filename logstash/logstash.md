# Logstash

# build local test env
* nginx:
    * logs: /usr/local/Cellar/nginx/1.15.7/logs
    * conf: /usr/local/etc/nginx


``` yaml
input {
    beats {
        port => "5044"
    }
}
filter {
	if [message] !~ /panic/ {
	    drop { }
	}
}
output {
    file {
        path => "/usr/share/logstash/data/abc.log"
    }
}
```