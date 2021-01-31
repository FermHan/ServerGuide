本文教程地址：

- github：https://github.com/OUCvisionLab/ServerGuide/blob/master/openPAI.md
- CSDN：https://blog.csdn.net/hancoder/article/details/106423288
- linux基础使用可以参考：https://github.com/OUCvisionLab/ServerGuide/blob/master/README.md

因为校园网访问github图片经常不显示，css样式也比较少，推荐去csdn看，容易区分命令

> 后续管理员需要更新内容可以联系我，我在管理员教程里有写github的账号密码
>
> 有疑问可以评论留言

@[toc]

# 一、理论组集群使用方法

注：在本服务器中，docker命令前必须加sudo，本文没写sudo。正确命令：`sudo  docker ...`

> 没使用过docker的用户先下拉倒第二章1、2部分简单看一下pull、push

### 0 账号信息：

- openAI平台账号：(密)：向管理员申请账号
- nfs存储服务器：222.195.151.85:8011( 192.168.1.4)  用户名`ouc`  密码`123`
- 镜像仓库cvlab.qdxnzn.com：账号`admin` 密码`Harbor12345`。push密码也是`Harbor12345`
  - `sudo docker login --username=admin  cvlab.qdxnzn.com`   输入`Harbor12345`
- 集群(非管理员勿登录)：用户名`ouc`  密码`123`
  - 192.168.1.200 - 206。其中200为master n0，其余为n1-n6

### 1 登录集群平台

浏览器输入：222.195.151.231 ，输入账号密码。账号密码分为管理员账户和普通账户。

- 管理员账户用于注册普通账户和查看集群运行情况。
- 普通账户用于添加训练任务(保证外来人员无集群账号)

管理员账户如何添加普通账户：(需在管理员账户下进行)，点击左侧Administration/User Management，点击右侧Add User，填入Name和Password。然后退出管理员账户后，重启登录普通用户即可



### 2 传数据

把数据从个人电脑上传到集群的fs存储服务器上：222.195.151.85:8011( 192.168.1.4)  用户名ouc  密码123

window与linux互传用`ssh/xftp`工具(推荐MobaXterm软件)登录存储服务器。

>  如果是linux服务器间文件的互传可以看rsync命令https://blog.csdn.net/hancoder/article/details/113172109
>
> 里面的scp命令可以用于镜像里传数据集使用，可以写到网页中的命令中

登录文件服务器后，创建自己的文件夹

```bash
# 创建个人文件夹(一定要创建在data目录下，而且注意是 /data，不是home下的)
mkdir /data/姓名全字母

# 可以在文件服务器里对你的文件进行一些操作
# 过后把自己的data文件删了，因为数据集一般太占内存
```

将上传个人文件到`/data/YOURNAME`目录下。在后面的页面命令中，我们会让他挂载到镜像目录（如`/mnt`）下。

> 注：为什么不能放到别的目录下？
>
> 在文件服务器下`/etc/exports`有如下内容：` /data/ 192.168.1.0/24(rw,sync,fsid=0)`
> 表示只允许把存储服务器`/data`子目录的内容挂载到`192.168.1.X`的网段的目录上(镜像里的目录是任意的，如`/mnt`)。所以把个人文件放到文件服务器的除/data其他地方是挂载不上的。
>
> 千万不要在存储服务器的ssh里挂载，要在后面的网页命令里进行挂载，在存储服务器挂载的`/mnt`在docker容器里是读不到的。如果不小心在存储服务器里使用了挂载命令，请卸载`sudo umount /mnt`
>
> 另注：文件服务器是centos系统，要用yum install



### 3 docker的拉取与提交

在任务提交网页http://222.195.151.231/submit.html 的最下面有一个`Docker images`选项，

- 可以选择下拉列表的选项（未都测试过），
- 也可以选择`Custom`自定义按钮来上传镜像。填写的格式是docker的格式

> 下拉列表的选择完后可以看4小节了，选custom的接着往下看

##### 3.1 Custom

可以去如下docker hub链接里去搜`openpai`支持的docker镜像，这里提供几个常用链接

- https://hub.docker.com/r/openpai/standard/tags

- https://hub.docker.com/u/openpai

- 也可以去镜像仓库里找别人的镜像使用https://cvlab.qdxnzn.com/

- 尽量使用openpai官方提供的或在此基础上更改，不然大概率是openpai不支持的镜像。

- 制作openpai支持的镜像可以找个大概能用的镜像后修改python或者其他内容

- ```python
  # 更换python
  据我观察，openpai里一些tf和torch的镜像里面是继承的anaconda
  你可以找个有anaconda的镜像，然后在里面安装你所需要的包，虚拟镜像如何用可以看另外一个教程
  
  conda create -n py38 python=3.8
  which pip
  apt-get update
  apt-get install vim
  pip install -r requirements.txt
  ```

