---
title: Docker入门
date: 2024-05-17 17:14:13
tags: Docker
categories:
    笔记
---

# Docker入门教程

## Ubuntu Docker安装步骤

```shell
# 卸载旧的源
sudo apt-get remove docker docker-engine docker.io containerd runc
# 更新软件包
sudo apt update 
sudo apt upgrade
# 安装docker依赖
sudo apt install apt-transport-https ca-certificates curl software-properties-common gnupg lsb-release
# 添加Docker官方GPG密钥
curl -fsSL curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# 添加Docker软件源
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# 安装Docker
apt-get install docker-ce docker-ce-cli containerd.io
# 查看Docker版本
sudo docker version
# 启动Docker
sudo systemctl start docker
# 查看Docker运行状态
sudo systemctl status docker
# 允许非Root用户执行docker 命令
# 1.添加docker用户组
sudo groupadd docker
# 2.将当前用户添加到用户组
sudo usermod -aG docker $USER
# 3.使权限生效
newgrp docker 
```

```shell
# 卸载docker
# 1.卸载依赖
sudo apt-get remove docker docker-engine docker.io containerd runc
# 2.删除资源
rm -rf /var/lib/docker   # docker的默认工作路径
```

## 配置阿里云镜像加速

```shell
sudo mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://wgg82qbx.mirror.aliyuncs.com"]
}
EOF

sudo systemctl daemon-reload

sudo systemctl restart docker
```

## Docker命令

