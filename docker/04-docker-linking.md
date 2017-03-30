##### 容器互联

```
1. 端口映射
2. Linking
```

##### 端口映射

```
IP:HostPort:ContainerPort      # 指定接口和指定端口
IP::ContainerPort              # 指定接口和随机端口
HostPort:ContainerPort         # 所有接口和指定端口

IP                             # 不写表示 全部接口
HostPort                       # 不写表示 随机端口
ContainerPort                  # 必须有

docker run -d -P --name web ubuntu python app.py
# 下面两条命令都能查到 端口映射情况
docker ps -l
docker inspect web

-P 随机 49000~49900，映射所有容器开放的端口
-p 
```

##### 查看端口映射

```
docker ps -a
docker ps -l

docker port web 5000
127.0.0.1:49155

docker inspect containerId

docker inspect "{{.Name}}" aeadfa123
```
##### Linking

```
--link name:alias 
name     # 容器名
alias    # 容器内使用的别名

docker run -d --name db redis
docker run -d -P --name web --link db:db nginx

➜  ~ docker run --rm --name web --link db:db nginx env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=79471eb94de3
DB_PORT=tcp://172.17.0.2:6379
DB_PORT_6379_TCP=tcp://172.17.0.2:6379
DB_PORT_6379_TCP_ADDR=172.17.0.2
DB_PORT_6379_TCP_PORT=6379
DB_PORT_6379_TCP_PROTO=tcp
DB_NAME=/web2/db
DB_ENV_GOSU_VERSION=1.7
DB_ENV_REDIS_VERSION=3.2.5
DB_ENV_REDIS_DOWNLOAD_URL=http://download.redis.io/releases/redis-3.2.5.tar.gz
DB_ENV_REDIS_DOWNLOAD_SHA1=6f6333db6111badaa74519d743589ac4635eba7a
NGINX_VERSION=1.11.10-1~jessie
HOME=/root
➜  ~ docker run --rm --name web --link db:db nginx cat /etc/hosts
127.0.0.1      localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2      db a3239ea25d2d
172.17.0.3      acd891e2fbd8
```

