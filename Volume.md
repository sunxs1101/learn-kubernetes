## Kubernetes存储

#### NFS mount

```
服务器设置共享目录，本地建立要挂载位置的目录。将服务器目录mount到客户端，使得在客户端使用

客户端和服务器端都需要启动portmap和nfs的service
服务器端编辑/etc/exports，添加需要共享的文件系统，然后service nfs restart
客户端配置/etc/fstab，设置需要mount的目录，这样每次重启后就会自动挂载服务器的NFS共享目录。
客户端也可以手动mount -t nfs 1.1.1.1:/remote /local过来

NFS是一个文件系统，本身没有提供信息传输的协议和功能，它利用RPC(Remote Procedure Call)实现信息的传输。
```

### Docker volume


### Kubernetes persist

Docker中的数据是临时的，容器的文件系统与容器的生命周期一致，当容器退出后其文件系统也随之被销毁，因此需要额外的持久化存储介质。如果需要持久化容器，需要使用Docker Volume挂载宿主机上的文件目录到容器中。

#### Kuberneters: Persistent Volumes and Claims
Kubernetes中的Volume是基于Docker进行扩展，学习[Persistent Volumes and Claims](https://github.com/kubernetes/kubernetes/blob/release-1.0/docs/user-guide/persistent-volumes.md)之前，首先我们要了解kubernetes中[volumes](https://github.com/kubernetes/kubernetes/blob/release-1.0/docs/user-guide/volumes.md)的概念。

容器中的磁盘文件是短暂存储的，当一个容器挂掉，文件会丢失;当在pod中运行容器，有必要在容器间共享数据;k8s的Volume抽象解决这两个问题。Docker也有[volumes](https://docs.docker.com/engine/tutorials/dockervolumes/)，在Docker中，一个volume只是磁盘或另一个容器中的目录，其生命周期不被管理，直到最近也只有local-disk-backed volumes，现在Docker提供volumes驱动，但是功能也很有限。

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

PersistentVolume(PV)是集群中的一块网路存储，由管理员提供。PVs像Volumes一样，是volume plugins, 但是它有一个独立于任一单个使用PV的pod的声明周期，这个API对象抓住存储细节。

一个PersistentVolumeClaim(PVC)是一个用户提出的存储请求，和pod很类似。Pods消耗节点资源，PVCs消耗PV资源，Pods需要特定资源(CPU和内存)。Claims需要特定大小和访问模式。

#### 一个volume和claim的生命周期

PVs是集群中的资源，PVC是对这些资源的请求，也对资源claim checks，PVs和PVCs之间的交互遵循下面的生命周期：

 - Provisioning
 
集群管理者创建一些PVs，他们有实际存储的细节

 - Binding

用户创建一个有特定容量请求和访问模式的PersistentVolumeClaim，master中的一个控制循环监测新的PVCs，寻找一个匹配的PV，并将他们绑定。用户会得到他们想要的，但是这个volume可能超出所需。

Claims会仍然不受限制，如果一个匹配的volume不存在。比如，一个集群提供很多50G volume不会匹配一个要求100G的PVC，当一个100G的PV加入集群后，这个PVC会受限。

 - Using

Pods使用claims作为volumes，集群检查claim来找bound volume，并为pod挂载这个volume。对于支持多种访问模式的volumes，当使用claim作为pod中的volume时，用户设置模式。

一旦用户有一个claim并且claim是有限制的，这个有限制的PV属于用户。用户调度Pods，通过将一个PersistentVolumeClaim包括在他们的Pod volume块中来访问他们声明的PVs。

 - Releasing

当一个用户做完他们的volume后，他们可以从API中删除PVC对象，这些API允许资源的reclamaton。当claim被删除后，volume被认为是发布了。

 - Reclaiming

一个PersistentVolume的reclaim策略告诉集群当volume发布后，如何操作volume。当前，volumes可以被保存或回收，Retention允许对资源的人工reclamation，对于支持它的volume插件，recycling在volume上执行基本操作，使得它对于一个新的claim有效。

### CephFS mount

在[https://github.com/kubernetes/kubernetes/tree/master/examples/volumes/cephfs]()中CephFS的







