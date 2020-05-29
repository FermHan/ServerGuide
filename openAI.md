本文教程地址：

- github：
- CSDN：https://blog.csdn.net/hancoder/article/details/106423288

https://cr.console.aliyun.com/cn-qingdao/instances/repositories

# 一、理论组集群使用方法

### 0 账号信息：

- 平台账号：(密)
  - 管理员(密)：用户名admin 密码admin-password
  - 现有普通用户：用户名test1  密码123123
- nfs存储服务器：222.195.151.85:8011( 192.168.1.4)  用户名ouc  密码123
- 集群：用户名ouc  密码123
  - 192.168.1.200 - 206。其中200为master n0，其余为n1-n6

### 1 登录集群平台

浏览器输入：222.195.151.231 ，输入账号密码。账号密码分为管理员账户和普通账户。

- 管理员账户用于注册普通账户和查看集群运行情况。
- 普通账户用于添加训练任务

管理员账户如何添加普通账户：(需在管理员账户下进行)

点击左侧Administration/User Management，点击右侧Add User，填入Name和Password。然后退出管理员账户后，重启登录普通用户即可



### 2 传数据

把数据从个人电脑上传到集群的fs存储服务器上：

用ssh/xftp工具登录存储服务器。

创建自己的文件夹

```sh
# 创建个人文件夹(一定要创建在data目录下)
mkdir /data/姓名全字母
```

将上传个人文件到`/data/YOURNAME`目录下

注：

> #在文件服务器下/etc/exports有如下内容： /data/ 192.168.1.0/24(rw,sync,fsid=0)
> 表示只允许把存储服务器/data目录下的内容挂载到192.168.1.X的网段的/mnt目录上。所以把个人文件放到其他地方是挂载不上的。
>
> 千万不要在存储服务器的ssh里挂载，在存储服务器挂载的/mnt在docker容器里是读不到的。如果不小心在存储服务器里使用了挂载命令，请卸载
>
> cd /
> sudo umount /mnt



### 3 编写进入docker后的命令

点击左侧Submit Job，在右侧的Task_role_1的Command栏中填入命令

```sh
apt update # 更新原
# 安装nfs客户端，以便下一步挂载存储(保证每个节点服务器安装过一次即可)
apt install -y nfs-common 

# 挂载 #(从存储服务器挂载到docker容器里)
mount -o nolock -t nfs 192.168.1.4:/data/姓名 /mnt
##挂载完后你的文件传到了/mnt里，
##比如原来文件地址：/data/hanfeng/test.sh，现在地址为：/mnt/test.sh。
##注意没有了姓名

# cd找你的个人文件，
cd /mnt
# 修改python环境 
# 如pip pip install tensorflow-gpu=1.12

# 执行训练任务 # 尽量使用绝对路径，注意是没有你姓名那层目录的
python /mnt/test.py

# 现有python环境是docker容器的python，如果要切换python版本请百度如何卸载python，安装module的原理也一样。
```

```sh
#测试环境
apt update
apt install -y nfs-common 
mount -o nolock -t nfs 192.168.1.4:/data/module /mnt
#/data/models/research/slim/download_and_convert_data.py 
# ===> /mnt/research/slim/download_and_convert_data.py
python /mnt/research/slim/download_and_convert_data.py --dataset_name=cifar10 --dataset_dir=/tmp/data

python /mnt/research/slim/train_image_classifier.py --dataset_name=cifar10 --dataset_dir=/tmp/data --max_number_of_steps=1000 --batch_size 64
```



### 4 提交任务

