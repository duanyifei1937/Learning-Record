# Kingeye-Monitor
> 金睛刚成立时，还没有监控，可以说一步步做起来的，主要经历了一下三个阶段；

## file discover prometheus
最开始使用prometheus文件服务发现方式，通过手动修改target，由于频繁上下线机器，Manual过于繁琐；
为此，写了一套自动添加节点的python项目：
    * 监控的目标节点及端口通过数据库生成目标json文件；
    * 接口化添加target、blackbox port，提交修改到mysql;

## k8s + consul prometheus
由于使用k8s, k8s cluster 提供原生service discover, 在通过consul注册、管理其他非k8s节点，实践监控；
* 优点：
    * 只需要针对非k8s节点进行manual管理，注册有mysql改成consul接口注册；
    * consul提供页面可视化；
    * 支持跨VPC,做线上、线下管理；
* 缺点：
    * consul daemon process 部署在node, 如果该daemon进程停掉，无法报警；
    * consul只用了管理节点、端口探测，未与业务使用的服务对接；(只是给运维用的工具，其实远不止于此)

## prometheus-operator
使用了ceph,将prometheus部署到了集群内部，原生提供报警的高可用，集群外通过MonitorTarget注册到k8s,虽然硬编码，但更加原生；
主要讲解这种方式的部署注意点~