Custom自定义镜像填写格式：

- 简写方式：`ufoym/deepo:tensorflow-py36-cu90`  
  - 代表docker hub里`ufoym`这个用户的`deepo`项目（就是镜像名字），提交版本`tag`是`tensorflow-py36-cu90`
- 如果要用我们自己镜像仓库里的，加上com前缀与目录地址`cvlab.qdxnzn.com/目录`
  - 如`cvlab.qdxnzn.com/目录/镜像名字:镜像版本号`



### 4 编写进入docker后的命令

点击左侧`Submit Job`，在右侧的Task_role_1的`Command`栏中填入命令

```bash
echo "task start..."
# 防止镜像里没有挂载命令
apt install -y nfs-common
apt update # 更新源

# 挂载 #(把存储服务器/data 挂载到 docker容器/mnt)
mount -o nolock -t nfs 192.168.1.4:/data/姓名  /mnt
##挂载完后你的文件传到了/mnt里，
##比如原来文件地址：/data/hanfeng/test.sh，现在地址为：/mnt/test.sh。
##注意没有了姓名

# cd找你的个人文件，
cd /mnt
# 修改python环境
# source activate py38
# 如 pip install tensorflow-gpu=1.12

# 除挂载外，可以用scp命令传另外一些文件到镜像里
# 文件服务器外网ip端口：222.195.151.85:8011

# 执行训练任务 # 尽量使用绝对路径，注意是没有你姓名那层目录的
python /mnt/test.py

# 现有python环境是docker容器的python，如果要切换python版本请百度如何卸载python，安装module的原理也一样。
```

如下用法：

测试环境1

```bash
echo "task start..."
apt update
echo "task end..."

mount -t nfs -o rw,nolock 192.168.1.4:/data/models /mnt
```

测试环境2

```bash
echo "task start..."
apt update
apt install -y nfs-common
echo "task 1..."
mount -o nolock -t nfs 192.168.1.4:/data/models /mnt
echo "task 2..."
python /mnt/research/slim/download_and_convert_data.py --dataset_name=cifar10 --dataset_dir=/tmp/data
echo "task 3..."
python /mnt/research/slim/train_image_classifier.py --dataset_name=cifar10 --dataset_dir=/tmp/data --max_number_of_steps=1000 --batch_size 64


# 存储服务器和docker容器的文件对应关系如下：
# /data/models/research/slim/download_and_convert_data.py 
# ===> /mnt/research/slim/download_and_convert_data.py
```



> apt install -y nfs-common 已经安装过了

### 5 提交任务

网页：222.195.151.231

在command里写上面小节4里编写好的训练命令

