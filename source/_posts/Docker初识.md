---
title: Docker初识
date: 2020-09-03 23:38:16
categories:
- Docker
tags:
- Docker
---

# 简介

Docker是一个开源的应用容器引擎

Docker支持将软件编译成一个镜像，然后在镜像中各种软件做好配置，将镜像发布出去，其他使用者可以直接使用这个镜像

运行中的这个镜像称为容器，容器启动是非常快速的

![Docker](https://gitee.com/tongying003/MapDapot/raw/master/img/20200903234429.svg)

<!-- more -->

# 核心概念

**Docker主机（Host）**：安装了Docker程序的主机（Docker直接安装在操作系统之上）

**Docker客户端（Client**）：连接Docker主机进行操作

**Docker仓库（Registry）**：用来保存各种打包好的软件镜像

**Docker镜像（Images）**：软件打包好的镜像，放在Docker仓库中

**Docker容器（Container）**：镜像启动后的实例称为一个容器，容器是独立运行的一个或一组应用

![Docker2](https://gitee.com/tongying003/MapDapot/raw/master/img/20200903234429.svg)

**使用Docker的步骤**：

1）安装Docker

2）去Docker仓库找到这个软件对应的镜像

3）使用Docker运行这个镜像，这个镜像就会生成一个Docker容器

4）对容器的启动停止就是对软件的启动停止



# 安装Docker

[Docker官网安装教程](https://docs.docker.com/engine/install/centos/)

启动Docker

```shell
systemctl start docker
```

停止Docker

```shell
systemctl stop docker
```

更换阿里镜像源

- 进入https://cr.console.aliyun.com/
- 点击左侧`镜像加速器`

- 执行命令`vim /etc/docker/daemon.json`，加入如下内容

```json
{
  "registry-mirrors": ["https://kh4uoxfp.mirror.aliyuncs.com"]
}
```

- 重启docker

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

# 常用操作

## 镜像操作

| 操作 | 命令                     | 说明                                                    |
| ---- | ------------------------ | ------------------------------------------------------- |
| 检索 | `docker search 关键字`   | 在docker hub上检索镜像详细信息，如镜像的TAG             |
| 拉取 | `docker pull 镜像名:tag` | :tag是可选的，tag表示标签，多为软件的版本，默认是latest |
| 列表 | `docker images`          | 查看所有本地镜像                                        |
| 删除 | `docker rmi image-id`    | 删除指定的本地镜像                                      |

## 容器操作

| 操作     | 命令                                                         | 说明                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 运行     | `docker run --name container-name -d image-name`<br>eg: `docker run --name myredis -d redis` | `--name`:自定义容器名<br>`-d`：后台运行<br>image-name: 指定镜像模版 |
| 列表     | `docker ps`（查看运行中的容器）                              | 加上`-a`可以查看所有容器                                     |
| 停止     | `docker stop container-name/container-id`                    | 停止当前运行的容器                                           |
| 启动     | `docker start container-name/container-id`                   | 启动容器                                                     |
| 删除     | `docker rm container-id`                                     | 删除指定容器                                                 |
| 端口映射 | -p 6379:6379<br>eg:`docker run -d -p 6379:6379 --name myredis docker.io/redis` | -p:主机端口（映射到）容器内部的端口                          |
| 容器日志 | `docker logs container-name/container-id`                    |                                                              |
| 更多命令 | https://docs.docker.com/reference/                           |                                                              |
|          |                                                              |                                                              |

软件镜像—>运行镜像—>产生一个容器（正在运行的软件）

**示例**

```shell
1. 搜索镜像
[root@Mangoo ~]# docker search tomcat
2. 拉去镜像
[root@Mangoo ~]# docker pull tomcat
3. 根据镜像启动容器
[root@Mangoo ~]# docker run --name mytomcat -d tomcat:latest
4. 查看运行中的容器
[root@Mangoo ~]# docker ps
5. 停止运行中的容器
[root@Mangoo ~]# docker stop a3ba0917b5dd
6. 查看所有的容器
[root@Mangoo ~]# docker ps -a
7. 启动容器
[root@Mangoo ~]# docker start a3ba0917b5dd
8. 删除容器
[root@Mangoo ~]# docker rm a3ba0917b5dd
9. 端口映射
[root@Mangoo ~]# docker run -p -d 8888:8080 tomcat
10. 访问tomcat首页
1) 在阿里云安全组配置中打开8888端口
2）若是启用了防火墙，配置防火墙并打开8888端口
3）在阿里源拉取的镜像webapps目录下为空，需将webapps.dist改名为webapps
4) 访问阿里云主机外网ip:8888

11. 查看容器的日志
[root@Mangoo ~]# docker logs 8428ed7be348
12.更多命令可以参考每个镜像的文档
```

 