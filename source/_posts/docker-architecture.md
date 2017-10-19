---
title: "[Docker] Docker架构" 
catalog: true
date: 2017-05-25 20:23:13
subtitle: "Docker架构详细讲解"
header-img: "docker-architecture-header.png"
tags:
- Docker
catagories:
- Docker
---

# Docker总体架构
---
Docker是一个开源软件项目，让应用程序部署在容器下工作，实现轻量级的操作系统虚拟化解决方案。<br/>

## Docker简介
---
Docker的基础是Linux容器(LXC)技术，在LXC基础上Docker进行再次封装，提供了一个额外的软件抽象层以及操作系统层虚拟化的自动管理机制。<br>
Docker利用Linux内核中的资源分脱机制，例如cgroups和命名空间namespace提供了资源隔离和安全保障，来创建独立的软件容器（实则为进程）。由于Docker是通过操作系统层的虚拟化实现隔离，所以Docker容器在运行时，并不像虚拟机（VM）需要额外的操作系统资源开销。<br>
![vm_struct](vm_struct.png)<br>
![docker_struct](docker_struct.png)<br>

## Docker总架构图
---
![docker_architecture](docker_architecture.png)<br>
Docker总架构图如上图。主要包括
``` javascript
    Docker Client:用户通过Docker Client发起容器的管理请求，发送至Docker Deamon。
    Docker Deamon:作为Docker的主体部分，首先需要提供server功能使其能接收Docker Client的请求。所有的请求任务都是通过Docker Deamon内部中的Engine来进行处理，且每一项都以Job的形式存在。
    Docker Registry:是一个Docker镜像存储仓库。
    Graph:已下载容器镜像的储存管理。
    Driver:Docker的驱动模块，通过Driver驱动，Docker可以实现对容器执行环境的定制。
    Libcontainer:是一个使用Golang语言设计实现的库，该库可以不依靠任何依赖，直接访问Linux内核中与容器相关的API。
    Docker Container:Docker容器。
```
(1)、Docker Client

Docker Client是Docker架构中用户与Docker Deamon建立通信的客户端。在客户端机器上，用户可以使用可执行文件docker作为Docker Client，发送多个管理container的请求至Docker Deamon。<br>
Docker Client可以通过以下三种方式和Docker Deamon建立通信：tcp://host:port，unix://path_to_socket和fd://sockerfd。与Docker Deamon建立连接并传输参数时，Docker Client可以通过命令行Flag参数的形式,设置安全传输协议(TLS)的相关参数，保证传输的安全性。<br>
Docker Client发送容器管理请求后，请求由Docker Deamon接收并处理，当Docker Client接收到返回的请求响应并作简单处理后，Docker Client一次完整性的生命周期就此结束。<br>

<br>
(2)、Docker Deamon

Docker Deamon是Docker架构中一个常驻在后台的系统进程。其主要作用有：<br>
&ensp;&ensp;a.接收并处理Docker Client发起的请求<br>
&ensp;&ensp;b.管理所有的容器<br>
Docker Deamon运行时，会在后台启动一个server，server负责接收Docker Client发送的请求。接收请求后server通过路由与分发调度，找到对应的handler来处理请求并返回。<br>
<br>

![docker_deamon](docker_deamon.png)

Docker Deamon的架构大致可以分为三部分：Docker Server、Engine和Job。

1、Docker Server

Docker Server在Docker架构中是专门为Docker Client服务的，它的功能是接收并调度分发Docker Client发送的请求。

![docker_server](docker_server.png)

在Docker Deamon启动过程中，Docker Server是第一个完成的。Docker Server通过gorilla/mux创建了一个mux.Router路由器，提供了请求的路由分发功能。在Go语言中，gorilla/mux是一个强大的URL路由器以及调度分发器。创建路由器后，Docker Server为mux.Router中添加路由项，每个路由项由HTTP请求方法、URL和Handler组成。

由于Docker Client通过HTTP协议访问Docker Deamon，故Docker Server创建完mux.Router之后，将Server的监听地址及mux.Router作为参数，创建一个httpSer=http.Serve{}，最终执行httpSer.Serve()请求服务。
 
在服务过程中，Docker Server在listener上接收Docker Client的访问请求。对于每个Docker Client请求，都会创建一个全新的goroutine来服务。在goroutine中，Docker Server首先读取请求内容，然后解析其中的flag参数，接着匹配对应的路由项，随后调用相应的Handler来处理，最后Handler处理完请求之后给Docker Client回复响应。

2、Engine



