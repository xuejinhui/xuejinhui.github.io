---
title: "Docker安装及常用命令" 
catalog: true
date: 2017-05-25 20:23:13
subtitle: "Docker利用yum源安装以及常用命令"
header-img: "docker-install-header.png"
tags:
- Docker
- Docker-yum-Install
- Blog
catagories:
- Docker
---

# Docker的安装以及常用命令
---
  本篇文章将简单介绍docker、如何安装以及常用的docker命令。
## 1、Docker的安装
---
### Linux环境(centos)
---
#### &ensp;&ensp;系统要求
---
```bash
    1、64位版本CentOS 7,并且要求内核版本不低于3.10
    2、卸载旧版本：sudo yum remove docker
```
#### &ensp;&ensp;使用yum源安装
---
```bash
    ##先安装依赖包
    $ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
    
    ##添加docker yum源
    $ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    
    ##安装
    $ sudo yum makecache fast
    $ sudo yum install docker-ce
```
#### &ensp;&ensp;启动Docker CE
---
```bash
    $ sudo systemctl enable docker
    $ sudo systemctl start docker
```
## 2、Docker常用命令
---
### 基础类
---
#### 查看docker信息
---
```bash
    ##查看docker版本
    $ sudo docker version
    
    ##显示docker系统的信息
    $ sudo docker info
    
    ##显示某个容器的日志信息
    $ sudo docker logs [CONTAINER]
    
    ##故障检查
    $ sudo service docker status
    
    ##启动/关闭docker
    $ sudo service docker start/stop
```
### 容器类
---
#### 查看容器信息
---
```bash
    ##查看当前运行的容器
    $ sudo docker ps
    
    ##查看所有的容器
    $ sudo docker ps -a
    
    ##查看所有的容器的id和信息
    $ sudo docker ps -a -q
    
    ##查看容器或镜像的参数，默认返回的json格式
    $ sudo docker inspect [CONTAINER]
```
#### 容器同步命令
---
```bash
    ##保存对容器的修改
    $ sudo docker commit
    
    ##对比容器的修改
    $ sudo docker diff
    
    ##进入到一个正运行的容器
    $ sudo docker attach [CONTAINER]
```
#### 容器操作命令
---
```bash
    ##创建一个容器 -i:保持标准输入 -t:terminal -d:后台运行
    $ sudo docker create -i -t -d --name CONTAINER_NAME IMAGE_NAME
    
    ##启动一个容器 -d:后台运行 -p:映射端口
    $ sudo docker run -d -p 127.0.0.1:8080:80 [CONTAINER_NAME]
    
    ##创建并启动一个容器 -rm:运行完即删除
    $ sudo docker run -i -t -rm --name [CONTAINER_NAME] [IMAGE_NAME]
    
    ##运行一个新容器，同时为它命名、端口映射、文件夹映射
    $ sudo docker run --name [CONTAIN_NAME] -p 9023:22 -d -v /host/path:/contain/path -v /host/path1:/contain/path1 [IMAGE_NAME]
    
    ##一个容器连接到另一个容器
    $ sudo docker run -i -t -d --link [LINK_CONTAINER_NAME] --name [CONTAIN_NAME] [IMAGE_NAME] [CMD] 
    
    ##删除容器
    $ sudo docker rm [CONTAINER_ID]
    
    ##删除所有容器
    $ sudo docker rm `docker ps -a -q`
    
    ##容器随系统自启动
    $ sudo docker run --restart=always
    
    ##容器保存为镜像
    $ sudo docker commit [CONTAINER_ID] [IMAGE_NAME]
    
    ##启动/暂停/重启容器
    $ sudo docker start|stop|restart [CONTAINER_ID]
    
    ##暂停|恢复容器
    $ sudo docker pause|unpause [CONTAINER_ID]
    
    ##杀死一个或多个指定容器
    $ sudo docker kill -s KILL [CONTAINER_ID]|`docker ps -a -q`
    
    ##交互式进入容器
    $ sudo docker exec -it [CONTAIN] bash
    
    ##查看容器的root的用户密码
    $ sudo docker logs [CONTAIN] 2>&1 | grep '^User:' | tail -n1
    
    ##导入导出容器
    $ sudo docker import [*]
    $ sudo docker export [CONTAINER_ID] > [/path/filename]
```
  
### 镜像类
---
#### 远程镜像
---
```bash
    ##登录docker
    $ sudo docker login
    
    ##拉取远程最新镜像
    $ sudo docker pull [IMAGE_NAME]:[VERSION]
    
    ##标记本地镜像
    $ sudo docker tag [CONTAIN_ID] [IMAGE_NAME]
    
    ##将镜像推送至远程仓库,默认是Docker Hub
    $ sudo docker push 
```
#### 本地镜像
---
```bash
    ##列出所有镜像
    $ sudo docker images
    
    ##列出本地名为xx的所有镜像
    $ sudo docker images [IMAGE_NAME]
    
    ##查看指定镜像的创建历史
    $ sudo docker history [IMAGE_ID]
    
    ##本地移除镜像
    $ sudo docker rmi [IMAGE_ID]
    
    ##移除所有本地镜像
    $ sudo docker rmi `docker images -a -q`
    
    ##指定镜像保存为tar
    $ sudo docker save -o [/path/name] [IMAGE_NAME]:[VERSION]
    
    ##载入镜像
    $ sudo docker load -i [/path/name]
    
    ##构建自己的镜像
    ##注意：docker在build镜像的时候会打包[/path/Dockerfile]所有文件作为镜像的上下文使用
    $ sudo docker build -t [IMAGE_NAME] [/path/Dockerfile]
```