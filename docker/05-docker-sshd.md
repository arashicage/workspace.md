##### hub.docker.com

```
hub.docker.com search centos-sshd 或 `ubuntu-sshd`
such as https://hub.docker.com/r/komukomo/centos-sshd/~/dockerfile/
```

```
通常情况下
[server]
yum -y install openssh-server

[client]
ssh-keygen
ssh-copy-id user@node1  其实质是将 ~/.ssh/id_rsa.pub 内容追加到服务端 authorized_key
```


##### CentOS

```
====== 试验 1 ======
* 参考自 http://blog.csdn.net/cmzsteven/article/details/49096645
* 说实在的，这篇文章里 kengen 生成到 /etc/ssh/ssh_host_rsa_key 坑了我不少时间
* 基础镜像 centos:6.8
* ssh_host_rsa_key 生成到系统目录(本例为宿主机 /etc/ssh/)
* 从 ssh_host_rsa_key.pub 生成 authorized_keys 放到镜像中
```

```
#! /bin/bash

docker ps -aq --filter=ancestor=centos-sshd:latest | xargs docker rm -f
docker rmi centos-sshd:latest
rm -f /etc/ssh/ssh_host_rsa_key
rm -f /etc/ssh/ssh_host_rsa_key.pub
rm -f ssh_host_rsa_key
rm -f ssh_host_rsa_key.pub
rm -f id_rsa
rm -f id_rsa.pub
rm -rf ~/.ssh
rm -f authorized_keys

ssh-keygen -q -t rsa -b 2048 -f /etc/ssh/ssh_host_rsa_key -C '' -N ''
cat /etc/ssh/ssh_host_rsa_key.pub > authorized_keys
cp /etc/ssh/ssh_host_rsa_key .

cat <<EOF > Dockerfile
FROM centos:6.8
MAINTAINER by arashicage (arashicage@yeah.com)

# RUN yum -y update
RUN yum -y install openssh-server
RUN sed -i 's/UsePAM yes/UsePAM no/g' /etc/ssh/sshd_config

ADD ssh_host_rsa_key /etc/ssh/ssh_host_rsa_key
# ADD ssh_host_rsa_key.pub /etc/ssh/ssh_host_rsa_key.pub

RUN chmod 600 /etc/ssh/ssh_host_rsa_key
# RUN chmod 600 /etc/ssh/ssh_host_rsa_key.pub
RUN mkdir -p /root/.ssh
ADD authorized_keys /root/.ssh/authorized_keys

RUN echo "root:redhat"| chpasswd
# RUN echo "redhat" | passwd --stdin root

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]
EOF

docker build -t centos-sshd:latest .

docker run -d -p 10022:22 centos-sshd:latest

docker ps -a
```

```
# 下面两种方式，需要输入密码
ssh root@localhost -p 10022
ssh root@192.168.100.11 -p 10022

# 下面的方式，可以免输密码
ssh root@localhost -p 10022 -i ssh_host_rsa_key

# 虽然将 authorized_keys 放到镜像中，但是起不到免密的作用
# 但是既然已经ssh 连接了，可以通过下面几个步骤实现免密
# rm -rf ~/.ssh
# ssh-keygen -q -t rsa -b 2048 -f ~/.ssh/id_rsa -C '' -N ''
# ssh-copy-id root@localhost -p 10022

结论1: 也就是 add 到镜像中的 authorized_keys 必须和远程登录的主机 ~/.ssh/ 中公钥一致才能免密
```

-------

```
====== 试验 2 ======

* 基础镜像 centos:6.8
* ssh_host_rsa_key 生成到非系统目录(本例为宿主机 test-2 目录下，其实名称无所谓，这里换成了名字 id_rsa)
* 从 id_rsa.pub 生成 authorized_keys 放到镜像中
```

```
#! /bin/bash

docker ps -aq --filter=ancestor=centos-sshd:latest | xargs docker rm -f
docker rmi centos-sshd:latest
rm -f /etc/ssh/ssh_host_rsa_key
rm -f /etc/ssh/ssh_host_rsa_key.pub
rm -f ssh_host_rsa_key
rm -f ssh_host_rsa_key.pub
rm -f id_rsa
rm -f id_rsa.pub
rm -rf ~/.ssh
rm -f authorized_keys

ssh-keygen -q -t rsa -b 2048 -f id_rsa -C '' -N ''
cat id_rsa.pub > authorized_keys

cat <<EOF > Dockerfile
FROM centos:6.8
MAINTAINER by arashicage (arashicage@yeah.com)

# RUN yum -y update
RUN yum -y install openssh-server
RUN sed -i 's/UsePAM yes/UsePAM no/g' /etc/ssh/sshd_config

ADD id_rsa /etc/ssh/ssh_host_rsa_key
# ADD ssh_host_rsa_key.pub /etc/ssh/ssh_host_rsa_key.pub

RUN chmod 600 /etc/ssh/ssh_host_rsa_key
# RUN chmod 600 /etc/ssh/ssh_host_rsa_key.pub
RUN mkdir -p /root/.ssh
ADD authorized_keys /root/.ssh/authorized_keys

RUN echo "root:redhat"| chpasswd
# RUN echo "redhat" | passwd --stdin root

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]
EOF

docker build -t centos-sshd:latest .

docker run -d -p 10022:22 centos-sshd:latest

docker ps -a
```

