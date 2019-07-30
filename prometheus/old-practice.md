# Old Practice
> 介绍了之前使用prometheus的实践

对基于文件发现的prometheus进行了修改，同时使用k8s和Consul作为服务发现,之后推荐prometheus-operator进行MonitorTarget管理；

## 痛点1：
之前prometheus使用static file service discover,每次增加或删除节点需要修改配置文件、或者修改数据库记录重新生成target json file,过程繁琐、容易遗漏，对其他同学不可见; 修改为采用consul动态发现target,并使用consul api添加服务监控；

## prometheus 通过consul动态修改target接入

###  consul介绍
consul同zk/eureka一样，属于服务注册组件，但属于[此文章](https://mp.weixin.qq.com/s/zoadNOzu8q2FGBNYeM6RVw)的模式三；

### consul monitor target
```
node:
master-server: 3
client: 66(非k8s节点)

srevice:
未在业务代码中加入服务注册，使用consul api添加(service / port / node_ip)

service_name 用于alertmanager 子路由，因此命名规范为：
op_xx / dw_xx / kvision_xx 等

针对尚未注册的服务请联系添加端口探测，七层监控也将添加 in future.
```

        
