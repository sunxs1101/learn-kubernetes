```
#!/bin/bash

# References:
#  1. https://github.com/k8sp/tls/blob/master/bidirectional/create_tls_asserts.bash

# 1. 创建我们自己CA的私钥：

openssl genrsa -out ca.key 2048

# 创建我们自己CA的CSR，并且用自己的私钥自签署之，得到CA的身份证：

openssl req -x509 -new -nodes -key ca.key -days 10000 -out ca.crt -subj "/CN=we-as-ca"

# 2. 创建server的私钥，CSR，并且用CA的私钥自签署server的身份证：

openssl genrsa -out bootstrapper.key 2048
openssl req -new -key bootstrapper.key -out bootstrapper.csr -subj "/CN=bootstrapper"
openssl x509 -req -in bootstrapper.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out bootstrapper.crt -days 365
```


# 创建私有domain registry

将10.10.10.202作为domain registry server，设置domain registry名为10.10.10.202，创建过程如下
 -  在/etc/ssl/openssl.cnf中的v3_ca部分添加subjectAltName = IP:10.10.10.202
```
[ v3_ca ]
subjectAltName = IP:10.10.10.202
```
 - 生成TLS证书

为了让registry被集群中其他机器可用，Docker engine需要用TLS来认证，证书生成操作如下
```
mkdir -p certs && openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key -x509 -days 365 -out certs/domain.crt  
```
此过程中会提示有一些选项需要自定义输入
 - 让每个docker daemon信赖证书
```
mkdir /etc/docker/certs.d/10.10.10.202:5000
cp domain.crt /etc/docker/certs.d/10.10.10.202:5000/ca.crt 
```
Registry在/etc/docker/certs.d/10.10.10.202:5000/路径查找TLS证书
 - 重启engine deamon:  `systemctl restart docker`
 - 启动registry:2
```
docker run -d -p 5000:5000 --restart=always --name registry \
  -v `pwd`/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  registry:2
```
 - 测试registry

```
docker pull ubuntu
docker tag ubuntu 10.10.10.201:5000/ubuntu
docker push 10.10.10.201:5000/ubuntu

core@00-25-90-c0-f6-ee ~/certs $ docker push 10.10.10.201:5000/ubuntu
The push refers to a repository [10.10.10.201:5000/ubuntu]
unable to ping registry endpoint https://10.10.10.201:5000/v0/
v2 ping attempt failed with error: Get https://10.10.10.201:5000/v2/: dial tcp 10.10.10.201:5000: getsockopt: connection refused
 v1 ping attempt failed with error: Get https://10.10.10.201:5000/v1/_ping: dial tcp 10.10.10.201:5000: getsockopt: connection refused
```
这样的registry是不安全的，需要添加认证
 - 为Registry添加认证

为了限制其他用户对registry的访问，增加registry的安全性，需要为registry添加认证，操作如下
```
mkdir auth
docker run --entrypoint htpasswd registry:2 -Bbn testuser testpassword > auth/htpasswd
```
用户名设置为testuser，密码设置为testpassword，这两个都可以自己设定。关闭前面步骤中的registry，按下面操作重新启动registry
```
docker run -d -p 5000:5000 --restart=always --name registry \
  -v `pwd`/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  -v `pwd`/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  registry:2
```
 - 本机对domain registry的访问测试

在202上登录domain registry名为10.10.10.202的Registry，需要输入registry认证的用户名密码和生成TLS时设置的邮箱。登录后push镜像，如下
```
docker login 10.10.10.202:5000
docker tag ubuntu 10.10.10.202:5000/ubuntu
docker push 10.10.10.202:5000/ubuntu
```
 - 集群中其他机器对domain registry的访问测试

首先从202机器上的拷贝domain.crt到集群中某机器的相应路径
```
cp domain.crt /etc/docker/certs.d/10.10.10.202:5000/ca.crt
```
然后登录Registry进行push/pull操作
```
docker login 10.10.10.202:5000
docker push 10.10.10.202:5000/ubuntu
```
本例中domain registry名是registry server的IP，如果domain registry名不是registry server的IP，需要在/etc/hosts中添加registry server的IP与主机名，将IP映射到domain registry名。
```
vi /etc/hosts
添加10.10.10.202 myregistrydomain.com
```
### 参考
 - https://docs.docker.com/registry/deploying/#/let-s-encrypt
 - https://docs.docker.com/registry/insecure/


## wangyi pr
review https://github.com/k8sp/auto-install/pull/126

https://github.com/k8sp/auto-install/tree/6edee7e63eecc796276ad5da89f028f0774333fb/bsroot