```
ssh root@localhost -p 10022
ssh root@192.168.100.11 -p 10022

# 下面的方式，可以免输密码
ssh root@localhost -p 10022 -i ssh_host_rsa_key

# 虽然将 authorized_keys 放到镜像中，但是起不到免密的作用
# 但是既然已经ssh 连接了，可以通过下面几个步骤实现免密
# rm -rf ~/.ssh
# ssh-keygen -q -t rsa -b 2048 -f ~/.ssh/id_rsa -C '' -N ''
# ssh-copy-id root@localhost -p 10022

结论1: 也就是 add 到镜像中的 authorized_keys 必须和远程登录的主机 ~/.ssh/ 中公钥一致才能免密

结论2: 效果是和试验1 是一样的，所以 ssh-keygen 时 私钥公钥放哪无所谓，名字无所谓
```

-------


```
====== 试验 3 ======

* 基础镜像 centos:6.8
* ssh_host_rsa_key 生成到 ~/.ssh
* 从 id_rsa.pub 生成 authorized_keys 放到镜像中
```

```
#! /bin/bash

docker ps -aq --filter=ancestor=centos-sshd:latest | xargs docker rm -f
docker rmi centos-sshd:latest
rm -f /etc/ssh/ssh_host_rsa_key
rm -f /etc/ssh/ssh_host_rsa_key.pub
rm -f ssh_host_rsa_key
rm -f ssh_host_rsa_key.pub
rm -f id_rsa
rm -f id_rsa.pub
rm -rf ~/.ssh
rm -f authorized_keys

ssh-keygen -q -t rsa -b 2048 -f ~/.ssh/id_rsa -C '' -N ''
cat ~/.ssh/id_rsa.pub > authorized_keys
cp ~/.ssh/id_rsa .

cat <<EOF > Dockerfile
FROM centos:6.8
MAINTAINER by arashicage (arashicage@yeah.com)

# RUN yum -y update
RUN yum -y install openssh-server
RUN sed -i 's/UsePAM yes/UsePAM no/g' /etc/ssh/sshd_config

ADD id_rsa /etc/ssh/ssh_host_rsa_key
# ADD ssh_host_rsa_key.pub /etc/ssh/ssh_host_rsa_key.pub

RUN chmod 600 /etc/ssh/ssh_host_rsa_key
# RUN chmod 600 /etc/ssh/ssh_host_rsa_key.pub
RUN mkdir -p /root/.ssh
ADD authorized_keys /root/.ssh/authorized_keys

RUN echo "root:redhat"| chpasswd
# RUN echo "redhat" | passwd --stdin root

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]
EOF

docker build -t centos-sshd:latest .

docker run -d -p 10022:22 centos-sshd:latest

docker ps -a
```

```
# 因为authorized_key 和登录主机的 ~/.ssh/id_rsa.pub 内容一致，下面的方式免密
ssh root@localhost -p 10022

远程登录继续需要输入密码
ssh root@192.168.100.11 -p 10022 

# 下面的方式，可以免输密码
ssh root@localhost -p 10022 -i ssh_host_rsa_key

# 虽然将 authorized_keys 放到镜像中，但是起不到免密的作用
# 但是既然已经ssh 连接了，可以通过下面几个步骤实现免密
# rm -rf ~/.ssh
# ssh-keygen -q -t rsa -b 2048 -f ~/.ssh/id_rsa -C '' -N ''
# ssh-copy-id root@localhost -p 10022

结论1: 也就是 add 到镜像中的 authorized_keys 必须和远程登录的主机 ~/.ssh/ 中公钥一致才能免密

结论2: 效果是和试验1 是一样的，所以 ssh-keygen 时 私钥公钥放哪无所谓，名字无所谓

结论3: 同结论1，保持一致时就免密了
```

-------

```
====== 试验 4 ======

* 基础镜像 centos:6.8
* ssh_host_rsa_key 生成到 ~/.ssh
* 从 id_rsa.pub 生成 authorized_keys 放到镜像中
* 试验3的技术上，去掉 ADD id_rsa /etc/ssh/ssh_host_rsa_key 行不行
```

