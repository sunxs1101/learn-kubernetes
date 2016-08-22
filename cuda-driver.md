## 安装cuda-driver

 - 方法 A：https://github.com/emergingstack/es-dev-stack
 - 方法 B：https://github.com/Clarifai/coreos-nvidia  此方法编译方便，推荐

https://github.com/k8sp/k8s-tensorflow/issues/7

仔细阅读了 @pineking 贴的两个链接。都很受启发：

1. http://blog.clarifai.com/how-to-simplify-building-nvidia-gpu-drivers-on-coreos

    我之前在Docker Container里build和执行nvidia driver，用的是这个[办法](https://github.com/emergingstack/es-dev-stack)。但是它引入了一些对CoreOS的假设条件。在其中的[一个issue里](https://github.com/emergingstack/es-dev-stack/issues/8)，大家讨论了用CoreOS为开发者提供的一个development Docker image来build，就可以不需要这些假设条件了。而 @pineking  贴的[这个方法](http://blog.clarifai.com/how-to-simplify-building-nvidia-gpu-drivers-on-coreos)对应的[Github repo](https://github.com/Clarifai/coreos-nvidia/blob/master/build.sh#L12)就用的是CoreOS的development container。

1. http://blog.clarifai.com/how-to-scale-your-gpu-cloud-infrastructure-with-kubernetes

    描述用 1. 中描述的方法，在Kubernetes on CoreOS机群里跑Tensorflow jobs。


## 方法 A

 - 安装CUDA镜像
 
 CoreOS内核中不包含CUDA GPU driver，我们要将CUDA GPU driver作为核模块，同tensorflow一块加载到Docker镜像中，建立Docker镜像需要
 大量的磁盘空间，这个因为要下载linux源代码和CUDA toolkit，将这两块放到主机目录，然后映射到VM。
 ```
 $ git clone http://github.com/emergingstack/es-dev-stack.git
 $ cd es-dev-stack/corenvidiadrivers
 $ docker build -t cuda .
 -t选项为新镜像设置仓库名和镜像名，也可以为镜像设置标签。
 
 ```
 
 
 insmod(install modules) 模块A：向内核加载模块A
 lsmod(list modules)：显示已经载入系统的模块