![image-20240516121849040](https://mzy777.oss-cn-hangzhou.aliyuncs.com/img/image-20240516121849040.png)

![在这里插入图片描述](https://mzy777.oss-cn-hangzhou.aliyuncs.com/img/61f881d798344b0e8ded77eb5181bc9e.png)

### Docker容器生命周期管理

```shell
# 创建并运行容器
docker run --name 容器名 镜像名
-d #后台启动
-it /bin/bash   #以交互方式
-p 宿主机端口：容器端口  # 暴露端口
ctrl + p + q 切换终端
# 创建但不启动容器
docker create --name 容器名 镜像名:tag
# 启动一个或多个已经被停止的容器
docker start  容器名
# 停止一个运行中的容器
docker stop 容器名
# 重启容器
docker restart 容器名
# 杀掉一个运行中的容器
docker kill 容器名
# 删除一个或者多个容器
docker rm 容器名
# 暂停容器中所有的进程
docker pause 容器名
# 恢复容器中所有的进程
docker unpause 容器名
# 进入容器的终端
docker exec -i -t myhello-docker /bin/bash
```

不关闭容器，退回宿主机。这样的话，容器还在后台运行。

![img](https://mzy777.oss-cn-hangzhou.aliyuncs.com/img/1729889-20240419153307055-1060946887.png)

### 容器操作

```shell
# 列出容器
docker ps 
# 查看容器中运行的进程信息
docker top
# 获取容器日志
docker logs 容器名
# 显示容器资源的使用情况，包括：CPU、内存、网络I/O等
docker stats
# 从容器创建一个新的镜像
docker commit -a "提交的镜像作者" -m "提交的文字说明" 容器名/容器ID 镜像名：tag
# 容器与主机之间的数据拷贝
docker cp 主机路径 容器ID:路径
# 检查容器里文件结构的更改
docker diff 容器名
# 查看容器详细信息
docker inspect 容器id
```

### 镜像仓库

```shell
# 登陆到一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub
docker login
# 登出一个Docker镜像仓库
docker logout
# 从镜像仓库中拉取或者更新指定镜像
docker pull 镜像名:tag
# 将本地的镜像上传到镜像仓库,要先登陆到镜像仓库
docker push 镜像名:tag
# 从Docker Hub查找镜像
docker search 镜像名
```

### 本地镜像管理

```shell
# 列出本地镜像
docker images
# 删除本地镜像
docker rmi 镜像名
# 标记本地镜像，将其归入某一仓库
docker rmi -f $(docker images -aq) # 删除全部镜像
docker tag 镜像名:tag 用户名/仓库:tag
# 使用 Dockerfile 创建镜像
docker build -t 镜像名:tag 路径
# 查看指定镜像的创建历史
docker history 镜像名:tag
# 将指定镜像保存成 tar 归档文件
docker save -o  文件名.tar  镜像名:tag
# 导入使用 docker save 命令导出的镜像
docker load -i 镜像名:tag
# 从归档文件中创建镜像
docker import 文件名.tar
```

### 版本信息

```shell
# 显示 Docker 系统信息，包括镜像和容器数
docker info
# 显示 Docker 版本信息
docker version
```

## Docker容器数据卷

**什么是容器数据卷？**

容器数据卷（Container Data Volumes）是Docker管理的一种特殊类型的存储区域，它为容器提供了一种**持久化数据**、**共享数据**以及与宿主机或其他容器之间进行**数据交互**的有效方式。

>如果数据都在容器中，那么容器删除，数据就会丢失！MySQL的容器删除 = 删库 => 跑路
>
>容器之间可以有一个数据共享的技术。
>
>docker容器中产生的数据，同步到本地。这就是卷技术！即容器的目录挂载到Linux上。

```shell
# 挂载目录，并运行镜像进入容器，在容器内部的/home目录下创建测试文件
# docker run -it -v 主机目录:容器目录
docker run -it --name ubuntu01 -v /home/mzy/ubuntu:/home ubuntu /bin/bash
```

**匿名挂载和具名挂载**

```shell
#匿名挂载 -v 容器目录
docker run -d -P --name nginx01 -v /etc/nginx nginx
#具名挂载(不能加路径)
docker run -d -P --name nginx02 -v 具体卷名:/etc/nginx nginx
# 查看所有卷的情况
docker volume ls
#查看具体卷名信息
docker volume inspect 具体卷名
# 没有具体路径的卷名都在`/var/lib/docker/volumes`下
```

**容器间挂载**

```shell
# --volumes-from 父容器
docker run -it --name docker02 --volumes-from docker01 ubuntu
```

![image-20240517161642812](https://mzy777.oss-cn-hangzhou.aliyuncs.com/img/image-20240517161642812.png)

### 实战：mysql同步数据

```shell
# 获取镜像
docker pull mysql:5.7
# 运行容器，需要做数据挂载
# 启动mysql，需要配置密码

-d 后台启动
-p 端口映射
-v 数据卷挂载
-e 环境配置
--name 容器名

docker run -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7
```

## Dockerfile

![image-20240516122104731](https://mzy777.oss-cn-hangzhou.aliyuncs.com/img/image-20240516122104731.png)

>**Dockerfile**是用来构建Docker镜像的文件，是由一条条构建镜像所需的指令和参数构成的脚本。
>
>构建三部曲：
>
>1. 编写Dockerfile文件
>2. docker build命令创建镜像
>3. docker run依赖镜像运行容器实例

![image-20240516122520985](https://mzy777.oss-cn-hangzhou.aliyuncs.com/img/image-20240516122520985.png)

| **指令**   | **说明**                                      |
| ---------- | --------------------------------------------- |
| FROM       | 设置镜像使用的基础镜像                        |
| MAINTAINER | 设置镜像的作者                                |
| RUN        | 编译镜像时运行的脚本                          |
| CMD        | 设置容器的启动命令                            |
| LABEL      | 设置镜像的标签                                |
| EXPOSE     | 设置镜像暴露的端口                            |
| ENV        | 设置容器的环境变量                            |
| ADD        | 编译镜像时复制文件到镜像中                    |
| COPY       | 编译镜像时复制文件到镜像中                    |
| ENTRYPOINT | 设置容器的入口程序                            |
| VOLUME     | 设置容器的挂载卷                              |
| USER       | 设置运行RUN CMD ENTRYPOINT的用户名            |
| WORKDIR    | 设置RUN CMD ENTRYPOINT COPY ADD指令的工作目录 |
| ARG        | 设置编译镜像时加入的参数                      |
| ONBUILD    | 设置镜像的ONBUILD指令                         |
| STOPSIGNAL | 设置容器的退出信号量                          |

![image-20240517163715102](https://mzy777.oss-cn-hangzhou.aliyuncs.com/img/image-20240517163715102.png)

```markdown
CMD：指定这个容器启动的时候要运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT：指定这个容器启动的时候要运行的命令，可以追加命令
```

Dockerfile里的内容：

```dockerfile
FROM node:14-alpine
COPY index.js /index.js
CMD node /index.js
```

```shell
# 创建镜像
#					镜像名：tag 	当前目录
docker build -t hello-world:v1.0.0 .	
# 创建容器并运行容器实例
docker run hello-world
```

## 为什么Docker会比VM虚拟机快

- docker有着比虚拟机更少的抽象层

  >docker不需要Hypervisor（虚拟机）实现硬件资源虚拟化，运行在docker容器上的程序直接使用的都是实际物理机的硬件资源。因此在CPU、内存利用率上docker将会效率上有明显优势。

- docker利用的是宿主机的内核，而不需要加载操作系统OS内核

  >Docker容器共享宿主机的操作系统内核，不需要像虚拟机一样每个实例都运行完整的操作系统。这使得Docker容器在启动和停止时更加轻量级和快速。

|            | Docker容器              | 虚拟机（VM）                |
| ---------- | ----------------------- | --------------------------- |
| 操作系统   | 与宿主机共享OS          | 宿主机OS上运行虚拟机OS      |
| 存储大小   | 镜像小，便于存储与传输  | 镜像庞大（vmdk,vdi 等）     |
| 运行性能   | 几乎无额外性能损失      | 操作系统额外的CPU, 内存消耗 |
| 移植性     | 轻便、灵活，适用于Linux | 笨重，与虚拟化技术耦合度高  |
| 硬件亲和性 | 面向软件开发者          | 面向硬件运维者              |
| 部署速度   | 快速，秒级              | 较慢，10s以上               |
