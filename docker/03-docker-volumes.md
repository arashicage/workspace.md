##### 数据卷

```
数据卷的作用
1. 容器间共享

docker run -it --name web -v /webapp utuntu
docker 

docker run -it \
-v /Users/arashicage/workspace/workspace.go:/workspace.go \
-v /Users/arashicage/workspace/workspace.misc:/workspace.misc:ro \
-v /Users/arashicage/workspace/.bash_history:/.bash_history \
ubuntu \
/bin/bash -c 'ls -ld /workspace*;df -h'

# 可以挂载目录，也可以挂载文件，上面的第三条，将容器的命令历史记录到了宿主机下
# 挂载的数据卷，默认权限是 rw

drwxr-xr-x 5 root root 170 Feb 14 02:46 /workspace.go
drwxr-xr-x 6 root root 204 Sep  5  2016 /workspace.misc
Filesystem      Size  Used Avail Use% Mounted on
none             60G  1.5G   55G   3% /
tmpfs          1000M     0 1000M   0% /dev
tmpfs          1000M     0 1000M   0% /sys/fs/cgroup
osxfs           465G  196G  269G  43% /workspace.go    # df 命令只能看到第一个，这里的大小和为挂载目录所在磁盘的大小和使用率
/dev/vda2        60G  1.5G   55G   3% /etc/hosts
shm              64M     0   64M   0% /dev/shm
tmpfs          1000M     0 1000M   0% /sys/firmware

[挂载多个数据卷，用 df 查看只会看到一个，会让人迷惑]
```

##### 数据卷容器

```
➜  ~ docker run -it -v /dbdata ubuntu
root@ab02f626baa0:/# df -h
Filesystem      Size  Used Avail Use% Mounted on
none             60G  1.6G   55G   3% /
tmpfs          1000M     0 1000M   0% /dev
tmpfs          1000M     0 1000M   0% /sys/fs/cgroup
/dev/vda2        60G  1.6G   55G   3% /dbdata   # 60G 大小
shm              64M     0   64M   0% /dev/shm
tmpfs          1000M     0 1000M   0% /sys/firmware

docker run -it -v /dbdata --name dbdata1 ubuntu
docker run -it -v /dbdata --name dbdata2 ubuntu

docker run -it --volumes-from dbdata1 --name db1 postgres
docker run -it --volumes-from dbdata1 --name db2 postgres
docker run -it --volumes-from db1 --name db3 postgres      
docker run -it --volumes-from dbdata1 --name --volumes-from dbdata2 db4 postgres

# 数据卷容器
# 多对多，传递性
# 数据卷容器的数据卷可以被多个其他容器通过 --volumes-from 进行挂载
# 同一个容器可以多次使用 --volumes-from 来挂载多个容器卷里的数据卷
# 传递性， --volumes-from 参数后面可以是挂载了其他数据卷的某个容器
# --volumes-from 后面的数据卷容器不需要保持运行状态
# 删除挂载的数据卷容器，容器中的数据卷不会自动删除
```



