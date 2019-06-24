==**For theory and texture group only.**==

为增加查阅体验与防止后续服务器内容的更改，请直接在github观看 https://github.com/OUCvisionLab/ServerGuide

本次更新内容：

- 会让3个人左右共同使用和维护一台专属服务器（自己组的服务器自行解决），也留下了1台机动的服务器。
- 考虑到留学生，所以采用了中英结合的书写方式
- 解释了一些虚拟环境、源、ssh，传文件的知识

[TOC]

# 1、服务器列表Server list

| IP              | 端口号Port | capacity                                       | user name | password | Note          |
| --------------- | ---------- | :--------------------------------------------- | --------- | -------- | ------------- |
| 222.195.151.170 | 9013       | RAM:128G, CPU:2.3GHz*40, GPU:none              | ouc-13    | b301     | GPU:none      |
| 222.195.151.170 | 9015       | RAM:128G, CPU:2.3GHz*40, GPU:TESLA K40c 11G *1 | ouc-15    | b301     | LY,ZTG        |
| 222.195.151.170 | 9028       | RAM:32G, CPU:3.5GHz*8, GPU:1080Ti 11G *2       | ouc-28    | b301     | LJX,HF        |
| 222.195.151.170 | 9029       | RAM:32G, CPU:3.5GHz*8, GPU:2080 8G *2          | ouc-29    | b301     | HF            |
| 222.195.151.66  | 9010       | RAM:32G, CPU:3.3GHz*4, GPU:TITAN X 12G         | ouc-10    | b301     | Aman,Israel   |
| 222.195.151.66  | 9018       | RAM:32G, CPU:3.5GHz*8, GPU:1080Ti 11G *2       | ouc-18    | b301     | ZQQ,LJH,Sadia |
| 222.195.151.66  | 9019       | RAM:32G, CPU:3.5GHz*8, GPU:1080Ti 11G *2       | ouc-19    | b301     | SQY,SCX       |

!!!  port for transporting files is 91-- ,NOT 90--

传文件的端口是91--，选的协议是sftp。

# 2、登录服务器login

