## docker学习
 - docker run [options] IMAGE [command]
``` 
docker run用来基于特定的镜像创建一个容器，如docker run ubuntu echo "hello world"
sudo docker run -i -t --name mytest ubuntu:latest /bin/bash
-i表示使用交互模式
-t表示分配一个伪终端
--name可以指定docker run命令启动容器的名词，无此选项，Docker将为容器随机分配一个名字
在镜像名后添加tag来区分同名的镜像，如ubuntu:14.04，默认选取tag为latest的镜像。
```

### docker镜像关键概念
```
registry:保存docker镜像，分公共和私有两种。默认的registry是Docker hub，Docker hub中有两种类型的仓库：
用户仓库(user registry)和顶层仓库(top-level repository)。用户仓库的镜像都是由Docker用户创建的，其命名由用户名和仓库名
两部分组成，如jamtur01/puppet。与之相对，顶层仓库只包含仓库名部分，如ubuntu仓库。


repository:具有某个功能的Docker镜像的所有迭代版本构成的镜像库，registry由一系列经过命名的repository组成。
registry是repository的集合，repository是镜像集合
```
### 基于Dockerfile构建新镜像

```
sudo docker build -t="jamtur01/static_web:v1" .
-t选项为新镜像设置仓库和名称，jamtur01为仓库，static_web为镜像名，v1为标签。后面的.告诉Docker到本地目录去找Dockerfile文件。
也可以制定一个Git仓库的源地址来制定Dockerfile的位置，如下
sudo docker build -t="jamtur01/static_web:v1" \git@github.com:jamtur01/docker-static_web
```
构建上下文

### 从镜像启动容器

```
sudo docker run -d -p 80 --name static_web jamtur01/static_web \ nginx -g "daemon off;"
-d:在后台跑
--name:容器名为static_web
-p:控制Docker在运行时应该公开哪些网络端口给外部(宿主机)，运行一个容器时，Docker通过两种方法来在宿主机上分配端口。
 - Docker可以在宿主机上随机选择一个位于49000～49900的一个比较大的端口来映射到容器中的80端口上。
 - 可以在Docker宿主机中制定一个具体的端口号来映射到容器中的80端口上。
使用"docker ps "命令查看容器的端口分配情况，"docker port CONTAINERID port"查看端口映射情况。
```
### Dockerfile指令

 - CMD
 - ENTRYPOINT
 - WORKDIR
 - ENV
 - USER
 - VOLUME
 - ADD
 - COPY
 - ONBUILD

## 运行自己的Docker Registry

流程：利用dockerfile创建镜像->启动registry，搭建localhost registry->利用证书，搭建domain registry
有时我们希望构建和存储包含不想被公开的信息或数据的镜像，有两个选择
 - 利用Docker Hub上的私有仓库
 - 在防火墙后面运行你自己的Registry

如何搭建私有仓库
```
1. 在localhost搭建私有仓库
从Docker容器安装一个Registry很简单，
sudo docker run -p 5000:5000 registry
将本地镜像打完标签，上传push到新的Registry
docker push 

2. Running a domain registry
为了使得registry更广泛可用，Docker利用TLS证书来保证其安全，这在概念上和用SSL配置web服务器类似。
2.1 get a certificate
假定你有一个域名myregistrydomain.com，它的DNS记载指向运行registry的主机，首先要从CA获取证书

1) Generate your own certification
mkdir -p certs && openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key -x509 -days 365 -out certs/domain.crt
2) Be sure to use the name myregistrydomain.com as a CN
3) Use the result to start your registry with TLS enabled
4) 令每个docker守护进程信赖证书，执行如下操作
拷贝domain.crt到/etc/docker/certs.d/myregistrydomain.com:5000/ca.crt
5) 重启后台进程

TLS使能的情况下，开启registy
docker run -d -p 5000:5000 --restart=always --name registry \
  -v `pwd`/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  registry:2

现在可以从其他docker主机上连接你的registry
docker pull ubuntu
docker tag ubuntu myregistrydomain.com:5000/ubuntu
docker push myregistrydomain.com:5000/ubuntu
docker pull myregistrydomain.com:5000/ubuntu



```
#### Override specific configuration options

通过docker run中-e选项或者在Dockerfile中使用ENV指令来指定配置变量。


## kubernetes学习

pod可以想象成一个篮子，而容器则是篮子里的鸡蛋。

## docker部署
docker的设计理念是希望用户能够保证一个容器只运行一个进程，即只提供一种服务。
然而，单一容器是无法满足需求的，通常用户需要利用多个容器，分别提供不同的服务，并在不同容器间互连通信，最后形成一个docker集群。

docker run命令的--link选项建立容器间的互联关系。



##使用docker镜像和仓库
 - sudo docker images列出镜像，镜像保存在仓库中，仓库存在Registry中，默认的Registry是由docker公司运营的公共Registry服务，即Docker Hub。
 
用docker run从镜像启动一个容器时，如果该镜像不在本地，Docker会先从Docker Hub下载该镜像，下载latest的。
  - 查找镜像sudo docker search puppet
  
### 原来ubuntu 14.10 utopic不能安装docker的问题
docker不支持14.10utopic，一般安装utrusty。
在/etc/apt/source.list中做替换操作。
 1. sudo sh -c "echo deb https://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list"
 2. whereis curl
 3. 1722  curl: /usr/bin/curl /usr/bin/X11/curl /usr/share/man/man1/curl.1.gz
 4. 1723  sudo apt-get -y install curl
 5. 1724  sudo apt-get -y install
 6. 1725  sudo apt-get -y install -f
 7. 1726  sudo apt-get -y install -f --force-yes
 8. 1727  curl -s https://get.docker.io/gpg | sudo apt-key add -
 9. 1728  uname -a
 10. 1729  sudo apt-get update
 11. 1730  ls -l /sys/class/misc/device-mapper
 12. 1731  whereis curl
 13. 1732  sudo apt-get -y install curl
 14. 1733  sudo apt-get install lxc-docker
 15. 1734  sudo apt-get -f install lxc-docker
 16. 1735  sudo apt-get -f install
 17. 1736  sudo apt-get -f install docker.io
 18. 1737  vi helloworld.cpp
 19. 1738  sudo apt-get -f install emacs24

http://mirrors.aliyun.com/help/ubuntu
