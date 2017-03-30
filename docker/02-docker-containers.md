##### docker run

```
docker run -it ubuntu /bin/echo 'hello docker' 

1. 检查本地是否存在指定的 image，如果没有就从公有仓库 hub.docker.com 下载
2. 通过镜像创建容器并运行
3. 分配文件系统，在只读镜像层外面挂载一层可读写层
4. 从宿主主机配置的 docker0 网桥中桥接一个虚拟接口到容器中
5. 从地址池中配置一个 ip 给容器
6. 执行用户指定的应用程序
7. 执行完毕后容器被终止
```

##### 创建容器

```
docker create -it ubuntu
# 容器的几个状态
# created     <- docker create
# up          <- docker start/run/restart
# exit        <- docker stop/kill 或从 docker start run 结束
```

##### 启动容器

```
docker start containerID or containerName
# docker start -ai
# docker start -i，docker start -a 单独使用 会卡住
```

##### 运行容器

```
docker run -it ubuntu /bin/echo 'hello docker'     # 执行完 这条命令 容器就终止了
docker run -it ubuntu /bin/bash                    # 执行用户操作，最后 exit，ctrl+d 的时候容器就终止了
docker run -d ubuntu /bin/sh -c 'while true;do echo hello docker;sleep 1;done'     

# -d 以守护态运行，取决于命令，-d 不能和 --rm 同时使用
# docker run 等同于 docker create 然后 docker start
```

##### 查看日志

```
docker logs c93
```

##### 终止容器

```
docker stop c93
docker stop -t 10 c93
docker kill c93
```

##### 重启容器

```
docker restart c93
# docker stop .. docker start 
```

##### 列出容器

```
docker ps 
docker ps -l
docker ps -a
docker ps -aq

# -q 只列出 ID
```

##### 进入容器

```
docker attach   # 多个窗口attach时，同步显示，会相互阻塞
docker exec -it c93 /bin/bash

yum -y install util-linux
PID=$(docker inspect --format "{{ .State.Pid }} cb2)
PID=$(docker-pid c93)  # docker-pid 需要安装脚本
nsenter --target $PID --mount --uts --ipc --net --pid
# nsenter ns:namespace 的意思

```

##### 删除容器

```
docker rm [containerID or containerName]
docker ps -aq | xargs docker rm # 清空docker ps -a 列表
docker rm $(docker ps -aq)
docker rm -f # 强行终止并删除容器
docker rm -l # 删除容器的连接，保留容器
docker rm -v # 删除容器挂载的数据卷，好像不起作用
```

##### 导出容器

```
docker export -o ubuntu_export.tar ce5
docker export ce5 -o ubuntu_export.tar
docker export ce5 > ubuntu_export.tar
docker export > ubuntu_export.tar ce5
```

##### 导入容器

```
cat ubuntu_export.tar | docker import - u01/ubuntu:v1.1
docker import ubuntu_export.tar - u01/ubuntu:v1.1

# 导入之后变成镜像，docker image 查看
# 镜像的 存出 save   和 存入 load    保留完整记录，较大
# 容器的 导出 export 和 导入 import  丢弃历史和元数据，较小
```

##### 容器命名

```
docker run -it --name web ubuntu

# 命名的容器在操作时更加方便
```



