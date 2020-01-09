# 写性能优化

``` yaml
{
  "default" : {
    "order" : 0,
    "index_patterns" : [
      "*"
    ],
    "settings" : {
      "index" : {
        "refresh_interval" : "60s",
        "number_of_shards" : "3",
        "translog" : {
          "flush_threshold_size" : "1g",
          "sync_interval" : "60s",
          "durability" : "async"
        },
        "number_of_replicas" : "1",
        "merge" : {
          "scheduler" : {
            "max_thread_count" : "1",
            "max_merge_count" : "100"
          },
          "policy" : {
            "max_merged_segment" : "1g"
          }
        }
      }
    },
    "mappings" : { },
    "aliases" : { }
  }
}

```

* https://www.elastic.co/guide/en/elasticsearch/reference/current/tune-for-indexing-speed.html#multiple-workers-threads