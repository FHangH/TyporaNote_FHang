# Docker考试资料



##### 1. 容器

- 容器是一种轻量级、可移植、自包含的软件打包技术，使应用程序可以在几乎任何地方以相同的方式运行



##### 2. 容器虚拟化和传统虚拟化的区别



###### 容器虚拟化

- 启动一般是秒级；仅仅kernel所支持的os，系统支持量单机支持上千个容器，磁盘的使用一般为MB性能接近原生



###### 传统虚拟化

- 启动一般是分钟级，支持linux，windows，mac操作系统，系统支持量一般为几十个 磁盘使用一般为GB性能弱



##### 3. Namespace在容器里功能

- Namespace是命名空间隔离，主要就是将用户空间通namespace技术隔离开，容器内的进程互不影响。共用一个内核



##### 4. Cgroup的功能

- 资源限制
- 优先级分配
- 资源统计
- 任务控制



##### 5. Docker能否在32位的系统里运行

- 不能



##### 6. Docker的核心组件

- 镜像
- 容器
- 仓库



##### 7. 安装的docker版本

- 18.03.1-ce版本



##### 8. 搜索docker镜像nginx

```shell
Docker search nginx
```



##### 9. 下载centos 镜像

```shell
Docker pull centos
```



##### 10. 运行zabbix镜像，打开终端

```shell
Docker run -it zabbix /bin/bash
```



##### 11. 容器不停止，后台运行

- 先按ctrl + p 

- 再按 ctrl + q

  

##### 12. 删除现在所有的镜像

```shell
Docker rmi -f 'docker images -q -a'
```



##### 13. 查看上一个容器的状态

```shell
Docker stats docker 'ps -l -q'
```



##### 14. 查看容器的进程

```shell
Docker top <容器id>
```



##### 15. 查看容器的统计信息

```shell
Docker stats <容器id>
```



##### 16. 查看容器abc的详细信息

```shell
Docker inspect abc
```



##### 17. docker build命令构建镜像

- 通过源代码路径的方式
- 通过标准输入流的方式



##### 18. 运行容器test2使用容器test1的数据卷/date

```shell
Docker run -it --name test1 -v /date:/date nginx /bin/bash

Docker run -it --volumes-from test1 --name test2 nginx /bin/bash
```



##### 19. docker的存储驱动程序

- AUES
- Btrfs
- Device mapper
- OverlayFS
- ZFS
- VFS



##### 20. overlay文件系统读取文件，文件存在镜像里，它的工作过程

- 文件不存在于容器（upperdir）中。overlay/overlay2驱动程序执行一个copy_up操作将文件从镜像（复制lowerdir）到所述容器（upperdir）。容器然后将更改写入容器层中的文件的新副本



##### 21. overlay文件系统里有个目录upperdir里面

- Upperdir是容器的可写数据层，里面装的是对容器的更改内容



##### 22. overlay文件系统里有个目录lowerdir里面

- 里面装的是镜像



##### 23. docker把数据从宿主机挂载到容器的三种方式区别



###### Volumes

- 容器内的数据被存放到宿主机(linux)一个特定的目录下(/var/lib/docker/volumes/)。这个目录只有Docker可以管理，其他进程不能修改。如果想持久保存容器的应用数据，Volumes是Docker推荐的挂载方式。

###### Bind mounts

- 容器内的数据被存放到宿主机文件系统的任意位置，甚至存放到一些重要的系统目录或文件中。除了Docker之外的进程也可以任意对他们进行修改。

###### tmpfs volumes

- 容器的数据只会存放到宿主机的内存中，不会被写到宿主机的文件系统中，因此不能持久保存容器的应用数据。



##### 24. --net选项后可跟参数

- None
- host
- bridge
- overlay
- macvlan



##### 25. 项目需要多个容器交流，使用哪个网络

- Docker overlay网络



##### 26. 容器test2去链接test1，映射宿主机的80端口到容器的5000端口

```shell
Docker run -it -v 80:5000 –name test2 --network=container：test1 centos /bin/bash
```



##### 27. Orchestration

- 编排（Orchestration），描述了自动配置、协作和管理服务的过程。



##### 28. Orchestration的分类

- Docker Compose
- Docker Machine
- Docker Swarm



##### 29. compose 使用的步骤

1. 使用dockerfile定义你的应用依赖的镜像
2. 使用docker-compose.yml定义你的应用具有的服务
3. 通过docker-compose up命令创建并运行应用



##### 30. swarm的调度模块的第一阶段，过滤器几种

​	5种

- Constraints，约束过滤器
- Affnity，亲和性过滤器
- Dependency，依赖过滤器
- Health filter，会根据节点状态进行过滤
- Ports filter，会根据端口的使用情况过滤



##### 31. k8s的全称?古希腊话什么意思

- 全称是Kubernetes，在古希腊话中是舵手的意思



##### 32. 编写dockerfile，实现开启容器查看/目录，且能复写查看/mnt目录

```shell
#Vim dockerfile

FROM centos

RUN ls /

CMD [“ls”，”/mnt”]
```



##### 33. 简述Docker跨主机通信的常见网络解决方案

- Overlay: 基于VXLAN封装实现Docker原生Overlay网络
- Macvlan: Docker主机网卡接口逻辑上分为多个子接口，每个子接口标识一个VLAN。容器接口直接连接Docker主机
- 网卡接口: 通过路由策略转发到另一台Docker主机



##### 34. 综合

- Swarm集群操作题，数据中心服务器已存在centos7+Docker1.13.2结点

- 192.168.206.131/24(swarm_leader), 

- 192.168.206.132/24(node1), 

- 192.168.206.132/24(node1), 

- 192.168.206.132/24(node1)。

- 写出简要命令实现如下swarm集群功能。

- 1. 将结点swarm_leader配置swarm管理结点，其余结点作为普通从属结点。

     ```shell
     在192.168.206.131中
     docker swarm init --advertise-addr 192.168.206.131
     
     在192.168.206.132中
     docker swarm join --token <192.168.206.131创建swarm时生成的token值>
     或
     docker swarm join \
     
     ```

     

  2. 在swarm_leader结点运行名为“web_serv”的service,副本为3个，并设置leader结点不运行副本。

     ```shell
     在swarm_leader结点运行
     docker service scale web_serv=3
     docker node update --availbility drain swarm-manager
     ```

     

  3. 删除此“web_serv”,结点node1离开此swarm集群

     ```shell
     docker service rm web_serv
     docker node rm --force node1
     ```

     
