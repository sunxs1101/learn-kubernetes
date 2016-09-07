# learn-kubernetes

https://www.digitalocean.com/community/tutorials/elasticsearch-fluentd-and-kibana-open-source-log-search-and-visualization

## A Practical and Comprehensive Guide to DNS
k8s中DNS pod包括三个容器：kubedns，dnsmasq和healthz。

k8s使用skydns作为DNS server，我们只给service分配DNS name，每个service会被分配一个虚拟IP，销毁该Service之前，这个IP都不会再变化。相比之下，pod IP会随着pod的销毁而消失，service的IP映射到DNS。
service的是clusterIP，cluster_ip_range设置：
 - http://kubernetes.io/docs/getting-started-guides/ubuntu/
 - https://coreos.com/kubernetes/docs/latest/getting-started.html

SERVICE_CLUSTER_IP_RANGE is the virtual IP range that will be used by Kubernetes services, which is managed by kube-proxy on each of your nodes.
FLANNEL_NET is the virtual IP range used for pods/containers. It is managed by flanneld.

### supported DNS schema
"Normal"(not headless) Services被分配一个DNS记录，名字形式为：my-svc.my-namespace.svc.cluster.local，这个解析为Service的cluster IP。

"Headless"(without a cluster IP) Services也会被分配一个DNS记录，名字形式为my-svc.mynamespace.svc.cluster.local，不同于normal Services，这个解析成Service选出的pods的IP集合

SRV



### 如何找到DNS server

### DNS解析示例
 1. [ceph-docker](https://github.com/ceph/ceph-docker/tree/master/examples/kubernetes)
 


 2. [harbor](https://github.com/vmware/harbor/blob/master/docs/kubernetes_deployment.md#deploying-harbor-on-kubernetes)

##外部访问Service
一个Service可以看作一组提供相同服务的Pod的对外访问接口，Service作用于哪些Pod是通过Label Selector来定义。
Kubernetes支持两种对外提供服务的Service的type定义：NodePort和LoadBalancer。
 - NodePort：
   指定nodePort的值，系统就会在Kubernetes集群中的每个Node上打开一个主机上的真实端口号。能够访问Node的客户端就能通过这个端口访问到内部的Service了。NodePort的端口号定义有范围限制，默认为30000～32767

 - LoadBalancer：
   如果云服务商支持外接负载均衡器，


## [kubernetes network](https://coreos.com/kubernetes/docs/latest/kubernetes-networking.html)



##在k8s中部署Hello World

这个Hello World例子是一个Web留言板应用，基于PHP+Redis的两层分布式架构的Web应用，前端PHP Web网站通过访问后端Redis数据库来完成用户留言的查询和添加等功能。这个例子部署在kubernetes集群中，具有Redis读写分离能力。

为了实现读写分离，在Redis层采用了一个Master和两个Slave的高可用集群模式进行部署。其中Master实例用于前端写操作（添加留言），而两个Slave实例则用于前端读操作（读取留言）。