### bsroot

bootstrapper镜像包括dnsmasq, cloud-config-server和Docker registry。bsroot的想法是把这些服务需要的数据文件和配置文件
都放入宿主机的一个目录，目录会被挂载到bootstrapper容器中的/bsroot路径。

### 目录结构

 - /bsroot/dnsmasq.conf：这个是dnsmasq的配置文件，它通过github.com/k8sp/auto-install/bootstrapper项目从描述集群的cluster-desc.yml
 生成。
 - /bsroot/registry.yml：这个是Docker registry配置文件，它通过github.com/k8sp/auto-install/bootstrapper项目生成。
 - /bsroot/registry/：这个目录被作为registry volume挂载到bootstrapper容器，由运行在bootstrapper容器中的Docker registry service生成。
 这个目录在/bsroot/registry.yml中被提及。
 
### Build and Run

实际的bootstrapper镜像会包含很多服务，我写了一个Dockerfile来描述Docker registry。从Dockerfile中建立镜像
  `docker build -t registry -f registry.Dockerfile .`
运行bootstrapper，命名为registry
  `docker run -d --privileged -p 5000:5000 --name registry -v $(pwd)/bsroot:/bsroot registry`
然后就可以将一个镜像push进去
```
docker tag hello-world bootstrapper:5000/hello
docker push bootstrapper:5000/hello
```
### TLS和Docker Registry

为了使Docker registry被任意地点的机器可用，我们需要建立TLS，basic idea包括

 1. 生成self-signed CA key/certificate pair. 我们利用self-signed CA为bootstrapper server生成certificate，假定bootstrapper服务器
 可用通过bootstrapper主机名访问。运行`bsroot/gen-ca-and-bootstrapper-certs.bash`会生成。
 
 2. 让Docker registry服务使用bootstrapper证书
 
 挂载bsroot/到boostrapper容器的/bsroot路径，让docker registry服务加载/bsroot/bootstrapper.{crt,key}，详细参考registry.Dockerfile
 
 3. 把self-signed CA证书安装到集群的每个节点，这让每个节点可用访问运行在bootstrapper server中的registry.

  - CoreOS参考https://coreos.com/os/docs/latest/adding-certificate-authorities.html
  - Ubuntu，步骤如下：

```
sudo cp ca.crt /usr/share/ca-certificates/
sudo dpkg-reconfigure ca-certificates
sudo update-ca-certificates
```

### VM-based Testing

为了测试上述内容，我新建vm-cluster/Vagrantfile, 用vagrant up创建两个VMs: bootstrapper和node。确保

 1. 每个VM有两个NIC，eth0用户与主机通信，eth1连接到相同的虚拟局域网vboxnet6
 2. 两个VM都有安装了Docker
 3. 两个VM都添加到/etc/hosts，这样他们都可以用bootstrapper主机名访问bootstrapper VM
 4. 主机上创建~/work共享目录，挂载到两个虚拟机的/work目录
 
### Build and Test Bootstrapper

获取源代码，为bootstrapper生成TLS assets.
```
host $ mkdir ~/work
host $ export GOPATH=$HOME/work
host $ go get github.com/k8sp/auto-install
host $ cd $GOPATH/src/github.com/k8sp/auto-install/bsroot/bsroot
host $ ./gen-ca-and-bootstrapper-certs.bash
```
最后的命令生成ca.{crt,key}和bootstrapper.{crt,key}
然后我们在host上启动VMs
```
host $ cd $GOPATH/src/github.com/k8sp/auto-install/bsroot/vm-cluster
host $ vagrant up
```
这会将host上的~/work路径挂载到两个VMs的/work路径，在bootstrapper VM中创建和运行bootstrapper Docker image。
```
host $ vagrant ssh bootstrapper
bootstrapper $ cd /work/src/github.com/k8sp/auto-install/bsroot
bootstrapper $ docker build -t bootstrapper -f registry.Dockerfile .
bootstrapper $ docker run -d --privileged -p 5000:5000 --name registry -v $(pwd)/bsroot:/bsroot registry
```
登陆node VM，测试Docker registry
```
host $ vagrant ssh node
node $ docker pull hello-world
node $ docker tag hello-world bootstrapper:5000/hello
node $ docker push bootstrapper:5000/hello
```
Trouble shooting


## coreos CA

coreos添加CA, https://coreos.com/os/docs/latest/adding-certificate-authorities.html

Use self-signed CA to [encrypt communications with a container registry](https://coreos.com/os/docs/latest/registry-authentication.html) , 




