# logstash timestamp overwrite

* 在`filebeat --> kafka --> logstash --> ES`收集日志流程中，以filebeat push time作为 timestamp, 而且log message中依然会有time, 当filiebeat 文件句柄过多，push延时，就需要使用logstash 将message 中的时间戳覆盖掉 filebeat metadata `@timestamp`；

* 配置如下：

``` yaml
    filter {
      if [service] == "picturebook" {
        grok {
          patterns_dir => ["/usr/share/logstash/patterns"]
          patterns_files_glob => "*.patterns"
          match => {"message" => [
                    "%{CUSTOMDATE:logdate}",
                    "%{HTTPDATE:logdate}"
                    ]
          }
        }
        date {
          match => ['logdate', 'dd/MMM/YYYY:HH:mm:ss Z','YYYY/MM/dd HH:mm:ss.SSSSSS']
          timezone => "+08:00"
          locale => "en"
          target => "@timestamp"
        }
      }
      mutate {
        remove_field => ["offset","docker_container","beat"]
      }
    }



    filter {
      if [service] == "rtcsyncrecord" {
        grok {
          patterns_dir => ["/usr/share/logstash/pipeline"]
          patterns_files_glob => "*.patterns"
          match => {
             #"message" => ["%{YEAR:year}/%{MONTHNUM:month}/%{MONTHDAY:day} %{TIME:time}\t%{GREEDYDATA:alldata}"]
             "message" => "%{CUSTOMDATE:logdate}\t%{GREEDYDATA:alldata}"
          }
          add_field => {
            "logtime" => "%{month} %{day} %{year} %{time}"
            "year" => "%{year}"
            "name" => "duanyifei"
            "logdate" => "%{logdate}"
            "alldata" => "%{alldata}"
          }
        }
        date {
          #match => ['logtime', 'dd/MMM/YYYY:HH:mm:ss Z','YYYY-MM-dd HH:mm:ss.SSS']
          #match => ['logtime', 'MM dd yyyy HH:mm:ss.SSSSSS']
        }
      }
      else {
        drop { }
      }
      #mutate {
      #  remove_field => ["offset","docker_container","beat"]
      #}
    }
```