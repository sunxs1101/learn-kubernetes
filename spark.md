## 使用Docker和Kubernetes部署Spark集群

相比于在物理机上部署，在Kubernetes集群上部署Spark集群，具有以下优势：
 1. 快速部署：安装1000台级别的Spark集群，在Kubernetes集群上只需设定worker副本数目replicas=1000，即可一键部署。
 2. 快速升级：升级Spark版本，只需替换Spark镜像，一键升级。
 3. 弹性伸缩：需要扩容、缩容时，自动修改worker副本数目replicas即可。
 4. 高一致性：各个Kubernetes节点上运行的Spark环境一致、版本一致
 5. 高可用性：如果Spark所在的某些node或pod死掉，Kubernetes会自动将计算任务，转移到其他node或创建新pod。
 6. 强隔离性：通过设定资源配额等方式，可与WebService应用部署在同一集群，提升机器资源使用效率，从而降低服务器成本。

接下来我们将介绍，如何使用 Docker和Kubernetes创建Spark集群（其中包含1个Spark-master和N个Spark-worker）。
参考文档：https://github.com/kubernetes/kubernetes/tree/master/examples/spark

首先需要准备以下工具：
● 安装并运行 kubernetes 集群
● 安装 kubectl 命令行工具

一、构建Docker镜像
1. 从源码build：
下载：https://github.com/kubernetes/application-images/blob/master/spark
```
docker build -t index.caicloud.io/spark:1.5.2 .
docker build -t index.caicloud.io/zeppelin:0.5.6 zeppelin/
docker login index.docker.io
docker pull index.docker.io/caicloud/spark:1.5.2
docker pull index.docker.io/caicloud/zeppelin:0.5.6
```
2. 从镜像仓库pull：
a)  才云科技的镜像仓库（index.caicloud.io）
```
docker login index.caicloud.io
docker pull index.caicloud.io/caicloud/spark:1.5.2
docker pull index.caicloud.io/caicloud/zeppelin:0.5.6
```
b) DockerHub的镜像仓库（index.docker.io）
```
docker login index.docker.io
docker pull index.docker.io/caicloud/spark:1.5.2
docker pull index.docker.io/caicloud/zeppelin:0.5.6
```
二、在Kubernetes上创建Spark集群

首先下载创建Kubernetes应用所需的Yaml文件：https://github.com/caicloud/public/tree/master/spark

第一步：创建命名空间namespace

  Kubernetes通过命名空间，将底层的物理资源划分成若干个逻辑的“分区”，而后续所有的应用、容器都是被部署在一个具体的命名空间里。每个命名空间可以设置独立的资源配额，保证不同命名空间中的应用不会相互抢占资源。此外，命名空间对命名域实现了隔离，因此两个不同命名空间里的应用可以起同样的名字。创建命名空间需要编写一个yaml文件：

```
#namespace/namespace-spark-cluster.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: "spark-cluster"
  labels:
    name: "spark-cluster"
```
 - kubectl create -f namespace/namespace-spark-cluster.yaml
 - kubectl get ns

使用Namespace：(${CLUSTER_NAME}和{USER_NAME}可在kubeconfig文件中查看
 - kubectl config set-context spark --namespace=spark-cluster --cluster=${CLUSTER_NAME} --name=${USER_NAME}
 - kubectl config use-context spark

第二步：启动master服务

  先创建Spark-master的ReplicationController，然后创建spark所提供的两个Service(spark-master-service，spark-webui)。让Spark-workers使用spark-master-service来连接Spark-master，并且通过spark-webui来查看集群和任务运行状态。

spark-master-controller.yaml可参考如下

kubectl create -f replication-controller/spark-master-controller.yaml

2) 创建Master-Service：

  Kubernetes追求以服务为中心，并推荐为系统中的应用创建对应的Service。以nginx应用为例，当通过Replication Controller创建了多个nginx的实例（容器）后，这些不同的实例可能运行在不同的节点上，并且随着故障和自动修复，其IP可能会动态变化。为了保证其他应用可以稳定地访问到nginx服务，我们可以通过编写yaml文件为nginx创建一个Service，并指定该Service的名称（如nginx-service）；

此时，Kubernetes会自动在其内部一个DNS系统中（基于SkyDNS 和etcd实现）为其添加一个A Record, 名字就是 “nginx-service”。随后，其他的应用可以通过 nginx-service来自动寻址到nginx的一个实例（用户可以配置负载均衡策略）。
  spark-master-service.yaml可参考如下：

kubectl create -f service/spark-master-service.yaml

3) 创建WebUI-Service：

  如上所述，Service会被映射到后端的实际容器应用上，而这个映射是通过Kubernetes的标签以及Service的标签选择器实现的。例如我们可以通过如下的spark-web-ui.yaml来创建一个WebUI的Service, 而这个Service会通过 “selector: component: spark-master”来把WebUI的实际业务映射到master节点上。spark-webui.yaml可参考如下：

kubectl create -f service/spark-webui.yaml

创建完ReplicationController(rc)、Service(svc)后，检查 Master 是否能运行和访问：
```
kubectl get rc
kubectl get svc
kubectl get pods
```
确认master正常运行后，再使用Kubernetes proxy连接Spark WebUI：

kubectl proxy --port=8001

然后通过http://localhost:8001/api/v1/proxy/namespaces/spark-cluster/services/spark-webui/查看spark的任务运行状态。

第三步：启动 Spark workers

  Spark workers 启动时需要 Master service处于运行状态，我们可以通过修改replicas来设定worker数目（比如设定 replicas: 4，即可建立4个Spark Worker）。注意我们可以为每一个worker节点设置了CPU和内存的配额，保证Spark的worker应用不会过度抢占集群中其他应用的资源。

spark-worker-controller.yaml可参考如下：

kubectl create -f replication-controller/spark-worker-controller.yaml

查看 workers是否正常运行

 1.通过WebUI查看： worker就绪后应该出现在UI中。（这可能需要一些时间来拉取镜像并启动pods）

 2.通过kubectl查询状态（可看到spark-worker都已经正常运行）：

第四步：提交Spark任务

两种方式：Spark-client或Zeppelin。

 1 通过Spark-client，可以利用spark-submit来提交复杂的Python脚本、Java/Scala的jar包代码；
 2 通过Zeppelin，可以直接在命令行或UI编写简单的spark代码。

1.通过Spark-client提交任务

spark-client.jpg

然后选择一个worker pod，使用exec方式进入pod，再通过spark-submit提交任务：

spark-commit.jpg

2.通过Zeppelin提交任务
  使用Zeppelin提交时有两种方式：pod exec 和 zeppelin-UI。我们先创建zeppelin的ReplicationController：

zeppelin.jpg

a) 通过zeppelin exec pods方式提交
$ kubectl exec zeppelin-controller-5g25x -it pyspark

zeppelin2.jpg

b) 通过zeppelin UI方式提交
使用已创建的Zeppelin pod，设置WebUI的映射端口：



    














