##### 注册服务器 Registry

```
hub.docker.com          # 
gcr.io                  # kubernetes 相关的 images 都在这
hub.c.163.com           # 163.com
daocloud.io             # daocloud.io, 也提供加速器
index.tenxcloud.com     # 时速云

网易蜂巢-hub.docker.com
docker pull hub.c.163.com/library/nginx:latest

daocloud-hub.docker.com
docker pull daocloud.io/library/nginx:1.7.1

# 时速云-hub.docker.com
docker pull index.tenxcloud.com/docker_library/ubuntu
# 时速云-gcr.io/google_containers
docker pull index.tenxcloud.com/google_containers/kube-controller-manager-amd64
``` 
    
##### 仓库 Repository

```

```
    
##### 镜像 Image

```
基础镜像/根镜像，由官方创建，验证，支持，提供。单词表示，如:nginx
用户镜像，非官方提供，以用户或组织名开头，scott/nginx
```

##### 容器 Container

```

```


##### 自动创建
利用hub.docker.com 来中转 gcr.io 



##### 私有仓库搭建

```
docker run -d -p 5000:5000 -v /opt/data/registry:/tmp/registry registry
```


