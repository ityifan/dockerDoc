# Docker

------



# 1.常用的docker命令



```
cd "C:\Program Files\Docker\Docker"

DockerCli.exe -SwitchDaemon
```



```powershell
docker 

docker info
// 列出当前容器
docker container ps 
// 列出当前所有容器包括停止的
docker container ps -a
// 查看当前 image列表
docker image ls
//删除iamge 
docker image rm [name or id前几位]
//列出当前启动容器列表
docker container ls
//包括已经停运的容器
docker container ps -a
```

创建容器 

```powershell
 docker container run nginx
```

停止一个容器

```shell
docker container stop [name or ID前几位]
// 停止多个容器
docker container ps -aq 
docker container stop $(docker container ps -qa)
```

删除一个容器

```powershell
docker rm [name or ID前几位]
// 删除多个容器
docker rm $(docker ps -qa)
```

docker中不能直接删除一个正在运行的容器

# 2.容器的两种模式

## 1.attached模式（前台执行模式）

```powershell
//启动一个nginx服务将端口映射到外部80端口
docker run -p 80:80 nginx
```

win 的 ctrl+c不会停止一个容器 linux中 ctrl+c 会停止容器的运行

## 2.detached模式（后台执行模式）

```powershell
docker run -d -p 80:80 nginx
//重新attach到容器中
docker attch fa3(容器id)
// detached模式下 查看某个容器日志
docker logs (容器Id)
//动态跟踪logs
docker logs -f (容器Id)
```

d==>detached

# 3.容器交互式模式

## 1.直接创建

首先 我们创建出一个可交互的ubuntu

```powershell
docker container run -it ubuntu sh
// -it 交互式模式 
// sh 执行sh 命令
```

![image-20220903151606174](\iamge\image-20220903151606174.png)

如此便可得到一个可交互的ubuntu的系统

通过exit退出交互式

exit  退出之后 此容器也会停止运行



## 2.进入detached模式下的交互式模式



首先启动一个nginx服务

```powershell
docker container run -d -p 80:80 nginx
```



![image-20220910172019889](iamge\image-20220910172019889.png)



```powershell
docker exec -it [ID] sh
// exec 执行
//-it 交互式 
// sh 以shell形式
```

![image-20220910174920937](iamge\image-20220910174920937.png)

此时 exit  我们的容器并不会 停止运行

![image-20220910175103556](iamge\image-20220910175103556.png)



```powershell
docker top c2f
// 查看此容器启动的进程
```



![image-20220910192538874](iamge\image-20220910192538874.png)



## 3.docker container run 背后发生了什么？

```powershell
$ docker container run -d --publish 80:80 --name webhost nginx

// -d 后台执行
// publish ====> -p 端口映射
// name webhost 给镜像一个名字 如果不写 也会随机一个名字
```

- 1. 在本地查找是否有nginx这个image镜像，但是没有发现
- 1. 去远程的image registry查找nginx镜像（默认的registry是Docker Hub)
- 1. 下载最新版本的nginx镜像 （nginx:latest 默认)
- 1. 基于nginx镜像来创建一个新的容器，并且准备运行
- 1. docker engine分配给这个容器一个虚拟IP地址
- 1. 在宿主机上打开80端口并把容器的80端口转发到宿主机上
- 1. 启动容器，运行指定的命令（这里是一个shell脚本去启动nginx）

# 4.镜像的创建管理和发布



## 1.镜像的获取

- pull from `registry` (online) 从registry拉取
  - public（公有）
  - private（私有）
- build from `Dockerfile` (online) 从Dockerfile构建
- load from `file` (offline) 文件导入 （离线）

![image-20220910195225113](iamge\image-20220910195225113.png)



```powershell
docker image pull nginx
```

## 2.镜像的获取查看和删除

![image-20220910213436993](iamge\image-20220910213436993.png)

拉取固定版本

```
docker image pull nginx:1.20.0
```



从quay上拉去镜像

```
docker image pull quay.io/centos7/nginx-116-centos7 
```



查看docker image镜像跟多信息

```powershell
docker image inspect [ID]
```

![image-20220910220931855](iamge\image-20220910220931855.png)

可以查看到很多配置

![image-20220910221516367](iamge\image-20220910221516367.png)

镜像的删除

```powershell
docker image rm [ID]
```

注意:有正在运行的容器 是不能删除镜像的

要删除需要 先停止该镜像 之后再删除容器 最后再删除镜像

```powershell
docker container stop [ID]
docker container  ps -a
docker container rm [ID]
docker image rm [ID]
```

## 3.docker镜像的导出

1.导出

```powershell
docker image save nginx -o nginx.image
```

![image-20220910223152911](iamge\image-20220910223152911.png)

2.导入

```powershell
docker image load -i .\nginx.image
```

![image-20220910223833720](iamge\image-20220910223833720.png)



## 4.dockerfile介绍

- Dockerfile是用于构建docker镜像的文件
- Dockerfile里包含了构建镜像所需的“指令”
- Dockerfile有其特定的语法规则

## 5.镜像的构建和分享

以python为例子

构建Dockerfile文件

```dockerfile
FROM ubuntu:20.04
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y python3.9 python3-pip python3.9-dev
ADD hello.py /
CMD ["python3", "/hello.py"]
```

创建hello.py

```python
print("hello docker")
```

```powershell
docker image build -t hello .
// -t tag标签
//. 代表当前文件夹
```

![image-20220911143954963](iamge\image-20220911143954963.png)

image重命名

```powershell
docker image tag hello leon9907/hello:1.0
```

image上传dockerhub 

1.登录dockerbub

```powershell
docker login
```

2.上传image镜像

```powershell
 docker image push leon9907/hello:1.0
 //名字必须以自己id开头 冒号后标注tag
```

![image-20220911144828659](iamge\image-20220911144828659.png)

3.镜像的拉取

```powershell
docker pull leon9907/hello:1.0
```

![image-20220911145916494](iamge\image-20220911145916494.png)

## 6.通过commit创建镜像

```powershell
docker container commit fcb leon9907/nginx
docker container commit [ID] [dockerhub账号]/[镜像名称]
```

# 5.dockerfile完全指南

## 1.基础镜像的选择 (FROM)

基本原则

- 官方镜像优于非官方的镜像，如果没有官方镜像，则尽量选择Dockerfile开源的
- 固定版本tag而不是每次都使用latest
- 尽量选择体积小的镜像



dockerfile 常用常用语法(COPY && ADD)

```dockerfile
COPY --复制所在文件夹文件
ADD  --复制所在文件夹文件 如果为tar.gz或者其他压缩格式 会自动解压
WORKDIR --类似于linux中cd  切换路径  如果目标路径不存在 会自动创建一个目录
```

```dockerfile
FROM python:3.9.5-alpine3.13
COPY hello.py /app/hello.py
```

```dockerfile
FROM python:3.9.5-alpine3.13
ADD hello.tar.gz /app/
```

```dockerfile
FROM python:3.9.5-alpine3.13
WORKDIR /app
COPY hello.py hello.py
```

2.构建参数和环境变量(ARG && ENV)