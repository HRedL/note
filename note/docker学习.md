# docker学习

## 1.镜像/容器/仓库

### 1.1镜像

Docker镜像就是一个只读的模板。镜像可以用来创建Docker容器，一个镜像可以创建很多容器

### 1.2容器

Docker利用容器（Container）独立运行的一个或一组应用。容器适用镜像创建的运行实例

它可以被启动、开始、删除。每个容器都是相互隔离的、保证安全的平台

可以把容器看做是一个减半的Linux环境（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

容器的定义和镜像几乎一模一样，也是一堆层的统一视角，唯一区别在于容器的最上面那一层是可读可写的

### 1.3仓库

仓库是集中存放镜像文件的场所

仓库和仓库注册服务器时由区别的。仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签（tag）

参股分为公开仓库（Public）和私有仓库（Private）两种形式

最大的公开仓库是Docker Hub（https://hub.docker.com）

存放了数量庞大的镜像供用户下载。国内的公开仓库包括阿里云、网易云等

### 1.4理解

Docker本身是一个容器运行载体或称之为管理引擎。我们把应用程序和配置依赖打包形成一个可交付的运行环境，这个打包好的运行环境就似乎image镜像文件。只有通过这个镜像文件才能生成Docker容器。image文件可以看作是容器的模板。Docker根据image文件生成容器的实例。同一个image文件，可以生成多个同时运行的容器实例。

image文件生成的容器实例，本省也是一个文件，称为镜像文件

一个容器运行一种服务，当我们需要的时候，就可以通过docker客户端创建一个对应的运行实例，也就是我们的容器

至于仓储，就是放了一堆镜像的地方，我们可以把镜像发布到仓储中，需要的时候就从仓储中拉下来就可以了

## 2.Docker的安装

###### 2.1前置

###### 需要centos6.5以上版本

此处只是centos6.8版本的安装，如果是ubuntu，请百度

### 2.2安装

yum install -y epel-release

yum install -y docker -io

### 2.3安装后

配置文件位置： /etc/sysconfig/docker

启动docker后台服务： service docker start

docker version验证，用这个看一下你docker安装成功没

## 3.Docker的HelloWorld

### 3.1阿里云镜像加速

### 3.2helloworld

docker run hello-world



## 4.docker常用命令

### 4.1帮助命令

docker version

docker info

docker -help

### 4.1镜像命令

#### 4.1.1 docker images[options]

##### 介绍

列出本地主机上的镜像

##### OPTIONS说明

-a列出本地所有的镜像（含中间映像层）

-q值显示镜像ID

--digests：显示镜像的摘要信息

--no trunc：显示完整的镜像信息

##### 列出来的镜像的信息

repository：表示镜像的仓库源

TAG：镜像的标签

IMAGE ID：镜像ID

CREATED:镜像创建时间

SIZE：镜像大小

同一个仓库源可以有多个TAG，代表这个仓库源的不同个版本，我们使用REPOSITORY：TAG来定义不同的镜像

如果你不指定一个镜像的版本标签，例如你只使用ubuntu，docker将默认使用ubuntu：latest镜像

#### 4.1.2docker search 

##### 网站：

http://hub.docker.com

##### 命令

docker search [options] 镜像名字

##### options 说明

--no trunc：显示完整的镜像描述

-s：列出收藏数不小于指定值的镜像

--automated：只列出automated build类型的镜像

#### 

#### 4.1.3 docker pull 

从仓库中拉，，，镜像

例如docker pull tomcat等价于docker pull tomcat :latest

#### 4.1.4 docker rmi

移除某个镜像

docker rmi -f 镜像ID

docker rmi -f 镜像名1:TAG 镜像名2:TAG









# 还未学完



