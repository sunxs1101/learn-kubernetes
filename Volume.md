## Kubernetes存储

#### NFS mount

```
客户端和服务器端都需要启动portmap和nfs的service
服务器端编辑/etc/exports，添加需要共享的文件系统，然后service nfs restart

客户端配置/etc/fstab，设置需要mount的目录
客户端也可以手动mount -t nfs 1.1.1.1:/remote /local过来

```

### Docker volume

### Kubernetes persist

Docker中的数据是临时的，当容器销毁时，其中的数据会丢失。如果需要持久化容器，需要使用Docker
Volume挂载宿主机上的文件目录到容器中。
Kubernetes中的Volume是基于Docker进行扩展。

http://www.open-open.com/lib/view/open1438593641817.html 笔记如下
```
nfs：Kubernetes中通过简单的配置就可以挂载NFS到Pod中，而NFS中的数据时刻要永久保存的，同时NFS支持同时写操作。
```
#### Kuberneters: Persistent Volumes and Claims
学习[Persistent Volumes and Claims](https://github.com/kubernetes/kubernetes/blob/release-1.0/docs/user-guide/persistent-volumes.md)之前，首先我们要了解kubernetes中[volumes](https://github.com/kubernetes/kubernetes/blob/release-1.0/docs/user-guide/volumes.md)的概念。

容器中的磁盘文件是短暂存储的，当一个容器挂掉，文件会丢失;当在pod中运行容器，有必要再容器间共享数据;k8s的Volume抽象解决这两个问题。Docker也有[volumes](https://docs.docker.com/engine/tutorials/dockervolumes/)，在Docker中，一个volume只是磁盘或另一个容器中的目录，其生命周期不被管理，直到最近也只有local-disk-backed volumes，现在Docker提供volumes驱动，但是功能也很有限。

kubernetes volume和pod一样有生命周期，因此，一个volume会比运行在Pod中的任一容器更长久，容器重启时数据会被保存。
当然，当一个Pod不存在后，volume也会不存在。更重要的是，kubernetes支持多种类型volume，一个Pod可以同时使用任一数量的volumes。

最核心的，一个volume只是一个目录，可能在里面有一些数据，pod中容器可以访问这些数据，这些都由特定的volume使用。为了使用一个volume，一个pod要制定volume类型，挂载到容器的什么地方。容器中一个进程将文件系统看作是由Docker镜像和volumes组成，Docker image是这个文件系统的根基，任一volumes被挂载到镜像中指定路径。volume不能被挂载到其他volumes，或者与其他volume有连接。Pod中的每个容器互相独立的指定将volume挂载到哪儿。

#### Volume类型

 - emptyDir
 - hostPath
 - gcePersistentDisk
 - awsElasticBlockStore
 - nfs: nfs volume允许一个存在的NFS被挂载到pod中
 - iscsi
 - glusterfs
 - rbd
 - gitRepo
 - secret
 - persistentVolumeClaim

一个persistentVolumeClaim volume被用于挂载一个PersistentVolume到pod上，对用户而言，PersistentVolumes可以用来声明持久存储(比如GCE PersistentDisk或iSCSI volume)而不需要知道特定云环境的细节。下面具体介绍Persistent Volumes and Claims

PersistentVolume子系统为用户和管理员提供了一个API，它从存储如何使用摘录出存储如何提供的细节，我们介绍两个新的API资源：PersistentVolume和PersistentVolumeClaim。

PersistentVolume(PV)是集群中的一块网路存储，由管理员提供。PVs是volume plugins, 但是有一个独立于任一单个pod



































### CephFS mount








