##### 获取帮助 !!!

```
docker --help
docker-subcommand --help
[docs on line](https://docs.docker.com/)
dash offline
```

##### 搜索镜像

```
docker search ubuntu
docker search --limit=10 --filter=stars=3 ubuntu

# --flag=xx 和 --falg xx 是等效的
```

##### 拉取镜像

```
docker pull ubuntu[:tag default:latest]
docker pull ubuntu:12.04
docker pull registry.hub.com/ubuntu:latest

# registry.hub.com 是默认的仓库注册服务器
```

##### 查看镜像

```
docker images
docker images --format "{{ .Repository }}\t{{ .Tag }}\t{{ .ID }}"

# --format 选项在写 shell 脚本的时候真的很有用
# image 的 ID 是唯一的，其 tag 可以是多个。
```

##### 检视镜像

```
docker inspect ubuntu
docker inspect --format "{{ .ID }}" ubuntu
```

##### 删除镜像

```
docker rmi images [..images] 
# image 可以是 name:tag 或 id
docker rmi busybox
docker rmi busybox ubuntu
docker rmi 00f017a8c2a6 cf725f136fd2

# 当一个image 存在多个 tag 时，如果按 tag 删除镜像，只会删除指定的 tag，不会删除 image 本身
# 当还有基于该 image 的容器正在运行时，该镜像无法删除，docker rmi -f 
```

##### TAG

```
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
# tag 用途
#    标注版本信息 version
#    标注应用场景 dev prod beta alpha
#    比如从 docker 中转了 gcr.io 的镜像，pull 下来后，需要重新 tag 为 gcr.io/images
#    push 前需要 tag 为自己的空间下 docker tag ubuntu username/ubuntu:latest
```

##### 创建镜像

```
1. 从容器
2. 从模板
3. 从 dockerfile 构建

docker run -it ubuntu /bin/bash
... do some modification ...
docker commit -m "nginx installed" -a "arashicage" containerID u01:tag
docker images 
```

##### 导出镜像

```
docker save -o ubuntu_14.04.tar ubuntu:14.04
```

##### 导入镜像

```
docker load --input ubuntu_14.04.tar
docker load < ubuntu_14.04.tar
```

##### 上传镜像

```
docker tag ubuntu:latest username/ubuntu:latest
docker push username/ubuntu:latest

```

