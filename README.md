[TOC]

==**For theory and texture group only.**==

为增加查阅体验与防止后续服务器内容的更改，请直接在github观看 https://github.com/OUCvisionLab/ServerGuide  ， 若github网速慢也可去CSDN观看：https://blog.csdn.net/hancoder/article/details/88803079 

主要内容：

- IP设置：https://blog.csdn.net/hancoder/article/details/102881903 
- openAI集群+docker：https://blog.csdn.net/hancoder/article/details/106423288 

20.03.08次更新内容：

- 会让2个人左右共同使用和维护一台专属服务器（自己组的服务器自行解决），也留下了1台机动的服务器。
- 解释了一些虚拟环境、ssh，xftp、zerotier等可能用到的知识
- 如有跑实验需要，是可以临时调换的。



!!!禁止在服务器上阅读代码与长时间修改代码，浪费资源。

# 1、服务器列表

注：能力比较强的可以申请使用openpai集群，服务器单机资源有限

本服务器为OUC理论组管理，供D老师和G老师的学生使用。理论上其他组也有自己的服务器，请尽量使用自己组内服务器，方便管理和调度。

| IP              | Port            | capacity                                       | user name | password | Note         |
| --------------- | --------------- | :--------------------------------------------- | --------- | -------- | ------------ |
| 备注行          | 桌面是0，ssh是1 |                                                |           |          |              |
| 222.195.151.170 | 9013/9113       | RAM:128G, CPU:2.3GHz*40, ==GPU:none==          | ouc-13    | b301     | GPU:none;FTP |
| 222.195.151.170 | 9015/9115       | RAM:128G, CPU:2.3GHz*40, GPU:TESLA K40c 11G *1 | ouc-15    | b301     | LY,ZTG       |
| 222.195.151.170 | 9028/9128       | RAM:32G, CPU:3.5GHz*8, GPU:1080Ti 11G *2       | ouc-28    | b301     | LJX,LJH      |
| 222.195.151.170 | 9029/9129       | RAM:32G, CPU:3.5GHz*8, GPU:2080 8G *2          | ouc-29    | b301     | YYW,HF       |
| 222.195.151.66  | 9010/9110       | RAM:32G, CPU:3.3GHz*4, GPU:TITAN X 12G         | ouc-10    | b301     | Aman,Israel  |
| 222.195.151.66  | 9018/9118       | RAM:32G, CPU:3.5GHz*8, GPU:1080Ti 11G *2       | ouc-18    | b301     | ZZD,LWX,QXF  |
| 222.195.151.66  | 9019/9119       | RAM:32G, CPU:3.5GHz*8, GPU:1080Ti 11G *2       | ouc-19    | b301     | SCX,Sadia    |
|                 | 9055/9155       |                                                | 密        | 密       | SQY,ZQQ,GYH  |

注：

- 请注意区别端口是90XX还是91XX，一个是远程桌面，一个是ssh。我们组90XX是桌面，别的组有的90XX是ssh
- **传文件的端口是91--，选的协议是sftp。**
- 校园网有墙，校园外是无法访问无法访问服务器的，需要借助于part 6的zerotier软件



# 2、登录

**2.1 windows软件**

电脑中搜索桌面连接。或者在微软商店里搜远程桌面，道理都是一样的。或windows+r输入mstsc

