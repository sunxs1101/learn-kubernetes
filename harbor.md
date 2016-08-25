## harbor

Harbor是企业级registry server，用于存储和分发Docker images。Harbor通过添加企业级功能需求，比如安全/认证/管理等，来扩展了Docker
版本。作为一个企业级private registry，Harbor支持多registry的建立，并且每个registry会有镜像复制。
有了Harbor，镜像就会被存储在private registry，此外，Harbor提供高级安全特性，比如用户管理，访问控制和activity auditing。

Harbor可以通过docker-compose部署，可以离线部署，下面分别介绍这两种方式
### Install via docker compose
```
 - $ git clone https://github.com/vmware/harbor
 - 在配置文件Deploy/harbor.cfg中配置hostname,admin密码和邮箱等信息
 - $ cd Deploy
 - $ ./prepare
 - $ docker-compose up -d
 - 安装完成后，就可以访问web-UI
```
需要安装python, docker和[docker compose](https://docs.docker.com/compose/install/).

在202上新建Dockerfile文件，其中加载了python镜像，docker build -t harbor . 出现下面的问题

```
Step 9 : RUN docker-compose up -d
 ---> Running in b83cf11ecb74
Couldn't connect to Docker daemon. You might need to install Docker:

https://docs.docker.com/engine/installation/
The command '/bin/sh -c docker-compose up -d' returned a non-zero code: 1
```
需要docker和ubuntu，但是在
Dockerfile中可以用多个FROM创建多个镜像，但这个Dockerfile创建的容器，只有最后一个FROM起作用？

### Install via offline installer

参考https://github.com/vmware/harbor/blob/master/docs/installation_guide.md

The target host requires Python, Docker, and Docker Compose to be installed. 


### k8s安装harbor

参考才云科技的ppt，

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