```
#! /bin/bash

docker ps -aq --filter=ancestor=centos-sshd:latest | xargs docker rm -f
docker rmi centos-sshd:latest
rm -f /etc/ssh/ssh_host_rsa_key
rm -f /etc/ssh/ssh_host_rsa_key.pub
rm -f ssh_host_rsa_key
rm -f ssh_host_rsa_key.pub
rm -f id_rsa
rm -f id_rsa.pub
rm -rf ~/.ssh
rm -f authorized_keys

ssh-keygen -q -t rsa -b 2048 -f ~/.ssh/id_rsa -C '' -N ''
cat ~/.ssh/id_rsa.pub > authorized_keys
cp ~/.ssh/id_rsa .

cat <<EOF > Dockerfile
FROM centos:6.8
MAINTAINER by arashicage (arashicage@yeah.com)

# RUN yum -y update
RUN yum -y install openssh-server
RUN sed -i 's/UsePAM yes/UsePAM no/g' /etc/ssh/sshd_config

# ADD id_rsa /etc/ssh/ssh_host_rsa_key
# ADD ssh_host_rsa_key.pub /etc/ssh/ssh_host_rsa_key.pub

# RUN chmod 600 /etc/ssh/ssh_host_rsa_key
# RUN chmod 600 /etc/ssh/ssh_host_rsa_key.pub
RUN mkdir -p /root/.ssh
ADD authorized_keys /root/.ssh/authorized_keys

RUN echo "root:redhat"| chpasswd
# RUN echo "redhat" | passwd --stdin root

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]
EOF

docker build -t centos-sshd:latest .

docker run -d -p 10022:22 centos-sshd:latest

docker ps -a
```

```
# 因为authorized_key 和登录主机的 ~/.ssh/id_rsa.pub 内容一致，下面的方式免密
ssh root@localhost -p 10022

远程登录继续需要输入密码
ssh root@192.168.100.11 -p 10022 

# 下面的方式，可以免输密码
ssh root@localhost -p 10022 -i ssh_host_rsa_key

# 虽然将 authorized_keys 放到镜像中，但是起不到免密的作用
# 但是既然已经ssh 连接了，可以通过下面几个步骤实现免密
# rm -rf ~/.ssh
# ssh-keygen -q -t rsa -b 2048 -f ~/.ssh/id_rsa -C '' -N ''
# ssh-copy-id root@localhost -p 10022

结论1: 也就是 add 到镜像中的 authorized_keys 必须和远程登录的主机 ~/.ssh/ 中公钥一致才能免密

结论2: 效果是和试验1 是一样的，所以 ssh-keygen 时 私钥公钥放哪无所谓，名字无所谓

结论3: 同结论1，保持一致时就免密了

ssh root@localhost -p 10022
[root@docker-node1 test-4]# ssh root@localhost -p 10022
Connection closed by 127.0.0.1
结论4: 去掉 ADD id_rsa /etc/ssh/ssh_host_rsa_key，即镜像中没有 /etc/ssh/ssh_host_rsa_key 是不行的（影响ssh 提供服务），添加方式有很多种
1. ADD id_rsa /etc/ssh/ssh_host_rsa_key 从宿主机添加
2. build 的时候，service ssh start && service ssh stop 生成
3. build 的时候，RUN ssh-keygen -q -t rsa -f /etc/ssh/ssh_host_rsa_key -C '' -N ''
4. 从宿主机添加的和build 的时候命令生成都可以，表示内容无所谓，文件必须有

后面一个就是docker file 就是在build 的时候 service ssh start && service ssh stop 生成的
```

-------

##### Ubuntu

```
mkdir -p /root/ubuntu-sshd && cd /root/ubuntu-sshd

ssh-keygen -q -t rsa -f ~/.ssh/id_rsa -C '' -N ''

cat ~/.ssh/id_rsa.pub > authorized_keys

cat << EOF > run.sh
#!/bin/bash
/usr/sbin/sshd -D
EOF

FROM ubuntu
MAINTAINER arashicage arashicage@yeah.net

RUN apt-get update \
    && apt-get -y install openssh-server \
    && mkdir -p /var/run/sshd \
    && mkdir -p /root/.ssh \
    && sed -ri 's/session required pam_loginuid.so/#session required pam_loginuid.so/g' /etc/pam.d/sshd

ADD authorized_keys /root/.ssh/authorized_keys
ADD run.sh /run.sh
RUN chmod 755 /run.sh

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]

docker build -t ubuntu-sshd:latest .

docker run -d -p 10022:22 ubuntu-sshd
ssh root@localhost -p 10022

这种方式使用免密登录，不需要输入密码，如果重新生成 id_rsa，免密将无法登录，转为密码登录，但是整个过程都没有设置密码。这个时候只能通过
docker exec -it AA /bin/bash 或
nsenter 的方式进入了

id_rsa 重新生成后，能不能登录
如果要对多个容器配置ssh ，那么id_rsa 不能重复生成，意义何在

ssh-keygen -t rsa -P "" -f ~/.ssh/id_rsa 

ssh -o stricthostkeychecking=no node1
```