![](https://fermhan.oss-cn-qingdao.aliyuncs.com/img/20200529172131.png)

### 5 自定义制作docker镜像

如第4部分我们可以选择现有的docker镜像创建容器，也可以自己制作docker镜像。

请先阅读`二、docker基础`章节

制作镜像尽量使用一定的命名规则。

现有镜像是通过阿里云仓库制作的，我们在阿里云仓库中也有一部分制作好的镜像，可以pull。

```sh
# 从我的仓库中拉取镜像
sudo docker pull registry.cn-qingdao.aliyuncs.com/oucvisionlab/docker-repository:[镜像版本号]
```

按照第二章节制作完镜像后，可以push到阿里云仓库中，然后复制镜像的网络地址，用于提交任务时指定。

下面命令为push到我的仓库的办法，但尽量不要push到我的仓库(我会定期清理我的仓库)，尽量按照5.1章节的办法自己创建阿里云仓库，然后push到你自己的阿里云仓库里。然后按照我下面的方法进行push

```sh
# 1.登录仓库
cvlab.qdxnzn.com/123/ubuntu 
docker login --username=admin cvlab.qdxnzn.com/
#输入密码 ouc123456

#2. 将镜像推送到Registry,需要先登录完
## 打标签
docker tag [ImageId] cvlab.qdxnzn.com/123:[镜像版本号]
## push

docker push cvlab.qdxnzn.com/123:[镜像版本号]
docker push registry.cn-qingdao.aliyuncs.com/oucvisionlab/docker-repository:[镜像版本号]
```

#### push

> docker pull cvlab.qdxnzn.com/123/ubuntu:16.04

https://cvlab.qdxnzn.com



# 二、docker基础

### 1 docker概念

镜像images相当于模板，容器container相当于模板实例化出来的实例。

镜像存储分为本地仓库和远程仓库。仓库中只能存放镜像，不能存放容器。要上传到仓库需要先把容器commit为镜像。

每个镜像有3个主要属性：所在仓库REPOSITORY、标签 TAG、ID号IMAGE ID。

ID可以唯一标识一个镜像，REPOSITORY:TAG也能唯一标识一个镜像。

### 2 docker基本命令

```sh
#查看镜像，会显示IMAGE_ID、REPOSITORY、TAG
docker images 

# 查看容器
docker ps 

# 创建容器 (以镜像为模板) # 进入后命令行显示[root@容器id]  
docker run  -it 【REPOSITORY:TAG】 /bin/bash
# 如果有映射端口需要，可以使用下面命令，将本地8001端口映射到
docker run -d -p 本地端口:容器内端口  【REPOSITORY:TAG】 

# 退出容器
exit 或Ctrl+D 

# 容器形成镜像
docker commit [OPTIONS] CONTAINER_ID [REPOSITORY[:TAG]]
# 为方便使用，我们要求把TAG以其容器中软件版本命名，越详细越好，方便我们分辨
#如nocuda-python6-tf1.12。这样REPOSITORY[:TAG]表示为oucvisionlab/docker-repository:

# 打标签，相当于复制且重命名镜像，方便指定上传的仓库
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]

# 进入运行中的容器
docker exec -it 容器ID  /bin/bash

#根据Dockerfile创建镜像
docker build [OPTIONS] PATH | URL | -
docker build -t 机构/镜像名<:tags> 生成Dockerfile目录

# 删除镜像
docker image rm 

```

#### push与pull

```sh
# 1. 登录阿里云Docker Registry 
docker login --username=韩锋626 registry.cn-qingdao.aliyuncs.com
#输入密码 visionlab2020
## username中有中文是若干年前创建阿里云账号的时候不小心舔的中文,更改不了，所以

#2. 将镜像推送到Registry,需要先登录完
## 打标签
docker tag [ImageId] registry.cn-qingdao.aliyuncs.com/oucvisionlab/docker-repository:[镜像版本号]
## push
docker push registry.cn-qingdao.aliyuncs.com/oucvisionlab/docker-repository:[镜像版本号]



# 3. 从Registry中拉取镜像
sudo docker pull registry.cn-qingdao.aliyuncs.com/oucvisionlab/docker-repository:[镜像版本号]
```



### 3 Dockerfile

Dockerfile镜像描述文件

- 是一个包含组合镜像的命令的文本文档
- Docker通过读取Dockerfile中的指令按步自动生成镜像
- 把命令写在Dockerfile中，且Dockerfile必须命名为Dockerfile

基本命令：

```sh
FROM 【基准镜像】#基准镜像
FROM scratch # 不依赖任何镜像标准
MAINTAINER # 说明信息
# LABEL version= # 版本信息 
LABEL description =

# cd
WORKDIR  

# 复制、添加远程文件
ADD 【本地文件】  【镜像目录】

# 覆盖环境变量
ENV  JAVA_HOME /usr/local/openjdk8 

# 执行sh命令
## 方式1：run后直接加命令
RUN ${JAVA_HOME}/bin/jva -jhar test.jar
## 方式2：用数组的方式分隔空格
RUN["echo","123"]

# 暴露端口
EXPOSE 7000 
```

3种执行命令的方法：

```sh
# 1 RUN方式：在docker build构建时执行命令
RUN yum install -y vim # Shell命令格式
RUN ["yum","install","-y","vim"] # Exec命令格式，推荐

## shell与Exec命令格式区别：
###shell方式
#### 使用shell执行时，当前shell是父进程，生成一个子shell进程
#### 在子shell进程执行脚本，脚本执行完毕，退出子shell，回到当前shell
###使用Exec方式
#### Exec方式会用Exec进程替换当前进程，并且保持PID不变。执行完毕，直接退出，并不会退出之前的父进程环境

# 2 ENTRYPOINT方式：在docker run构建容器 容器启动时执行的命令  
## 且Dockerfile中只有最后一个ENTRYPOINT会被执行
ENTRYPOINT ["ps"] #推荐使用Exec格式

# 3 CMD方式：容器启动后执行默认的命令或参数
## 且Dockerfile中只有最后一个CMD会被执行
## 如容器启动时附加指令，则CMD被忽略  docker run ... ls
CMD ["ps","-ef"] #推荐使用Exec格式

# 组合使用方法：
ENTRYPOINT可以和CMD组合使用  如：
ENTRYPOINT ["ps"]
CMD ["-ef"]
这样我们就可以在build的时候添加另外的参数如-ef|grep ssh，这样就覆盖了CMD
```







### 构建cuda镜像

```sh
# 安装vim
apt-get install vim

#备份源
cp  /etc/apt/sources.list  /etc/apt/sources.list.backup
#更新源
vim /etc/apt/resources.list
替换为：
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse

保存退出


apt update
apt-get update

# ========================
# tools
# ------------------------
apt-get install -y --no-install-recommends \
		build-essential \
        apt-utils \
        ca-certificates \
        wget \
        git \
        nano \
        vim \
        libssl-dev \
        curl \
        openssh-server \
        unzip \
        unrar \
      
#make
git clone --depth 10 https://github.com/Kitware/CMake ~/cmake && \
    cd ~/cmake && \
    ./bootstrap && \
    make -j"$(nproc)" install 
    
#安装python
RUN apt-get update && \
apt-get install -y libreadline-dev && \ 
wget https://www.python.org/ftp/python/3.6.0/Python-3.6.0.tgz && \ 
tar -xvf Python-3.6.0.tgz && \ 
cd Python-3.6.0 && \ 
./configure && \ 
make && \ 
make install && \
ln -s /usr/local/bin/python3.6 /usr/local/bin/python && \
ln -s /usr/local/bin/python3.6-config  /usr/local/bin/python-config

RUN pip3 install --upgrade pip && \
    pip3 install --upgrade setuptools
    
RUN pip3 install tensorflow-gpu==1.12.0 -i https://pypi.tuna.tsinghua.edu.cn/simple


# ==================================================================
# config & cleanup
# ------------------------------------------------------------------

RUN   ldconfig && \
    apt-get clean && \
    apt-get autoremove && \
    rm -rf /var/lib/apt/lists/* /tmp/* ~/*  && \
    rm -R /Python-3.6.0 && \
    rm /Python-3.6.0.tgz


EXPOSE 8888 6006
```

cuda安装

```sh
wget http://developer.download.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda_10.1.243_418.87.00_linux.run
sudo sh cuda_10.1.243_418.87.00_linux.run

wget https://developer.nvidia.com/compute/cuda/10.0/Prod/local_installers/cuda_10.0.130_410.48_linux
sudo sh cuda_10.0.130_410.48_linux.run
```

更换pip源

# 三 阿里云仓库

#### 阿里云仓库

浏览器输入https://cr.console.aliyun.com/cn-qingdao/instances/repositories

创建命名空间

![](https://fermhan.oss-cn-qingdao.aliyuncs.com/img/20200529174457.png)

创建仓库：

![](https://fermhan.oss-cn-qingdao.aliyuncs.com/img/20200529195534.png)

然后点击创建好的仓库，在页面下面就提示了你怎么使用你的阿里云仓库。

创建完阿里云仓库后重新看第5部分，push到你阿里云仓库即可。

```sh
# 1. 登录阿里云Docker Registry 
docker login --username=韩锋626 registry.cn-qingdao.aliyuncs.com
#输入密码 visionlab2020
## username中有中文是若干年前创建阿里云账号的时候不小心舔的中文,更改不了，所以

#2. 将镜像推送到Registry,需要先登录完
## 打标签
docker tag [ImageId] registry.cn-qingdao.aliyuncs.com/oucvisionlab/docker-repository:[镜像版本号]
## push
docker push registry.cn-qingdao.aliyuncs.com/oucvisionlab/docker-repository:[镜像版本号]
```

