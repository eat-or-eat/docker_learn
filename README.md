> 一些简单的指令记录

为什么要用docker？

1.docker可以创建不同的环境供开发人员使用

2.docker打包好镜像后方便多/不同环境测试部署



# 一.下载安装和一些操作

## 1.下载安装

[官方安装教程](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker)

1. nvidia-docker2和nvidia-docker1的安装方法不一样，上面是nvidia-docker2的

## 2.常用的操作

### ①拉取镜像

[nvidia/cuda Tags | Docker Hub](https://hub.docker.com/r/nvidia/cuda/tags?page=1&ordering=-name)

找到对应的镜像有指令可以复制粘贴使用

### ②启动/进入镜像

sudo docker ps  -a #显示所有镜像，不加-a只显示正在运行的 

启动镜像：

`sudo docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi`

最好启动镜像的时候就映射好端口和挂在目录，不然还要改

`-p host:docker`  # 端口映射：将docker的端口映射到host上

`-v host:docker`  # 将host的目录挂在到docker上，可以共用文件；路径必须是绝对路径

进入镜像：

sudo docker exec -it 775c7c9ee1e1 /bin/bash  # 进入镜像 775c7c9ee1e1 为容器ID

### ③修改映射端口/挂载目录

[docker修改映射端口](https://blog.csdn.net/wkh___/article/details/114879500)

[docker修改挂载目录](https://blog.csdn.net/zedelei/article/details/90208183)

### ④删除docker

```
# 查询相关软件包
dpkg -l | grep docker
# 删除对应的包
sudo apt remove --purge docker.io
...
```



# 二.docker下安装Anaconda

## 1.安装Anaconda

`apt-get install wget `# 有的docker环境没有wget

`wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-5.1.0-Linux-x86_64.sh` # 这里可以选自己需要的anaconda版本，主要的区别就是默认的python，conda，jupyter等软件和工具的版本不同

`bash Anaconda3-5.1.0-Linux-x86_64.sh`  # 安装一路确认

## 2.换源，不然下载太慢

### ①换pip源

`mkdir ~/.pip` # 这个是新建/.pip的隐藏文件。
`vim ~/.pip/pip.conf` # vim打开并编辑内容

将以下内容复制进pip.conf文本

```
[global]
timeout = 6000
index-url = http://mirrors.aliyun.com/pypi/simple/
trusted-host = pypi.tuna.tsinghua.edu.cn
```

### ②换conda源

vim ~/.condarc # 编辑conda源的隐藏文件

将以下内容复制进去

```
channels:
  - https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
  - https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
  - defaults
show_channel_urls: true
```



# 三.配置jupyter notebook/lab

## 1.生成配置文件和密钥

`jupyter notebook --generate-config` # 生成配置文件，系统会提示文件在哪里

```
>>>from notebook.auth import passwd
>>>passwd
Enter passwd:				# 这里输入你的密码
Verify passwd:				#重复输入密码
>>>argon2:$argon2id$v=19$m=10240,t=10,p=8$YiTF3tI7ZWECTbNaa5VZtA$kaU+xbQSsMaZsH98sqoaGg  #复制这段秘钥
```

## 2.修改配置文件

```
vim ~/.jupyter/jupyter_notebook_config.py  # 这里个目录根据上面生成的目录确定
 
其中为了快速搜索定位，会用到下列命令
/App.ip # 回车
/App.open_brower #回车
/App.port #回车
/allow_remote_access #回车
```

```
找到如下几项，取消注释并修改
c.NotebookApp.password ='argon2:$argon2id$v=19$m=10240,t=10,p=8$YiTF3tI7ZWECTbNaa5VZtA$kaU+xbQSsMaZsH98sqoaGg' #秘钥
c.NotebookApp.ip='*' # *允许任何ip访问
c.NotebookApp.open_browser = False # 默认不打开浏览器
c.NotebookApp.port =8888 # 可自行指定一个端口, 访问时使用该端口
allow_remote_access = True
```



## 3.启动环境

`nohup jupyter lab --allow-root&`