# learn-kubernetes

https://www.digitalocean.com/community/tutorials/elasticsearch-fluentd-and-kibana-open-source-log-search-and-visualization

## DNS
k8s中DNS pod包括三个容器：kubedns，dnsmasq和healthz。

k8s使用skydns作为DNS server，我们只给service分配DNS name，每个service会被分配一个虚拟IP，销毁该Service之前，这个IP都不会再变化。相比之下，pod IP会随着pod的销毁而消失，service的IP映射到DNS。

Service的是Cluster IP，有nodePort。

##外部访问Service
一个Service可以看作一组提供相同服务的Pod的对外访问接口，Service作用于哪些Pod是通过Label Selector来定义。
Kubernetes支持两种对外提供服务的Service的type定义：NodePort和LoadBalancer。
 - NodePort
   指定nodePort的值，系统就会在Kubernetes集群中的每个Node上打开一个主机上的真实端口号。能够访问Node的客户端就能通过这个端口访问到内部的Service了。

 - LoadBalancer
   如果云服务商支持外接负载均衡器，

### 如何找到DNS server

##命令


