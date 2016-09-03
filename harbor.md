## harbor

Harbor是企业级registry server，用于存储和分发Docker images。Harbor通过添加企业级功能需求，比如安全/认证/管理等，来扩展了Docker
版本。作为一个企业级private registry，Harbor支持多registry的建立，并且每个registry会有镜像复制。
有了Harbor，镜像就会被存储在private registry，此外，Harbor提供高级安全特性，比如用户管理，访问控制和activity auditing。

Harbor管理员有自己的用户名和密码

Harbor可以通过docker-compose部署，可以离线部署，下面介绍docker-compose部署。
### Install via docker compose
```
 - $ git clone https://github.com/vmware/harbor
 - 在配置文件Deploy/harbor.cfg中配置hostname,admin密码和邮箱等信息
 - $ cd Deploy
 - $ ./prepare
 - $ docker-compose up -d
 - 安装完成后，就可以访问web-UI
```
在上述操作过程中需要先安装python, docker和[docker compose](https://docs.docker.com/compose/install/).
以docker的方式运行，需要在Dockerfile中有python,docker,docker-compose。Dockerfile中可以用多个FROM创建多个镜像，但这个Dockerfile创建的容器，只有最后一个FROM起作用，因此以Ubuntu作为base image，然后安装python,docker,docker-compose
在202上新建Dockerfile文件，其中加载了python镜像，docker build -t harbor . 出现下面的问题

```
Step 9 : RUN docker-compose up -d
 ---> Running in b83cf11ecb74
Couldn't connect to Docker daemon. You might need to install Docker:

https://docs.docker.com/engine/installation/
The command '/bin/sh -c docker-compose up -d' returned a non-zero code: 1
```
#### Configuring Harbor
在harbor.cfg中配置参数

 - hostname: 目标主机的hostname，应该是IP或者fully qualified domain name.
 - ui_url_protocol: 

### Install via offline installer

参考https://github.com/vmware/harbor/blob/master/docs/installation_guide.md

The target host requires Python, Docker, and Docker Compose to be installed. 

## k8s中运行harbor


配置参考才云科技的ppt和https://github.com/vmware/harbor/blob/master/docs/kubernetes_deployment.md#deploying-harbor-on-kubernetes ,链接中有四个yaml文件。

registry config文件需要有registry的IP(或DNS名)，但是在service创建之前不知道IP，有很多workarounds来解决这个额外难题：

 - 使用DNS名，service创建后，将DNS名与IP关联

参考： 

 - service创建后，用service IP重新build registry image，用kubectl rolling-update命令更新到新的image。

```
Deploy/kubernetes/mysql-rc.yaml
Deploy/kubernetes/proxy-rc.yaml
Deploy/kubernetes/registry-rc.yaml
Deploy/kubernetes/ui-rc.yaml
文件说明：

```
 - Step 1:容器化配置文件
```
cp /home/core/harbor/harbor/Deploy/kubernetes/dockerfiles/registry-dockerfile /home/core/harbor/harbor/Deploy/Dockerfile
docker build -t="harbork8s" .
```
 - Step 2:动态配置文件
```
./kubernetes/ui-rc.yaml
```
 - Step 3:
```
./kubernetes/dockerfiles/registry-config.yml
```
 - Step 4:



