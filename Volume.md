## Kubernetes存储

Docker中的数据是临时的，当容器销毁时，其中的数据会丢失。如果需要持久化容器，需要使用Docker
Volume挂载宿主机上的文件目录到容器中。
Kubernetes中的Volume是基于Docker进行扩展。

#### mount

```
客户端和服务器端都需要启动portmap和nfs的service
服务器端编辑/etc/exports，添加需要共享的文件系统，然后service nfs restart

客户端配置/etc/fstab，设置需要mount的目录
客户端也可以手动mount -t nfs 1.1.1.1:/remote /local过来


```