![](https://fermhan.oss-cn-qingdao.aliyuncs.com/img/20200531171438.png)



注意：只填入地址，没有前缀docker push

```bash
# 提交的镜像直接在网页上复制pull地址也可以(@sha格式)，自己编写也可以(:tag格式)。
# 格式1 # sha是版本号的编码
cvlab.qdxnzn.com/ouc/镜像名字@镜像版本号sha256:d2e056809cd55fc2605524e335efa9cc72b07ecffacb8cabf57335dc918d4fc3

# 格式2 # 拉取的是docker hub的，不是学校仓库的
openpai/pytorch-py36-cu90:lastest
# 或
cvlab.qdxnzn.com/ouc/镜像名字:镜像版本号cuda10.0-cudnn7.6-python6-tf15
```

第二次使用该镜像拉取得很快的，因为集群中有该镜像了。

- 在job name那里可以自己注解你训练的是什么任务

提交完任务后，可以取Jobs页面查看运行情况与标准输入、标准错误

### 6 自定义docker镜像/提交学校仓库

> 此部分是自己镜像仓库的使用，可以跳过

我们可以直接选择 cvlab.qdxnzn.com   里现有的镜像、可以基于这些镜像重新制作、也可以看第三部分自己从0开始制作。账号密码在开头

#### push与pull

如果你使用过`docker commit`命令，那你可能必须经历这步

```bash
# 1.先登录再push
sudo docker login --username=admin  cvlab.qdxnzn.com
#输入密码 Harbor12345

# 查看你的镜像id
sudo docker images 

#2. 将镜像推送到远程仓库,需要先登录
## 打标签，新的tag要写明该镜像中软件的版本，如python6-tf15-cuda10.0-cudnn7.6.0
sudo docker tag 【ImageId】 cvlab.qdxnzn.com/ouc目录/myYolo5镜像名字:v1版本号
# 无论你怎么起名，要注意学校仓库里要com之后那个单词的目录。其余都是你的镜像名字

## push
sudo docker push cvlab.qdxnzn.com/ouc目录/myYolo5镜像名字:v1版本号

# 注意我们为什么不直接在网页上pull拉取别人的镜像而要提交到学校的仓库？
# 是为了有时候我们会修改docker镜像，然后push提交到某一仓库我们在网页上才能有地址可拉取
# 容器形成镜像的
# docker commit [OPTIONS] CONTAINER_ID [REPOSITORY[:TAG]]

# 无用镜像及时删除整理
```

示例

```bash
sudo docker login --username=admin  cvlab.qdxnzn.com
# Harbor12345

sudo docker images
#REPOSITORY             TAG        IMAGE ID       CREATED       SIZE
#ultralytics/yolov5    latest    418a139445f7    16 hours ago   14.5GB


sudo docker tag 418a139445f7 cvlab.qdxnzn.com/ouc目录/myYolo5镜像名字:v1版本号
sudo docker push  cvlab.qdxnzn.com/ouc目录/myYolo5镜像名字:v1版本号
# 然后就可以在镜像仓库里看到你的镜像了
```

> 注：有些镜像在openpai里不好用，最好用openpai官方提供的镜像，非官方的什么镜像好用我也没太研究，只知道普通镜像是不好用的。
>
> 你也可以通过阿里云建立自己的仓库



# 二、docker基础学习

### 1 docker概念

镜像images相当于模板，容器container相当于模板实例化出来的实例。

镜像存储分为本地仓库和远程仓库。仓库中只能存放镜像，不能存放容器。要上传到仓库需要先把容器commit为镜像。

每个镜像有3个主要属性：所在仓库REPOSITORY、标签 TAG、ID号IMAGE ID。

ID可以唯一标识一个镜像，REPOSITORY:TAG也能唯一标识一个镜像。

### 2 docker基本命令

```bash
# 有的系统用户可能得加sudo

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
#如nocuda-python6-tf1.12。这样REPOSITORY[:TAG]表示为 cvlab.qdxnzn.com/docker-xxx:py38-tf12

# 打标签，相当于复制且重命名镜像，方便指定上传的仓库
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]

# 进入运行中的容器
docker exec -it 容器ID  /bin/bash

#根据Dockerfile创建镜像
docker build [OPTIONS] PATH | URL | -
docker build -t 机构/镜像名<:tags> 生成Dockerfile目录

# 删除镜像
docker image rm

# 登录仓库
sudo docker login --username=admin  cvlab.qdxnzn.com
# Harbor12345

# 1.先登录再push
sudo docker login --username=admin  cvlab.qdxnzn.com
#输入密码 Harbor12345


#2. 将镜像推送到远程仓库,需要先登录
## 打标签，新的tag要写明该镜像中软件的版本，如python6-tf15-cuda10.0-cudnn7.6.0
sudo docker tag 【ImageId】 cvlab.qdxnzn.com/ouc/镜像名字:[镜像版本号]
# 无论你怎么起名，要注意学校仓库里要com之后那个单词的目录。其余都是你的镜像名字

## push
sudo docker push cvlab.qdxnzn.com/ouc/镜像名字:[镜像版本号]
```



### 3 Dockerfile(附录)

> 无需观看

Dockerfile镜像描述文件

- 是一个包含组合镜像的命令的文本文档
- Docker通过读取Dockerfile中的指令按步自动生成镜像
- 把命令写在Dockerfile中，且Dockerfile必须命名为Dockerfile

基本命令：

```bash
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

```bash
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



# 三 Dockerfile模板(附录)

看到这说明你不想用我们https://cvlab.qdxnzn.com/harbor/projects/69/repositories 中的镜像，要从0开始制作镜像了。

- 提供几个openPAI可用镜像库：(不要直接基于普通版本ubuntu制作镜像，openPAI识别不了nfs命令)

https://hub.docker.com/u/openpai

https://hub.docker.com/r/nvidia/cuda/tags

这里提供几个openPAI基础镜像

```bash
docker pull openpai/pai.build.base:hadoop2.7.2-cuda9.0-cudnn7-devel-ubuntu16.04

docker pull openpai/tensorflow-py36-cu90:latest

docker pull openpai/tensorflow-py27-cu90:latest

docker pull openpai/pytorch:1.0-cuda10.0-cudnn7-devel

docker pull openpai/pytorch-py36-cu90:latest
```

找到镜像地址后，利用下面的Dockerfile模板安装一些必须软件

对于这个Dockerfile，一般来说从nvidia或者openPAI上下载下来基础镜像后，只需要修改第一行FROM就可以了。

我们本地仓库里cvlab.qdxnzn.com/ouc/theory-repository:为前缀的镜像基本已经指向过模板里的命令了，所以用不到这个Dockerfile，只需要基于我们镜像直接修改python、TensorFlow即可。

```bash
FROM openpai/pytorch:1.0-cuda10.0-cudnn7-devel
ENV LANG C.UTF-8
RUN apt-get update && \

	DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
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
        nfs-common \
        && \

    git clone --depth 10 https://github.com/Kitware/CMake ~/cmake && \
    cd ~/cmake && \
    ./bootstrap && \
    make -j"$(nproc)" install && \
    ldconfig && \
    apt-get clean && \
    apt-get autoremove && \
    rm -rf /var/lib/apt/lists/* /tmp/* ~/* 
 
 
RUN apt-get update && \
	apt-get install -y libreadline-dev 

    

EXPOSE 8888 6006
```



```bash
# ==================================================================
# tf 1.12.0 cuda9.0 cudnn7 py36
# ------------------------------------------------------------------

FROM nvidia/cuda:9.0-cudnn7-runtime-ubuntu16.04
ENV LANG C.UTF-8
RUN apt-get update && \

	DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
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
        nfs-common \
        && \

    git clone --depth 10 https://github.com/Kitware/CMake ~/cmake && \
    cd ~/cmake && \
    ./bootstrap && \
    make -j"$(nproc)" install 
 
 
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
    pip3 install --upgrade setuptools && \
    pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/
    
#RUN pip3 install tensorflow-gpu==1.12.0 -i https://pypi.tuna.tsinghua.edu.cn/simple

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



```dockerfile
# ==================================================================
# tf 1.12.0 cuda9.0 cudnn7 py36
# cuda10需要安装tf 1.15
# ------------------------------------------------------------------

FROM nvidia/cuda:9.0-cudnn7-runtime-ubuntu16.04
ENV LANG C.UTF-8
# 要安装的软件
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
        build-essential \
        apt-utils \
        ca-certificates \
        wget \
        nfs-common \
        git \
        nano \
        vim \
        libssl-dev \
        curl \
        openssh-server \
        unzip \
        unrar \
        && \

    GIT_CLONE="git clone --depth 10 https://github.com/Kitware/CMake ~/cmake && \
    cd ~/cmake && \
    ./bootstrap && \
    make -j"$(nproc)" install 
 
 
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
    pip3 install --upgrade setuptools  && \
    pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/
    
RUN pip3 install tensorflow-gpu==1.12.0 -i https://pypi.tuna.tsinghua.edu.cn/simple

# ==================================================================
# config & cleanup
# ------------------------------------------------------------------

RUN ldconfig && \
    apt-get clean && \
    apt-get autoremove && \
    rm -rf /var/lib/apt/lists/* /tmp/* ~/*  && \
    rm -R /Python-3.6.0 && \
    rm /Python-3.6.0.tgz


EXPOSE 8888 6006
```

在命令行执行生成镜像：

```bash
docker build  -t 【生成镜像仓库:TAG】   Dockerfile的所在目录
如
docker build  -t  cvlab.qdxnzn.com/ouc/theory-repository:python6-tf1.12-cuda9   ./
```

https://mirrors.aliyun.com/pypi/simple/

# 四 测试通过镜像(附录)

这里写一些测试通过的镜像

```bash
echo "task start..."
apt update
apt install -y nfs-common
echo "task 1..."
mount -o nolock -t nfs 192.168.1.4:/data/models /mnt
echo "pip install..."
python /mnt/research/slim/download_and_convert_data.py --dataset_name=cifar10 --dataset_dir=/tmp/data
echo "task 3..."
python /mnt/research/slim/train_image_classifier.py --dataset_name=cifar10 --dataset_dir=/tmp/data --max_number_of_steps=1000 --batch_size 64
```

该组命令在下面镜像上正常训练过：

- openpai/tensorflow-py36-cu90:latest
- openpai/pytorch-py36-cu90:latest

# 五 阿里云仓库(过期)

我们有校内仓库了，所以无需阿里云仓库了。此部分可以不看

#### 阿里云仓库

浏览器输入https://cr.console.aliyun.com/cn-qingdao/instances/repositories

创建命名空间

![](https://fermhan.oss-cn-qingdao.aliyuncs.com/img/20200529174457.png)

创建仓库：

![](https://fermhan.oss-cn-qingdao.aliyuncs.com/img/20200529195534.png)

然后点击创建好的仓库，在页面下面就提示了你怎么使用你的阿里云仓库。

创建完阿里云仓库后重新看第5部分，push到你阿里云仓库即可。

```bash
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