**2.1 Find remote connection**
![](https://raw.githubusercontent.com/FermHan/tuchuangsimi/master/20190325172634.png?token=AkTVJfvkXHdCyhSbXbtS6iokfCOR6xZNks5cmJ8MwA%3D%3D)

**2.2 Type in IP:port**
![](https://raw.githubusercontent.com/FermHan/tuchuangsimi/master/20190325172652.png?token=AkTVJYakMJzCIiVJDfjlIcg5KLcv0mctks5cmJ8owA%3D%3D)

**2.3 Type in username and password**

![](https://raw.githubusercontent.com/FermHan/tuchuangsimi/master/20190325135817.png?token=AkTVJVynPSWj1sb4ZEbO8wRyjpg_8P4cks5cmG46wA%3D%3D)

**2.4 Once in the system, modify someone.txt to note your name and usage time.And you can put your personal files in directory  /home/ouc-xx/**在home目录下创建自己的文件夹
![](https://raw.githubusercontent.com/FermHan/tuchuangsimi/master/20190325144734.png?token=AkTVJRoQTQFVopFyApR5WI9oEZziwdXtks5cmHnIwA%3D%3D)

**2.5 Before you run the code, please type in  ` nvidia-smi ` in the terminal to make sure there's no another user.**(在你正式跑代码之前，请输入`nvidia-smi`查看有没有其他用户在跑程序（通过红框部分看）)。

![](https://raw.githubusercontent.com/FermHan/tuchuangsimi/master/20190325150409.png?token=AkTVJdMwtfgMAto3CRd4hvoScKzyrl_kks5cmH2rwA%3D%3D)

nvidia-smi参数解释：

```
- Fan：显示风扇转速，数值在0到100%之间，是计算机的期望转速，如果计算机不是通过风扇冷却或者风扇坏了，显示出来就是N/A；
- Temp：显卡内部的温度，单位是摄氏度；
- Perf：表征性能状态，从P0到P12，P0表示最大性能，P12表示状态最小性能；
- Pwr：能耗表示；
- Bus-Id：涉及GPU总线的相关信息；
- Disp.A：是Display Active的意思，表示GPU的显示是否初始化；
- Memory Usage：显存的使用率；
- Volatile GPU-Util：浮动的GPU利用率；
- Compute M：计算模式；
```

# 3、Use python and tensorflow

### 3.1 First and Foremost：`nvidia-smi`

此处是作用是验证没有人在使用此服务器

### 3.2 create environment # 创建你自己的环境

> 环境解释：
>
> 之前为大家创建过公用的python3.6和python2.7等，但是大家好像更倾向于使用很特殊的环境。所以此处修改为创建自己虚拟环境的注意项。 
>
> 虚拟环境的原因：相当于两台服务器，互相使用的python不影响，即安装模块是安装到了指定某一python上，而不是为全部python安装模块。
>
> 公共python的位置：Anaconda原有的python路径是/home/ouc/Anaconda3/python。而你创建私有的python地址是/home/Anaconda3/env/YOURNAME/python
>
> 环境变量在~/.bashrc文件里，不要轻易改动。~代表home目录。安装anaconda时一路默认会自动写入bashrc中。

创建于使用虚拟环境方法：

```python 
# How to create a new environment创建环境
conda create -n YOURENAME python=PYTHONVERSION
# 如conda create -n hanfeng python=3.6。可以指定python版本，重要的是指定python版本。名称任意，推荐自己名字。anaconda3上也可安装python2.7。创建完可在/home/Anaconda3/env/YOURNAME/python目录下找到你的python

# 创建完自己的环境后，接下来一切操作都是在【进入自己创建的环境】的基础上进行的，下面语句是进入自己环境的方式，二选一。注：若使用python，但凡打开终端，第一步都该的进入自己的环境，否则操作都不是针对自己的环境的。
source activate YOURENAME 
#or 
conda activate YOURENAME
-------------------------------
# 不常用命令：
# View existing conda list 查看现有环境
conda info -e

source deactivate  #退出私有环境，返回公共环境
#or#conda deactivate

# How to delete your personal environment 删除虚拟环境
conda remove -n YOURNAME --all
```

### 3.3 install modules安装包：

#### 3.3.1 conda install

```PYTHON
1. # use conda。conda安装方式，必须先激活到自己创建的环境中
conda activate YOURENAME # or：source activate YOURENAME
conda install tensorflow-gpu=版本号
```

附conda更换镜像方式：

#### 3.3.2 pip install

- pip在线安装：

```PYTHON
conda activate YOURENAME # or：source activate YOURENAME
python -m pip install 模块 # 
```

- pip离线安装

```PYTHON
# 如何下载离线包查看3.3.3
#下载后用pip安装，安装时候输入完python -m pip install 把文件拖进去即可，相当于要输入文件绝对路径。注意文件两侧各有一个'号，压缩包直接拖进去即可，无需解压
conda activate YOURENAME # or：source activate YOURENAME
python -m pip install FILE下载的文件
```

- pip知识补充

  - 更换pip源方式

  ```PYTHON
  终端输入gedit  ~/.pip/pip.conf，打开后复制以下内容保存：
  [global]
  index-url = https://pypi.tuna.tsinghua.edu.cn/simple
  ```

  - 如果你要使用pip，**务必先激活到自己的虚拟环境`conda activate YOURENAME`，然后使用`python -m pip install XXX`的形式代替`pip install xxx`的形式**。因为直接输入的pip指向的并不是你的python，而是别人的。想要一探究竟可以打开/usr/local/bin下的pip文件看其原理
    尤其是像pytorch这种包，conda命令经常安不上，使用pip命令的时候一定要使用'python -m'方式。
  - pip3和pip是两个文件。

  如下图，使用`pip3 -V`或`pip -V`可以查看这两个文件指向哪里，即使用pip时默认为哪个python安装包。如图，分别指向的python是系统的python和anaconda的其中一个python。

  使用`which pip3`或`which pip`可以查看默认的pip3和pip在哪里。如图，pip3在/usr/bin目录下，pip在anaconda3/bin目录下

![](https://raw.githubusercontent.com/FermHan/tuchuang/master/20190621164105.png)

​		输入`gedit /usr/local/pip3`可以打开pip3修改第一行，修改为自己python的路径以后pip3以后默认的安装的就是你的python了。

![](https://raw.githubusercontent.com/FermHan/tuchuang/master/20190621164221.png)

​		但是上面只是介绍原理，实例使用中最实用的还是直接使用`python -m pip install 在线/离线包`，相当于指定了为哪个python安装包。

```PYTHON
conda activate YOURENAME # or：source activate YOURENAME
python -m pip install FILE下载的文件（在线离线均可）
```

#### 3.3.3 离线安装包的下载

- 方式一：module官网下载离线包

例：pytorch，官网https://pytorch.org/ 给出的pip安装方式显示的网址即是包的下载地址，可以去掉pip复制网址到浏览器下载。如下图选中部分即下载地址。

![](https://raw.githubusercontent.com/FermHan/tuchuang/master/20190621160354.png)

注：经测试cuda-9.0官网没给出下载pytorch地址，第三方给的下载文件https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/linux-64/ 测试时有次未安装成功，可以自己尝试。

- 方式二：镜像网下载离线包https://mirrors.tuna.tsinghua.edu.cn/anaconda/ 。

archive下是anaconda安装包

/pkgs/free/linux-64/下是如tensorflow等安装包

/cloud/pytorch/linux-64/下是torch和torchvision安装包

- 方式三：anaconda官网给的离线包：https://repo.continuum.io/pkgs/free/linux-64/

#### 3.3.4 源问题

清华源在2019-04-16被迫停止了anaconda镜像服务，但随后2019-06-15又获得了Anaconda镜像的授权。见下图，所以以后又能继续在线使用anaconda安装module了。

![](https://raw.githubusercontent.com/FermHan/tuchuang/master/20190621155850.png)

更改conda源的方式：

- 添加源：一般常用的是中科大源和清华源

```python 
#输入gedit ~/.condarc复制以下内容后保存：
channels:
  - https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
  - https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
  - defaults
show_channel_urls: true
```

- 删除源：conda config --remove-key channels

### 3.4 navigator

ps：你也可以使用navigator界面的方式进行上面创建虚拟环境安装包等操作

ps: Similarly,you can type in 

`anaconda-navigator`

in the terminal to open NAVIGATOR, and choose python from NAVIGATOR.

![](https://raw.githubusercontent.com/FermHan/tuchuangsimi/master/20190325170518.jpg?token=AkTVJfCdox_AKmenkZWtbZejKnxxdoVMks5cmJoSwA%3D%3D)



# 4、How to transfer files 传文件

如果需要传文件，无需去机房拷贝，用Xftp传即可，文件可在网上下载破解版。

You can install **Xftp** in windows to transfer files. # windows安装xftp软件

Protocol:sftp  # 协议选sftp

port is NOT 90-- BUT 91--  # 端口号是91--，而不是原来的90--

![](https://raw.githubusercontent.com/FermHan/tuchuang/master/20190531220350.png)![](https://raw.githubusercontent.com/FermHan/tuchuang/master/20190531222935.png)

![](https://raw.githubusercontent.com/FermHan/tuchuang/master/20190531220703.png)

![](https://raw.githubusercontent.com/FermHan/tuchuang/master/20190531220819.png)

# 5、只使用命令行不使用界面：Xshell

windows的远程的确有时候很卡，目前我也不知道该如何解决，或许在ubuntu18没有解决方案。至于为什么卡还在使用这个远程的原因请看6.2。

这里提高两种暂时的解决方案。

- 第一种方法是开teamviewer。但teamviewer与远程界面有时候有些矛盾，这可能是安的桌面的问题。所有teamviewer不一定100%有效。
- 第二种方法是用ssh命令行模式

> 若链接ssh不成功，可先安装：
>
> 如果ubuntu里没有ssh可以通过如下命令安装
> `sudo apt-get install openssh-server`，在ubuntu端安装ssh。
> 输入"sudo ps -e |grep ssh"-->回车-->有sshd,说明ssh服务已经启动，如果没有启动，输入"sudo service ssh start"-->回车-->ssh服务就会启动。
>
> 注：ssh也可在安装显卡驱动时使用，无需跑去机房安装。

在windows上安装xshell使用命令行：

破解xshell地址：https://blog.csdn.net/u011622631/article/details/88991941

安装后，打开xshell，新建链接如图，端口为91--

![](https://raw.githubusercontent.com/FermHan/tuchuang/master/20190531232754.png)

然后输入账户密码 后点ok

![](https://raw.githubusercontent.com/FermHan/tuchuang/master/20190531232837.png)

连接成功，界面如下：

![](https://raw.githubusercontent.com/FermHan/tuchuang/master/20190531233523.png)

![](https://raw.githubusercontent.com/FermHan/tuchuang/master/20190605152403.png)

# 6、What's more

### 6.1 为什么要安装小老鼠这个界面？

因为teamviewer总会出现商业版问题，所以无奈选择远程连接的方式，如果你使用时间较长，可以试着连teamviewer使用。

因为ubuntu不好实现远程连接，必须通过安装小老鼠界面间接控制ubuntu。ubuntu16可能有解决方案，但ubuntu18较难解决，而我们的ubuntu当初安装的是18版本，所以尽管小老鼠界面不美观，但还得接着使用。

- ubuntu18配置远程参考此链接的第二个方法https://blog.csdn.net/star2523/article/details/81152890
- 在ubuntu16下可能存在完美的解决方式请参考：https://blog.csdn.net/qq_37674858/article/details/80931254 ， https://www.cnblogs.com/xuliangxing/p/7642650.html
- 原来服务器配置人员的博客：https://blog.csdn.net/zhouxiaowei1120/article/details/80872919

> 如有在远程上打不开终端，可以使用sudo apt-get remove gnome*

### 6.2 如何安装cuda，显卡驱动等

参考链接 https://blog.csdn.net/hancoder/article/details/86634415

### 6.3 重装服务器系统后需要做什么

- 配置显卡驱动，cuda，cudnn
- 重新配置IP以便可以远程连接
- 安装ssh以便文件传输：

​	`apt-get install openssh-server`

​	vim /etc/ssh/sshd_config

​	将PermitRootLoginwithout-password注释，                                

​	添加一行： PermitRootLoginyes

​	`service ssh restart`

远程连接方式可看上面小老鼠问题

### 6.4 一些其他内容

6.5.1 配置环境变量的文件Some environment variables are configured in `~/.bashrc`

6.5.2 please debug your code on your PC to save server resources.

6.5.3 If your server resources are insufficient, please contact HAN. There may be servers unallocated for you.

6.5.4 以后更新尽量在此github更新IP等内容，账号即OUCvisionLab，密码可问管理员索要。

6.5.5 Maybe you want to install Anaconda2.To be honest, it's not usually used, because python2.7 has been involved in Anaconda3. If you think about it, what is noteworthy is that when you install anaconda2,

> Anaconda2 will now be installed into this location:home/xx/anaconda2
>
> -Press ENTER to confirm the location
>
> -Press CTRL-C to abort the installation
>
> -Or specify a different location below
>
> [/home/xx/anaconda2]>>>

don't press ENTER, you should type in your personal directory such as : `/home/ouc/Tom/Anaconda2/`

> Do you with the installer to prepend the Anaconda2 install location to PATH in your /home/ouc/.bashrc ?[yes|no]

please type in `no`
