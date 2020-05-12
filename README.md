# docker-deploy

使用docker部署服务

收集日常用到的一些服务，方便部署

## 设置代理

macOS下设置，在 `.docker/daemon.json` 添加

```bash
{
  "experimental" : false,
  "registry-mirrors" : [
    "https://xxx.mirror.aliyuncs.com"
  ],
  "debug" : true
}
```

注意：镜像地址需要设置为记得地址，可以在阿里云控制台里找到

## docker 基本命令操作

## 镜像

```bash
# 删除容器
docker rmi <image_name>
```

### 容器操作

```bash
# 启动容器，下面的命令输出一个 “Hello World”，之后终止容器
docker run ubuntu:18.04 /bin/echo 'Hello world'

# 启动一个 bash 终端，允许用户进行交互
docker run -t -i ubuntu:18.04 /bin/bash

# 守护态运行
docker run -d ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"

# 查看容器列表
docker container ls # 或 docker ps

# 查看log
docker container logs [container ID or NAMES]

# 启动容器
docker container start

# 终止容器
docker container stop

# 重启容器
docker container restart

# 查看终止状态的容器
docker container ls -a

# 进入容器
docker exec -it <container_id> bash

# 删除容器
docker container rm  <container_id>/<image_name>
docker rm 

# 清理所有处于终止状态的容器
docker container prune

# 删除所有容器
docker stop `docker ps -q -a` | xargs docker rm

# 删除所有标签为none的镜像
docker images|grep \<none\>|awk '{print $3}'|xargs docker rmi

# 查找容器IP地址
docker inspect 容器名或ID | grep "IPAddress"

# 创建网段, 名称: mynet, 分配两个容器在同一网段中 (这样子才可以互相通信)
docker network create mynet
docker run -d --net mynet --name container1 my_image
docker run -it --net mynet --name container1 another_image
```

