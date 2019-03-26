For theory and texture group only.

为增加查阅体验与防止后续服务器内容的更改，

本文档同步在github上，请前往观看 https://github.com/OUCvisionLab/ServerGuide

本次更新内容：

- 会让3个人左右共同使用和维护一台专属服务器，也留下了1,2台机动的服务器。
- 创建了大家主要用的python2.7和python3.6，不再需要 ~~conda create -n 你名字~~ 这种步骤了，主要是考虑到了有的同学不熟悉anaconda命令，也节省了一些服务器内存，避免了同学们在新使用某个服务器时都需要重建环境的漫长等待。（有的服务器可能因为在使用还未来得及安装，需要可联系网管）
- 与原有服务器用法不冲突，可以继续参考之前的用法。所以你可能只需要看3.3的两行代码即可关闭文档。
- 考虑到留学生，所以采用了中英结合的书写方式

[TOC]

# 1、服务器列表Server list

| IP              | 端口号Port | capacity                                       | user name | password | Note         |
| --------------- | ---------- | :--------------------------------------------- | --------- | -------- | ------------ |
| 222.195.151.170 | 9013       | RAM:128G, CPU:2.3GHz*40, GPU:none              | ouc-13    | b301     | Matlab 2018b |
| 222.195.151.170 | 9015       | RAM:128G, CPU:2.3GHz*40, GPU:TESLA K40c 11G *1 | ouc-15    | b301     |              |
| 222.195.151.170 | 9028       | RAM:32G, CPU:3.5GHz*8, GPU:1080Ti 11G *2       | ouc-28    | b301     |              |
| 222.195.151.170 | 9029       | RAM:32G, CPU:3.5GHz*8, GPU:2080 8G *2          | ouc-29    | b301     |              |
| 222.195.151.66  | 9010       | RAM:32G, CPU:3.3GHz*4, GPU:TITAN X 12G         | ouc-10    | b301     |              |
| 222.195.151.66  | 9018       | RAM:32G, CPU:3.5GHz*8, GPU:1080Ti 11G *2       | ouc-18    | b301     |              |
| 222.195.151.66  | 9019       | RAM:32G, CPU:3.5GHz*8, GPU:1080Ti 11G *2       | ouc-19    | b301     |              |

# 2、登录服务器login

**2.1 Find remote connection**
![](https://raw.githubusercontent.com/FermHan/tuchuangsimi/master/20190325172634.png?token=AkTVJfvkXHdCyhSbXbtS6iokfCOR6xZNks5cmJ8MwA%3D%3D)

**2.2 Type in IP:port**
![](https://raw.githubusercontent.com/FermHan/tuchuangsimi/master/20190325172652.png?token=AkTVJYakMJzCIiVJDfjlIcg5KLcv0mctks5cmJ8owA%3D%3D)

**2.3 Type in username and password**

![](https://raw.githubusercontent.com/FermHan/tuchuangsimi/master/20190325135817.png?token=AkTVJVynPSWj1sb4ZEbO8wRyjpg_8P4cks5cmG46wA%3D%3D)

**2.4 Once in the system, modify someone.txt to note your name and usage time.And you can put your personal files in directory  /home/ouc-xx/**
![](https://raw.githubusercontent.com/FermHan/tuchuangsimi/master/20190325144734.png?token=AkTVJRoQTQFVopFyApR5WI9oEZziwdXtks5cmHnIwA%3D%3D)

**2.5 Before you run the code, please type in  ` nvidia-smi ` in the terminal to make sure there's no another user.**(在你正式跑代码之前，请输入`nvidia-smi`查看有没有其他用户在跑程序（通过红框部分看）)。

![](https://raw.githubusercontent.com/FermHan/tuchuangsimi/master/20190325150409.png?token=AkTVJdMwtfgMAto3CRd4hvoScKzyrl_kks5cmH2rwA%3D%3D)



# 3、Use python and tensorflow

**3.1 First and Foremost：`nvidia-smi`**

**3.2 Both of python3.6 and python2.7 are ready and they can satisfy most of your needs.** 
   （python3.6和python2.7是已经准备好的环境，这两个环境能满足大家绝大多数需求。）

> If you need other special environments, you can Google it.
> (如果需要其他环境可百度自行安装)

## 最主要用法

**3.3 python(2.7 and 3.6) is available in the following three ways.**

> （可以通过3种方法启动指定python环境（现存2.7或3.6，满足绝大部分人需要））

- **==Method 1:==**(proposed)

```python
source deactivate  #退出原有环境，或deactivate python36
source activate python27 #大多数人只需要知道前两命令即可，接下来可以不用看了
#or
source avtivate python36
```

```python
# How to install modules:
conda install tensorflow-gpu

#How to check modules installed:
conda list
```

当然你也可以像以前一样，为了特殊需要创建特殊的python环境

```python 
# View existing conda list
conda info -e

# How to create a new environment
conda create -n YOURENAME python=PYTHONVERSION

# How to install special version tensorflow
conda install --channel https://conda.anaconda.org/anaconda tensorflow-gpu=VERSION

# How to delete your personal environment
conda remove -n YOURNAME --all
```



- **Method 2:**

Type in `anaconda-navigator` in the terminal to open NAVIGATOR, and choose python from NAVIGATOR.

In addition, you can check your modules installed 

![](https://raw.githubusercontent.com/FermHan/tuchuangsimi/master/20190325170518.jpg?token=AkTVJfCdox_AKmenkZWtbZejKnxxdoVMks5cmJoSwA%3D%3D)

- **Method 3:cd to the specified directory.**

You can find your ANACONDA in the directory such as: `/home/ouc/Anaconda3`.

You can find your personal python in: `Anaconda3/env/YOURNAME/python`.You can open terminal and type in `/home/Anaconda3/env/YOURNAME/python` to enter YOURNAME python.

And you may need to change the python path from relative path to absolute path in some sh files.

# 4、What's more

4.1 Maybe you want to install Anaconda2, but to be honest, it's not usually used. Because python2.7 has been involved in Anaconda3. If you think about it, What is noteworthy is that when you install anaconda2,

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

4.2 Some environment variables are configured in `~/.bashrc`

4.3 please debug your code on your PC to save server resources.

4.4 If your server resources are insufficient, please contact HAN. There may be servers unallocated for you.
