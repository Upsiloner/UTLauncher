# Docker-Compose 部署前后端

## 项目目录结构

分别将前后端项目克隆到对应的文件夹中

```
├── docker-compose.yml
├── frontend
│ └── dockerfile
├── backend
│ └── dockerfile
```

## 命令

### docker 镜像加速

linux 下修改`/etc/docker/daemon.json`，添加下列配置

```
{
  "registry-mirrors": [
    "https://mirror.ccs.tencentyun.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://registry.docker-cn.com"
  ]
}
```

添加后运行以下命令重启

```
systemctl daemon-reload
systemctl restart docker
```

启动 docker-compose 的命令

```
docker-compose up
```

### 代码更新后重新构建镜像

由于 Golang 需要编译，前端需要重新打包，所以修改代码后需要重新构建镜像

#### 全部重新构建

```
docker-compose down
docker-compose up --build
```

全部重新构建可能速度较慢，也可以单独构建某个服务镜像，有两种方式，以前后端分开举例

#### GinBackend

使用了 Air，可以自动监看代码更改实现热重载，在 linux 下正常，windows 下有问题，需要手动重新构建

```
# 删除当前容器，当前版本compose支持down某一个service，但是down会注销网络，使用up --build会出现找不到网络的问题，所以先使用build命令再up
docker-compose down backend

# 重建 Docker 镜像(配合 --build 选项来强制重新构建镜像)
docker-compose build backend
# 重新compose up
docker-compose up
```

#### Umi Front

前端是打包部署在 nginx 容器中的，所以每次代码更改都需要重新 build

```
# 先使用stop命令停掉frontend容器
docker-compose stop frontend
# 再使用rm移除frontend容器
docker-compose rm -f frontend

# 重建 Docker 镜像(配合 --build 选项来强制重新构建镜像)
docker-compose up --build frontend
# 如果遇到缓存导致构建未体现更新，可以使用以下命令
docker-compose build --no-cache frontend
docker-compose up -d
```

### 常用 docker 命令

```
# 删除所有未被标记且未被任何容器引用的镜像
docker image prune
```
