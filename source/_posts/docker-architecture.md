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

![docker_deamon](docker-deamon.png)

Docker Deamon的架构大致可以分为三部分：Docker Server、Engine和Job。


1、Docker Server

Docker Server在Docker架构中是专门为Docker Client服务的，它的功能是接收并调度分发Docker Client发送的请求。

![docker_server](docker-server.png)

在Docker Deamon启动过程中，Docker Server是第一个完成的。Docker Server通过gorilla/mux创建了一个mux.Router路由器，提供了请求的路由分发功能。在Go语言中，gorilla/mux是一个强大的URL路由器以及调度分发器。创建路由器后，Docker Server为mux.Router中添加路由项，每个路由项由HTTP请求方法、URL和Handler组成。

由于Docker Client通过HTTP协议访问Docker Deamon，故Docker Server创建完mux.Router之后，将Server的监听地址及mux.Router作为参数，创建一个httpSer=http.Serve{}，最终执行httpSer.Serve()请求服务。
 
在服务过程中，Docker Server在listener上接收Docker Client的访问请求。对于每个Docker Client请求，都会创建一个全新的goroutine来服务。在goroutine中，Docker Server首先读取请求内容，然后解析其中的flag参数，接着匹配对应的路由项，随后调用相应的Handler来处理，最后Handler处理完请求之后给Docker Client回复响应。


2、Engine

Engine是Docker架构中的运行引擎，也是其核心模块。Engine存储着大量的容器信息，同时管理着大部分Job的运行。除容器管理外，当Docker Deamon需要自身退出时，Engine还负责完成退出后的所有善后工作。


3、Job

Job是Docker执行工作的最基本单元。Dokcer Deamon每执行一项工作都会呈现为一个Job。例如，创建一个新的容器，这是一个Job；


4、Docker Registry

Docker Registry是一个存储容器镜像（Docker Image）的仓库。容器镜像是容器创建时用来初始化容器rootfs的文件系统内容。Docker Registry将众多容器镜像聚集起来，为Docker Deamon提供镜像服务。

Docker的运行过程中，Docker Deamon会与Docker Registry通信，实现搜索镜像，拉取镜像和上传镜像，所对应的Job分别是search、pull和push。

其中，在Docker架构中，Docker可以拥有公有和私有的Docker Registry。大家熟知的Docker Hub，就是全球最大的公有Registry。同时Docker也允许用户构建本地私有的Registry，使容器镜像的获取在内网完成。


5、Graph

Graph在Docker架构中扮演的角色是容器镜像的保管者。不论是Docker下载的镜像，还是Docker构建的镜像，均由Graph统一管理。由于Docker支持多种不同的镜像存储方式，如aufs、devicemapper和Btrfs等。对Docker而言，同一种类型的镜像被称为一个repository，如名称为ubuntu的镜像都同属一个repository；而同一个repository下的镜像则会因tag存在差异不同。

![docker_graph](docker-graph.png)


6、Driver

Driver是Docker架构中的驱动模块。通过Driver驱动，Docker可以实现对容器执行环境的定制。定制的纬度主要分为网络环境、存储方式以及容器执行方式。在Docker运行生命周期中，并非所有的操作都是针对Docker容器的管理，同时还包括用户对Docker运行信息的获取、对Graph的存储和记录等。所以，为了将仅与Docker容器相关的管理从Docker Deamon的所有逻辑中区分出来，Docker设计了Driver层来抽象不同类型各自的功能范畴。

Docker Driver可以分为以下三类驱动：graphdriver、networkdriver和execdriver。


graphdriver主要用于完成容器镜像的管理，包括从远程Docker Registry上下载镜像并储存，也包括本地构建完镜像后的存储。当用户下载指定的容器镜像时，graphdriver将容器镜像分层存储在本地的指定目录下；同时用户需要使用指定的容器镜像来创建容器时，graphdriver从本地镜像存储目录中获取指定的容器镜像，并按特定的规则为容器准备rootfs；另外，当用户需要通过指定Dockerfile构建全新镜像时，graphdriver会负责新镜像的存储管理。

在graphdriver的初始化过程之前，有4中文件系统或类文件系统的驱动Driver在Docker Deamon内部注册，他们分别是aufs、btrfs、vfs和devmapper。其中，aufs、btrfs以及devmapper用于容器镜像的管理，vfs用于容器volume的管理。Docker在初始化时，优先通过获取系统环境变量“DOCKER_DRIVER”来提取所要使用driver的类型。因此，在之后的所有Graph操作，都使用该driver来执行。

graphdriver的架构如下：

![docker_graphdriver](docker-graphdriver.png)


networkdriver的作用是完成Docker容器网络环境的配置，其中包括Docker Deamon启动时为Docker环境创建网桥；Docker容器创建前为其分配相应的网络接口资源；以及为Docker容器分配IP、端口并与宿主机做NAT端口映射，设置容器防火墙配置等。

networkdriver的架构如下：

![docker_networkdriver](docker-networkdriver.png)


execdriver作为Docker容器的执行驱动，负责创建容器运行时的命名空间，负责容器资源使用的统计与限制，负责容器内部进程的真正运行等等。

![docker_execdriver](docker-execdriver.png)


7、libcontainer

libcontainer是Docker架构中一个使用GOLANG语言设计实现的库，设计初衷是希望可以不依靠任何依赖，直接访问内核中与容器相关的系统调用。

由于libcontainer的存在，Docker可以直接调用libcontainer，而最终操作容器的namespaces、cgroups、apparmor、网络设备以及防火墙规则等。这一系列操作的完成都不需要依赖LXC或者其它包。

libcontainer架构图如下：

![docker_libcontainer](docker-libcontainer.png)

另外，libcontainer提供了一套完整的标准接口来满足上层对容器管理的需求。或者说，libcontainer屏蔽了Docker上层对容器的直接管理。又由于libcontainer使用Golang这种跨平台的语言开发，且本身可以被上层多种编程语言访问，因此Docker在跨平台会走得更好。


8、Docker container

Docker container（Docker容器）是Docker架构中服务交付的最终体现形式。









