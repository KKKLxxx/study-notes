# Docker简介

## 一、Docker是什么

Docker是一个容器化平台，它以容器的形式将程序及其所有依赖项打包在一起，以确保程序在任何环境中正常运行

## 二、Docker的作用

Docker主要的应用场景就是，在大规模部署应用时，需要统一各种环境，比如MySQL、Redis等，并且各种组件可能需要升级，如果有上百台机器，一个一个去操作是很麻烦的。Docker可以将一个环境打包成镜像，并上传到仓库，在其他机器上可以直接从仓库中获取新镜像，然后实例化为容器，并直接运行。对于旧的镜像，直接删除即可。大大简化了应用的部署

## 三、Docker与虚拟机的区别

虚拟机运行时需要自己虚拟化一套完整的硬件，并在虚拟硬件上运行操作系统，再在操作系统中运行应用

Docker免去了硬件虚拟化，直接调用宿主机的硬件资源。镜像中只保留了程序本身及其运行所需要的依赖

### Docker的优点

- 资源占用少、扩展性高：因为免去了硬件虚拟化这一步骤
- 运行更快：因为直接调用宿主机的硬件资源
- 成本低：一台服务器可以运行多个容器
- 快速回滚：若需回滚可直接实例化之前的镜像

### 虚拟机的优点

虚拟机能够提供系统级别的隔离，而Docker只提供了进程级别的隔离，所以虚拟机相对Docker更加安全（相当于虚拟机中毒不会影响宿主机，但Docker中病毒会影响宿主机）

## 四、Docker的组成

主要分为三部分：

- 仓库（Repository）
- 镜像（Image）
- 容器（Container）

仓库用于存储镜像，一个镜像可以创建多个容器

## 五、Docker的工作模式

Docker采用CS工作模式，即Client-Server。在Docker Client中来输入Docker的各种命令，而这些命令会传送给在Docker宿主机上运行的守护进程，由守护进程负责实现Docker的各种功能

