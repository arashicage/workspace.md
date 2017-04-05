##### docker 的启动配置文件

```
# 在使用 upstart 管理启动服务的系统中，比如：debian/ubuntu
/etc/default/docker

# 在使用 systemd 管理启动服务的系统中，比如 rhel-7，centos-7 等
/etc/systemd/system/docker.service.d/docker.conf
```

##### docker daemon/client

```
docker daemon 
    (default)unix://var/run/docker.sock
    docker daemon -H 0.0.0.0:1234   # 监听本地 TCP/1234 端口
docker client
    (default)unix://var/run/docker.sock
    docker -H tcp://0.0.0.0:1234    # 连接本地 TCP/1234 端口
```

#### docker 核心技术
* 命名空间
* 控制组
* 联合文件系统
* 网络虚拟化

```
LXC Linux Container
libcontainer 0.9 版本以后
```

内核，文件系统，网络，PID，UID，IPC，
内存，磁盘IO，CPU，网络IO

##### 命名空间 Namespace
[命名空间讲解](http://www.baidu.com)
利用命名空间的特性，每个容器都可以拥有自己单独的命名空间。运行在容器中的应用就像是运行在独立的操作系统环境中一样。不同容器中的进程彼此不可见，都认为自己是独占系统的。
###### 进程命名空间
同一进程在不同命名空间其进程号是不一样的。
子空间中的进程对父空间是可见的。
同一进程在父空间和子空间分别有一个进程号来对应。
###### 网络命名空间
网络命名空间，使得每个容器的网络能隔离开来。docker 使用虚拟网络设备，不同命名空间的网络设备veth 连接到主机的 docker0 网桥上。
###### IPC命名空间
interprocess communication，通常配合进程命名空间一起使用。
###### 挂载命名空间
类似chroot，挂载命名空间允许不同命名空间的进程看到的文件结构不同。
###### UTS命名空间
每个容器拥有独立的主机名或域名。
###### 用户命名空间
每个容器都可以有root 用户，在容器内，可以使用特定的内部用户执行程序，而非本地系统上的用户。

##### 控制组 ControlGroup
对共享资源进行隔离，限制，审计，优先级控制，控制
##### 联合文件系统 UNION FS，AUFS
高性能分层文件系统，对文件系统的修改可以视为一次提交，层层提交。不同目录可以挂载到同一个虚拟文件系统，并设置不同的读写权限。类似git

AUFS，OverlayFS，devicemapper，btrfs，vfs，zfs

##### 网络虚拟化 

```
网络接口: 与外界相通，收发数据
路由机制: 不同子网之间进行通信

docker0 在内核层连通其他物理或虚拟网卡，将所有容器和本地主机放到同一个物理网络。

(1) 创建一对虚拟接口，分别放到本地主机和新容器中
(2) 本地主机一端桥接到默认的 docker0 或指定网桥上，并具有一个唯一的名字，如 veth65f9
(3) 容器一端放到新容器中，并修改名字作为 eth0，这个接口只在容器的命名空间可见
(4) 从网桥可用地址段中获取一个空闲地址分配给容器的 eth0，并配置默认路由到桥接网卡 veth65f9。

$ sudo docker run -i -t --rm --net=none base /bin/bash
root@63f36fc01b5f:/#

$ sudo docker inspect -f '{{.State.Pid}}' 63f36fc01b5f
2778
$ pid=2778
$ sudo mkdir -p /var/run/netns
$ sudo ln -s /proc/$pid/ns/net /var/run/netns/$pid

$ ip addr show docker0
21: docker0: ...
inet 172.17.42.1/16 scope global docker0

$ sudo ip link add A type veth peer name B
$ sudo brctl addif docker0 A
$ sudo ip link set A up

$ sudo ip link set B netns $pid
$ sudo ip netns exec $pid ip link set dev B name eth0
$ sudo ip netns exec $pid ip link set eth0 up
$ sudo ip netns exec $pid ip addr add 172.17.42.99/16 dev eth0
$ sudo ip netns exec $pid ip route add default via 172.17.42.1

```

```
/etc/resolv.conf  默认和宿主机一致
/etc/hostname     主机名 
/etc/hosts        容器自身的条目
# docker 1.2.0 可以在容器中编辑，运行过程中保留，终止重启后无效，docker commit 后也不会保留下来

```


##### 容器访问外部网络

```
# 查看
sysctl net.ipv4.ip_forward
# 设置
sysctl -w net.ipv4.ip_forward = 1
sysctl -p

上面的转发是必须的，另外 docker 还干了啥
iptables -t nat -nvL POSTROUOTING
容器所有到外部网络的连接，源地址都会被NAT成本地系统的IP地址。这是使用 iptables 的源地址伪装 SNAT 操作实现的

iptables -t nat -nvL POSTROUTING
Chain POSTROUTING (policy ACCEPT 102 packets, 7728 bytes)
 pkts bytes target     prot opt in     out     source               destination
  427 27435 MASQUERADE  all  --  *      !docker0  172.17.0.0/16        0.0.0.0/0
# 容器中出来的流量伪装成系统网卡发出
```

##### 容器之间互访

```
1. 网络拓扑连通
    默认情况下，容器的网络接口都连接到 docker0 上，网络拓扑是连通的
2. 防火墙策略 --icc=true
    docker 服务器启动时，默认会添加 允许 转发策略到本地 iptables FORWARD 链。在docker 配置文件中设置 DOCKER_OPTS=--icc=false 将禁用容器互联。docker 启动时设置 --iptables=false 将禁止上述行为。
    
    即使 --icc=false，通过--link 可以访问指定容器开放的端口，如果 --iptables 也设置为 false 了，那就没戏了
    
   --icc=false，--iptables=true ，使用 --link 至少保证 --iptables=true
   --link 时，会为互联的两个容器分别添加一条 accept 规则，允许互相访问开放的端口 
```

##### 外部访问容器

```
-p 映射指定端口
-P 映射全部端口

本质是在宿主机的 iptables 中添加规则，通过 DNAT，目标地址改为 容器地址

```

OpenvSwitch 网桥

容器直连
两个命名空间，一对pair，各放一个