![](https://fermhan.oss-cn-qingdao.aliyuncs.com/img/20200626192119.png)

**2.2 输入IP和端口**

输入IP:port，如`222.195.151.170:6666`

![](https://fermhan.oss-cn-qingdao.aliyuncs.com/img/20200626192215.png)

端口是90XX，其他组的服务器有的是91XX是桌面端口。

**2.3 输入用户名密码**

session模式选择Xorg（个别机器选sesman-Xvnc），然后输入账号`ouc-机器后两位id`，密码`b301`

![](https://fermhan.oss-cn-qingdao.aliyuncs.com/img/20200626192344.png)

**2.4 登录系统后 在home目录下(/home/ouc-xx/)创建自己的文件夹，请勿随意放置个人文件**

进去后，最好在桌面上备注好你的名字以及使用时间。

![](https://fermhan.oss-cn-qingdao.aliyuncs.com/oldGithub/20190325144734.png?token=AkTVJRoQTQFVopFyApR5WI9oEZziwdXtks5cmHnIwA%3D%3D)

2.5 ==**在你正式跑代码之前，请输入`nvidia-smi`查看有没有其他用户在跑程序（通过红框部分看）)。如果中间的显存占用率只有几十MB，那么就说明没人在跑程序。**==

![](https://fermhan.oss-cn-qingdao.aliyuncs.com/img/20200626192757.png)

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

### 浏览器问题

如果发现打不开服务器的浏览器，可能是因为bug的原因，我们可以安装谷歌浏览器代替：

安装方法：去谷歌官网找安装包(下载地址在页面下方其他平台里)  或者用wget方式下载

下载后传到服务器上

然后

```
sudo dpkg -i 下载的包.deb
然后就可以在左上角的浏览器里找到了
```



#  3、使用python、tensorflow

### 3.1 `nvidia-smi`

输入 `nvidia-smi`，此处是作用是验证没有人在使用此服务器

<img src="https://fermhan.oss-cn-qingdao.aliyuncs.com/img/20200626192757.png" style="zoom:50%;" />

==但要注意这里的cuda版本不准确，要用`nvcc -V`查看cuda版本==

#### nvidia-smi报错解决方案

如果报错

```
NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver. Make sure that the latest NVIDIA driver is installed and running.
```

一种方法是重装驱动，但如果已经重装过了但是重启后又报错了，就可以使用下面的方式

```sh
# 查看之前安装的驱动版本
ls /usr/src | grep nvidia
# 比如这里我输出了
nvidia-440.31


sudo apt-get install dkms #DKMS全称是Dynamic Kernel Module Support，它可以帮我们维护内核外的这些驱动程序，在内核版本变动之后可以自动重新生成新的模块。

sudo dkms install -m nvidia -v 440.31 #440.31是安装驱动的版本
```

然后就可以了。

如果还不可以说明不是内核的问题，是驱动没安好，继续重装吧



### 3.2 创建你自己的环境

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
# ---|*_*|--- 查看现有环境 ---|*_*|---
conda info -e

# ---|*_*|--- 创建python环境 ---|*_*|---
conda create -n YOURENAME python=PYTHONVERSION
# 如conda create -n hanfeng python=3.6。可以指定python版本，重要的是指定python版本。名称任意，推荐自己名字。anaconda3上也可安装python2.7。创建完可在/home/Anaconda3/env/YOURNAME/python目录下找到你的python
# 如遇到权限问题，可以尝试先执行如 sudo chmod 777 /home/ouc-19/anaconda3

# ---|*_*|--- 以下两条指令都要执行一下，之前发现过不执行莫名其妙进去别人环境的问题 ---|*_*|---
source activate # 重新进入虚拟环境
conda deactivate # 退出虚拟环境

# ---|*_*|--- 激活环境 ---|*_*|---
# 创建完自己的环境后，接下来一切操作都是在【进入自己创建的环境】的基础上进行的，下面语句是进入自己环境的方式，二选一。注：若使用python，但凡打开终端，第一步都该的进入自己的环境，否则操作都不是针对自己的环境的。
source activate YOURENAME 
#or 
conda activate YOURENAME
-------------------------------
#  ---|*_*|--- 不常用命令 ---|*_*|--- 
# 退出环境：
source activate # 重新进入虚拟环境
conda deactivate # 退出虚拟环境
#or#source deactivate 

#  ---|*_*|--- 删除虚拟环境 ---|*_*|--- 
conda remove -n YOURNAME --all
```

##### 使用自己环境示例：

```sh
(hanfeng) ouc-10@ouc-10:~$ conda info -e
# conda environments:
base                     /home/ouc-10/anaconda3
hanfeng               *  /home/ouc-10/anaconda3/envs/hanfeng

(base) ouc-10@ouc-10:~$ python -V
Python 3.7.4

(base) ouc-10@ouc-10:~$ conda activate hanfeng

(hanfeng) ouc-10@ouc-10:~$ python -V
Python 3.7.4 # 还是3.7的环境，明显不是我自己的

(hanfeng) ouc-10@ouc-10:~$ which python
/home/ouc-10/anaconda3/envs/hanfeng/bin/python # 路径居然正确？

# 执行退出
(hanfeng) ouc-10@ouc-10:~$ source activate
(base) ouc-10@ouc-10:~$ conda deactivate
# 重新进入
ouc-10@ouc-10:~$ conda activate hanfeng

(hanfeng) ouc-10@ouc-10:~$ python -V # 版本正确了
Python 3.6.2 :: Continuum Analytics, Inc.

(hanfeng) ouc-10@ouc-10:~$ pip -V
pip 9.0.1 from /home/ouc-10/anaconda3/envs/hanfeng/lib/python3.6/site-packages (python 3.6)

(hanfeng) ouc-10@ouc-10:~$ pip3 -V
pip 20.2.2 from /home/ouc-10/anaconda3/lib/python3.7/site-packages/pip (python 3.7)


# 安装包 
python -m pip install tensorflow-gpu
说明：请一定要使用如上的方式安装包，不要直接pip install tensorflow-gpu或者pip3 install
原因：我们可以输入which pip查看当前默认使用的pip，可以发现pip可能确实匹配到了正确的自己环境，
但pip3就不是自己的环境，所以尽量使用python -m pip install的命令。下面有更详细的说明
(hanfeng) ouc-10@ouc-10:~$ which pip
/home/ouc-10/anaconda3/envs/hanfeng/bin/pip
(hanfeng) ouc-10@ouc-10:~$ which pip3
/usr/bin/pip3
```



### 3.3 安装包：

推荐离线安装。去https://pypi.org/ 下载离线包。`pip install 离线包`

在线安装容易失败。

#### 3.3.1 conda install

```PYTHON
# conda安装方式，必须先激活到自己创建的环境中
conda activate YOURENAME # or：source activate YOURENAME
conda install tensorflow-gpu=版本号
```

另外需要注意版本的关系，tf、torch与cuda版本有关，要验证自己的版本是否可用，可用如下类似的方式验证

```python
import torch
torch.cuda.is_available()  # 输出true才代表当前版本的torch能用
# false的话更改cuda版本或者torch版本
# 更改cuda的教程在末尾
# 可以取torch官网看与cuda的对应关系
https://pytorch.org/get-started/previous-versions/
```

#### 3.3.2 pip install

- pip在线安装：

```PYTHON
conda activate YOURENAME # 或：source activate YOURENAME
python -m pip install 模块 # 
# 还可以在文件中写好后，指定源路径，加速下载
python -m pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
# 其中requirements.txt内容格式如下
torch==1.4.0
torchvision==0.5.0
```

- pip离线安装

```PYTHON
# 如何下载离线包查看3.3.3
#下载后用pip安装，安装时候输入完python -m pip install 把文件拖进去即可，相当于要输入文件绝对路径。注意文件两侧各有一个'号，压缩包直接拖进去即可，无需解压
conda activate YOURENAME # or：source activate YOURENAME
python -m pip install FILE下载的文件
如
python -m pip install tensorflow-gpu==1.15
```

- pip知识补充

  - 更换pip源方式：见后文

  - 如果你要使用pip，**务必先激活到自己的虚拟环境`conda activate YOURENAME`，然后使用`python -m pip install XXX`的形式代替`pip install xxx`的形式**。因为直接输入的pip指向的并不是你的python，而是linux里的python。想要一探究竟可以打开/usr/local/bin下的pip文件看其原理
    尤其是像pytorch这种包，conda命令经常安不上，使用pip命令的时候一定要使用'python -m'方式。
  - pip3和pip是两个文件

```sh
(hanfeng) ouc-10@ouc-10:~$ which python
/home/ouc-10/anaconda3/envs/hanfeng/bin/python

# 使用`pip3 -V`或`pip -V`可以查看这两个文件指向哪里，即使用pip时默认为哪个python安装包。如图，分别指向的python是系统的python和anaconda的其中一个python。
(hanfeng) ouc-10@ouc-10:~$ pip -V
pip 9.0.1 from /home/ouc-10/anaconda3/envs/hanfeng/lib/python3.6/site-packages (python 3.6)

(hanfeng) ouc-10@ouc-10:~$ pip3 -V
pip 20.2.2 from /home/ouc-10/anaconda3/lib/python3.7/site-packages/pip (python 3.7)

# 使用`which pip3`或`which pip`可以查看默认的pip3和pip在哪里 #如下，pip3在/usr/bin目录下
(hanfeng) ouc-10@ouc-10:~$ which pip
/home/ouc-10/anaconda3/envs/hanfeng/bin/pip

(hanfeng) ouc-10@ouc-10:~$ which pip3
/usr/bin/pip3
```

![](https://fermhan.oss-cn-qingdao.aliyuncs.com/oldGithub/20190621164105.png)
	
输入`gedit /usr/local/pip3`可以打开pip3修改第一行，修改为自己python的路径以后pip3以后默认的安装的就是你的python了。

![](https://fermhan.oss-cn-qingdao.aliyuncs.com/oldGithub/20190621164221.png)
	
但是上面只是介绍原理，实例使用中最实用的还是直接使用`python -m pip install 在线/离线包`，相当于指定了为哪个python安装包。

```PYTHON
conda activate YOURENAME # or：source activate YOURENAME
python -m pip install FILE下载的文件（在线离线均可）
```

#### 3.3.3 离线安装包的下载

- 方式一：module官网下载离线包

例：pytorch，官网https://pytorch.org/ 给出的pip安装方式显示的网址即是包的下载地址，可以去掉pip复制网址到浏览器下载。如下图选中部分即下载地址。

![](https://fermhan.oss-cn-qingdao.aliyuncs.com/oldGithub/20190621160354.png)

注：经测试cuda-9.0官网没给出下载pytorch地址，第三方给的下载文件https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/linux-64/ 测试时有次未安装成功，可以自己尝试。

- 方式二：镜像网下载离线包https://mirrors.tuna.tsinghua.edu.cn/anaconda/ 。

archive下是anaconda安装包

/pkgs/free/linux-64/下是如tensorflow等安装包

/cloud/pytorch/linux-64/下是torch和torchvision安装包

- 方式三：anaconda官网给的离线包：https://repo.continuum.io/pkgs/free/linux-64/

- 方式四：pip地址：https://pypi.org/

  ```sh
  #以numpy为例，下载下来离线包之后
  python -m pip install ./numpy-1.15.0-cp27-cp27mu-manylinux1_x86_64.whl
  ```

  

#### 3.3.4 源问题

linux软件源：https://developer.aliyun.com/mirror/ubuntu?spm=a2c6h.13651102.0.0.3e221b11JFiKUi

##### 更改conda源

- 添加源：一般常用的是中科大源和清华源

``` python 
#输入gedit ~/.condarc复制以下内容后保存：
channels:
 - https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
 - https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/
 - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
 - defaults
show_channel_urls: true
```

- 删除源：conda config --remove-key channels

##### 更改pip源

```sh
# 临时更换：-i后面跟上源地址
pip install pythonModuleName -i https://mirrors.aliyun.com/pypi/simple

#永久更改
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

或者采用如下方式：
#创建目录
mkdir -p ~/.pip
#修改配置文件
vim  ~/.pip/pip.conf
#写入以下内容并保存
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```




### 3.4 navigator

ps：你也可以使用navigator界面的方式进行上面创建虚拟环境安装包等操作

![](https://fermhan.oss-cn-qingdao.aliyuncs.com/oldGithub/20190325170518.jpg?token=AkTVJfCdox_AKmenkZWtbZejKnxxdoVMks5cmJoSwA%3D%3D)

### Anaconda2

若有特殊情况需要安装Anaconda2（Anaconda3里有python2.7），可以使用如下方法：

> Anaconda2 will now be installed into this location:home/xx/anaconda2
>
> -Press ENTER to confirm the location
>
> -Press CTRL-C to abort the installation
>
> -Or specify a different location below
>
> 这时不要选择回车，否则会覆盖原来的Anaconda3的环境变量。自己输入一个home下的路径即可
>
> Do you with the installer to prepend the Anaconda2 install location to PATH in your /home/ouc/.bashrc ?[yes|no]
>
> 要选择no，否则会覆盖原来的Anaconda环境变量，使别人无法正常使用

# 4、后台训练任务screen/nohup &

我们在ssh上`python XXX.py`开始训练任务后，如果关闭了ssh窗口，任务就会中断，我们可以使用如下两种方式之一进行后台训练

- `nohup python train.py 参数 &`：即以nohub开头，以&结尾。默认将当前的输出打印在当前目录的nohup.out文件里
  - nohup表示不挂断
  - &表示后台运行命令
- screen 

### 4.1 screen

场景：比如我们`vi test.txt`编辑到一半的时候要回来了不得不关机，但我们明天重开机编辑的内容就撤销了，我们可以使用如下的方式进行临时保存

```sh
# ===安装screen===
sudo apt install screen

# ===在要执行的任务前加上screen===
screen python aaa.py
# 也可以使用-S指定任务名: screen -S jobName python aaa.py  
# 可以用-L指定输出的日志，会在当前目录下生成screenlog.0文件。可以通过tail查看最终的日志。可以参考https://blog.csdn.net/weixin_44058333/article/details/99693489
建议的用法如下：
screen -L -S job123 python aaa.py

# 暂时离开刚页面
# 按Ctrl+a，按完后再按d，就退出了训练页面，但任务还在进行
# 给窗口自定义命名：ctrl+a 按完后再按A

# ===重新连接会话===
#screen -ls找到screen的任务
screen -ls
There is a screen on:

        16582.pts-1.tivf06      (Detached)

1 Socket in /tmp/screens/S-root.

# ===连接screen的任务===
screen -r 16582
# 又能看到训练页面了

# ======其他内容========
# 将当前在另一个终端attach的会话强制退出，在当前终端接管：screen -d name screen -r name

# 上下移动：ctrl-a w向前，b向后

# Ctrl-a +S 分隔屏幕，Ctrl-a+Tab可以切换区域，Ctrl-a+Q关闭其他区域块

# 如果连不上，可能是其他页面没有关掉
# screen -ls ，发现其状态是 Attached，正常可连接的是 Detached，说明之前用户还占有那个session，直接踢掉他。
screen -D  -r [name]
```

4.2 nuhup ... &

 

# 5、ssh工具MobaXterm

原来我们推荐xftp和xshell作为ssh工具。现在我们推荐MobaXterm作为ssh工具。Xterm可以实现ssh、xftp等功能，直接文件拖拽互传，对新手友好，界面优美。

下载地址：https://mobaxterm.mobatek.net/download-home-edition.html

端口是91XX，而不是90XX，也有的服务器这两个端口是反的，

#### ssh基础命令

```BASH
pwd #打印当前目录
cd 目录 #转换目录，分为相对路径和绝对路径，第一位是/的是绝对路径
wget www.链接 #下载文件
ls #查看当前目录文件
mkdir #创建目录
vim #编辑文件
# vim分为命令模式，插入模式和底线模式。了解前两个即可
# 刚进去的适合是命令模式，此时只能查看文件，按h左j下k上l右；按i进入插入模式编辑文件
# 编辑好后按Esc，然后输入:wq代表保存退出，q！代表不保存退出
# 补充：命令模式下：u撤销 Ctrl+r取消撤销 x删除当前光标字符 X是向前删 x是向后删
# 其他命令自己学习

sudo apt install lrzsz
rz 上传文件 # 需要先安装sudo apt install lrzsz
sz下载文件
```

> 若链接ssh不成功，可先安装：
>
> 如果ubuntu里没有ssh可以通过如下命令安装
> `sudo apt-get install openssh-server`，在ubuntu端安装ssh。
> 输入"sudo ps -e |grep ssh"-->回车-->有sshd,说明ssh服务已经启动，如果没有启动，输入"sudo service ssh start"-->回车-->ssh服务就会启动。
>
> 注：ssh也可在安装显卡驱动时使用，无需跑去机房安装。

# 6、外网使用服务器zerotier

我们通过zerotier这个软件映射校园网到校园外网(家中局域网)

操作步骤：

2. 下载Windows版本的Zerotier：`MSI Installer (x86/x64)`（为方便，临时提供百度云方式下载连接：

   https://pan.baidu.com/s/1eiJy2A5PxT-o0yyhnR3dWA 提取码：f58H ）。（百度云链接失效的话可以去官网下载：https://www.zerotier.com/download/  。只是加入网络的话不需要注册账号）

3. 安装好后，在Zerotier软件界面（或者在任务栏小图标上右键），点击join Netwok，输入ID号：a09acf023363b091。需要联系后台管理员通过才能通过授权（请主动联系一下，要不然不知道请求访问的ip具体是谁，也不能及时通过请求）。

4. IP：访问新的指定的服务器IP（与校园网IP不一样了）。==新的IP地址为：`192.168.192.两位IP号`，比如你原来的服务器是ouc-28，那么服务器新的IP即：`192.168.192.28`==。zerotier的ip全是这个规则

5. 端口：因为IP已改变，原来映射的端口也就无需使用了。==远程桌面的端口号是3389，SSH/XFTP的端口号是22。账号密码不变==

- 连接远程桌面：端口是3389。示例：`192.168.192.10:3389` （起作用，但是有些地区被墙的原因，网速慢会连接不上）
- 连接ssh/xftp：端口是22。示例`192.168.192.10:22` 

延迟问题不太方便解决，有的地区连接就很稳定，有的地区可能只能连ssh体会一下龟速网络，所以请学习一下ssh使用。

-----------------分割线----------------

### 6.2 附：

管理员分配zerotier IP给新的服务器：

百度如何获取Network ID，我们之前已经创建过了，所以无需再次创建

在服务器上：

```bash
# 安装zerotier
curl -s https://install.zerotier.com | sudo bash
# 加入指定的"局域网"
sudo zerotier-cli join a09acf023363b091
# 在zerotier官网登录创建该"局域网"ID的用户，允许上一步的服务器加入网络，在列表前面的勾选框里勾选新加入的服务器，然后就可以通过Managed IPs访问了

#尝试了moon。好像也没什么效果，可能有错误
# zerotier-cli orbit 11bdb40555 11bdb40555

#ssh -p 9012 ouc@222.195.151.170
#ssh -p 9011 ouc@222.195.151.170

# ssh -p 9128 ouc-28@222.195.151.170
```



# 7、重装系统（非常不建议）

首先说明：非常不建议自己重装系统。即使要重装系统，也要负责地把需要的各项都装好，不给别人添麻烦。

重装系统后需要把第6部分的内容全部配置好

### 7.1 安装系统

如果真有需要安装系统，比较推荐安装ubuntu16，因为ubuntu16对远程桌面支持比较好。要保证用户名密码与原来设定一致。

### 7.2 ==设置IP==

重新配置IP以便可以远程连接  https://blog.csdn.net/hancoder/article/details/102881903 

### 7.3 ==安装cuda，显卡驱动等==

参考链接 https://blog.csdn.net/hancoder/article/details/86634415

##### 解决重启后显卡驱动失效的问题

刚才显卡驱动还能用，重启一下， `nvidia-smi` 却报错误

```bash
NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver. 
Make sure that the latest NVIDIA driver is installed and running.
# 这是因为重启后系统内核更新，显卡驱动就不匹配到了，但是显卡驱动还在系统某个角落
```

参考解决方案：[https://blog.csdn.net/hangzuxi8764/article/details/86572093](https://link.zhihu.com/?target=https%3A//blog.csdn.net/hangzuxi8764/article/details/86572093)

```bash
# 查看现有nvidia显卡驱动的版本号
ls /usr/src | grep nvidia
# 比如查到了 418.87.00 

sudo apt install dkms
sudo dkms install -m nvidia -v 418.87.00
```



### 7.4 远程内容

- 配置远程桌面： https://blog.csdn.net/hancoder/article/details/102882153 
- 安装ssh以便文件传输： https://blog.csdn.net/hancoder/article/details/102881903 

```BASH
一般只需要进行：
apt-get install openssh-server
service ssh restart

特殊情况可以修改文件：
vim /etc/ssh/sshd_config
将PermitRootLoginwithout-password注释，                                
添加一行： PermitRootLoginyes
```

还需要安装anaconda

# 8、What's-more

### 8.1 为什么要安装小老鼠这个界面？

因为teamviewer总会出现商业版问题，所以无奈选择远程连接的方式，如果你使用时间较长，可以试着连teamviewer使用。

因为ubuntu不好实现远程连接，必须通过安装小老鼠界面间接控制ubuntu。ubuntu16有很好的有解决方案（无奈当初别人装的是18系统），而ubuntu18因为版本原因远程桌面的选择很少。所以尽管小老鼠界面不美观，但还得接着使用。

- ubuntu18配置远程参考此链接的第二个方法https://blog.csdn.net/star2523/article/details/81152890

- 在ubuntu16下可能存在完美的解决方式请参考：https://blog.csdn.net/qq_37674858/article/details/80931254 ， https://www.cnblogs.com/xuliangxing/p/7642650.html

- 原来服务器配置人员的博客：https://blog.csdn.net/zhouxiaowei1120/article/details/80872919

> 注：安桌面的`echo xfce4-session >~/.xsession`命令是向home目录的`.xsession`文件末尾写入xfce4-session

### 8.2 一些其他内容

- 配置环境变量的文件：`~/.bashrc`
- 服务器应只跑实验时使用，调试代码请尽量在个人电脑上线调试好
- 有问题咨询组内
- 以后更新尽量在此github账号更新IP等内容，以便同步。账号即OUCvisionLab，密码可问管理员索要。

### 8.3 linux必备技能

- linux基础指令：https://blog.csdn.net/hancoder/article/details/104304735
- vim基础用法：https://blog.csdn.net/hancoder/article/details/104304509
- ip配置： https://blog.csdn.net/hancoder/article/details/102881903 
- cuda、显卡驱动：https://blog.csdn.net/hancoder/article/details/86634415
- 配置远程桌面： https://blog.csdn.net/hancoder/article/details/102882153
- 安装ssh： https://blog.csdn.net/hancoder/article/details/102881903 
- ftp配置：https://blog.csdn.net/hancoder/article/details/100988807

### 8.3 contact me

QQ：553736044@qq.com
