## Kubernetes存储

Docker中的数据是临时的，当容器销毁时，其中的数据会丢失。如果需要持久化容器，需要使用Docker
Volume挂载宿主机上的文件目录到容器中。
Kubernetes中的Volume是基于Docker进行扩展。
