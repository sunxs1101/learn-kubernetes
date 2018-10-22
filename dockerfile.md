```
FROM centos:centos7
WORKDIR /home
RUN yum update -y 
RUN yum install -y wget bzip2 \
	gcc gcc-c++ cmake make vim* gdb libboost-all-dev libbz2-dev zlib1g-dev \	
	valgrind \
    && yum clean all

RUN wget http://mirror.centos.org/centos/7/os/x86_64/Packages/boost-1.53.0-27.el7.x86_64.rpm && \
    yum -y install boost-1.53.0-27.el7.x86_64.rpm boost-devel && \
    rm boost-1.53.0-27.el7.x86_64.rpm

RUN wget http://10.210.208.36/rpms-ad/feed-test/base_release-3.0.0-3.x86_64.rpm && \
    yum -y install base_release-3.0.0-3.x86_64.rpm && \
    yum clean all && \
    rm base_release-3.0.0-3.x86_64.rpm


FROM ubuntu:18.04
WORKDIR /home
RUN apt-get update && \
    apt-get install -y wget alien \
	gcc-4.8 g++-4.8 cmake vim gdb libboost-all-dev libbz2-dev zlib1g-dev \	
	valgrind \
    && apt-get clean -y

RUN wget http://10.210.208.36/rpms-ad/feed-test/base_release-3.0.0-3.x86_64.rpm && \
    alien -d base_release-3.0.0-3.x86_64.rpm && \
	dpkg -i base-release_3.0.0-4_amd64.deb && \
	rm base-release_3.0.0-4_amd64.deb  base_release-3.0.0-3.x86_64.rpm

RUN wget http://10.210.208.36/rpms-ad/feed-test/boost_1_53_0.tar.gz && \
    tar -xzf boost_1_53_0.tar.gz && \
    cd boost_1_53_0 && \
    ./bootstrap.sh && \
    ./b2 install && \
    rm ../boost_1_53_0.tar.gz
``` 
