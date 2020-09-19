# Docker命令

## 帮助命令

```bash
docker version #显示docker的版本信息
docker info    #显示docker的系统信息，包括镜像和容器的数量
docker [命令] --help #帮助命令 比如 docker --help显示所有docker命令，又比如 docker images --help显示images命令的帮助信息
```

命令行文档地址：[命令行](https://docs.docker.com/engine/reference/commandline/docker/)。

## 镜像命令

### docker images 显示本地所有镜像

```bash
[root@init_center ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              bf756fb1ae65        8 months ago        13.3kB

# 常用参数
-a 显示所有镜像
-q 只显示id
```

### docker search 从docker hub搜索镜像

```bash
[root@init_center ~]# docker search golang
NAME                             DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
golang                           Go (golang) is a general purpose, higher-lev…   3347                [OK]
ilanyu/golang-reverseproxy       golang-ReverseProxy                             88                                      [OK]
circleci/golang                  CircleCI images for Go                          15                                      [OK]                                                                
```

### docker pull 从docker hub下载镜像

```bash
# 下载镜像 docker pull 镜像名[:tag]
# 如果不写tag，默认tag是latest，即最后一个版本
[root@init_center ~]# docker pull mysql
Using default tag: latest # mysql通过tag指定版本
latest: Pulling from library/mysql
d121f8d1c412: Pull complete		#分层下载，docker 镜像是分层的
f3cebc0b4691: Pull complete
1862755a0b37: Pull complete
489b44f3dbb4: Pull complete
690874f836db: Pull complete
baa8be383ffb: Pull complete
55356608b4ac: Pull complete
dd35ceccb6eb: Pull complete
429b35712b19: Pull complete
162d8291095c: Pull complete
5e500ef7181b: Pull complete
af7528e958b6: Pull complete
Digest: sha256:e1bfe11693ed2052cb3b4e5fa356c65381129e87e38551c6cd6ec532ebe0e808
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest #真实地址

```

### docker rmi 从本地删除镜像

```bash
# 可以通过id、标签或者摘要来删除
# 可以指定删除多个镜像
# 可以增加-f参数强制删除
# 可以使用 docker rmi -f $(docker images -aq) 删除所有镜像

[root@init_center ~]# docker rmi mysql
Untagged: mysql:latest
Untagged: mysql@sha256:e1bfe11693ed2052cb3b4e5fa356c65381129e87e38551c6cd6ec532e                                                                                                                              be0e808
Deleted: sha256:e1d7dc9731daa2c79858307d65ef35f543daf97457b9c11b681f78bb86f7a158
Deleted: sha256:303dd82484b7080d07af8ab7f383755d8b4d723a2ade8c2e3a516ae59fbc1d63
```

### docker commit 提交容器副本生成新镜像

```bash
# docker commit 可以根据容器在我们进行自定义后生成一个新的镜像
# 新生成的镜像包含我们对这个容器所进行的更改
# 以后我们就可以根据这个新的镜像生成基于我们需求的容器
# 命令如下
docker commit -m="提交的描述信息" -a="作者" 容器 要创建的镜像名:[标签名]
```



## 容器命令

### docker run 创建容器

```bash
# 要先有镜像才可以创建容器，所以我们先pull一个容器
docker pull centos
```

```bash
# 创建容器
docker run [可选参数] image

# 参数设置
--name="name" # 给容器设置名称，用于区分容器
-d			  # 后台方式运行
-it           # 使用交互方式运行，进入容器查看内容
-p            # 指定容器端口 
	-p ip:主机端口:容器端口
	-p 主机端口: 容器端口 （常用） # 映射，访问主机的端口可以访问到容器内的端口
	-p 容器端口
-P # 大P是随机分配主机端口，但是容器内部启用的端口是应用决定的
-e # 注入环境变量， 比如mysql需要密码，在启动时可以通过环境变量注入
	# -e MYSQL_ROOT_PASSWORD=123456
-v # 绑定数据卷
```

```bash
# -it 测试

# 运行命令后可以看到主机名称的改变，此时已经进入容器
# 使用exit让容器停止并退出
# 使用 ctrl + p + q 让容器不停止退出

[root@init_center ~]# docker run -it centos /bin/bash
[root@4a867b283441 /]# exit
exit
```

### docker ps 查看运行的容器

```bash
[root@init_center ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES


# -a 参数可以查看全部运行的容器，包括历史运行过的
[root@init_center ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
4a867b283441        centos              "/bin/bash"         6 minutes ago       Exited (0) 4 minutes ago                       musing_pascal
a6b664c14cb8        hello-world         "/hello"            6 hours ago         Exited (0) 6 hours ago                         inspiring_buck

# -n=? 可以查看最近创建的容器, ?表示最近的几个，比如-n=1表示最近的一个
[root@init_center ~]# docker ps -n=1
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
4a867b283441        centos              "/bin/bash"         7 minutes ago       Exited (0) 5 minutes ago                       musing_pascal

# -q 只显示容器的编号
[root@init_center ~]# docker ps -aq
4a867b283441
a6b664c14cb8
```

### Ctrl+P+Q 不停止容器退出

```bash
[root@init_center ~]# docker run -it centos
[root@c770fdd28327 /]# [root@init_center ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
c770fdd28327        centos              "/bin/bash"         2 minutes ago       Up 2 minutes                            upbeat_cray

```

### docker rm 删除容器

```bash
docker rm 容器		#删除指定容器，加-f参数才能删除运行中的容器

docker rm -f $(docker ps -aq)	#删除所有容器
```

### 启动和停止容器

```bash
docker start 容器	#启动容器
docker restart 容器	#重启容器
docker stop 容器	#停止当前运行的容器
docker kill 容器	#强制停止当前容器
```

### 后台启动容器的问题

```bash
# 比如当我们使用 docker run -d centos 想要后台启动一个centos容器时
# 使用docker ps 查看运行的容器， 会发现centos这个容器不在列表中，停止了
# 这是因为容器后台运行必须要有一个前台进程，docker发现容器中没有进程，自己没有提供服务，就会立即停止。
```

### docker logs查看日志

```bash
docker logs [可选参数] 容器

# -t 参数 显示时间戳
# -f 跟踪实时日志
# --details        显示更多的信息
# --since string   显示自某个timestamp之后的日志，或相对时间，如42m（即42分钟）
# --tail string    从日志末尾显示多少行日志， 默认是all
# --until string   显示自某个timestamp之前的日志，或相对时间，如42m（即42分钟）
```

### docker top 查看容器内部的进程信息

```bash
docker top 容器
```

### docker exec 进入容器

```bash
docker exec -it 容器 /bin/bash
```

### docker inspect 查看镜像元数据

```bash
docker inspect 容器
```

### docker cp 从容器内拷贝文件到主机

```bash
docker cp 容器:容器内路径 目的主机路径
```

# 容器数据卷

## 什么是容器数据卷

当我们在使用`docker`容器的时候，会产生一系列的数据文件，这些数据文件在我们删除`docker`容器时是会消失的，但是其中产生的部分内容我们是希望能够把它给保存起来另作用途的，`Docker`将应用与运行环境打包成容器发布，我们希望在运行过程中产生的部分数据是可以持久化的的，而且容器之间我们希望能够实现数据共享。

通俗地来说，`docker`容器数据卷可以看成使我们生活中常用的`u盘`，它存在于一个或多个的容器中，由`docker`挂载到容器，但不属于联合文件系统，`Docker`不会在容器删除时删除其挂载的数据卷。

## 挂载数据卷

> 使用命令挂载

`docker run`的`-v`参数可以在`docker`创建时挂载数据卷：

```bash
docker run -it -v 主机目录:容器内的目录
```

可以使用多个`-v`映射多个目录。

> 通过Dockerfile挂载

```dockerfile
# 创建dockerfile文件时写入挂载数据卷
# 这时通过dockerfile创建的镜像在生成容器时就会挂载数据卷
FROM centos

# 挂载数据卷（匿名挂载）
# 这里写的是容器内的目录
VOLUME ["/volume01", "/volume02"]

CMD echo "---end---"

CMD /bin/bash

```



### 匿名挂载和具名挂载

我们在挂载数据卷时可以不指定主机目录，而是指定一个名称或者不指定名称。指定名称就是具名挂载，不指定名称就是匿名挂载。

匿名挂载：

```bash
# 不指定主机目录，也不指定挂载名称
docker run -v 容器内目录
```

具名挂载：

```bash
# 不指定主机目录，但是指定一个名称
docker run -v 名称:容器内目录
```

可以看到，匿名挂载和具名挂载都是没有指定目录的，所有的容器中的卷，没有指定目录的情况下都是在主机`/var/lib/docker/volumes/xxx/_data`下面，`xxx`代表名称，当没有指定名称时，`docker`会随机分配名称。

### 指定数据卷的读写权限

挂载数据卷时，还可以指定数据卷的读写权限（只针对容器内部）。

```bash
docker run -v 主机目录:容器目录:ro/rw

# ro表示只读，rw表示可读可写
# 如果指定为ro 那么这个路径就只能通过主机进行操作，容器内部没有权限进行修改
```

## 数据卷容器

数据卷容器可以多个容器之间共享数据：

```bash
# 比如我们已经有了一个容器docker01,上面已经挂载了两个目录volume01和volume02

# 新创建一个容器docker02
docker run -it --name docker02 --volumes-from docker01 centos

# --volumes-from 命令可以指定docker02与docker01
# volume01和volume02两个目录的共享
# docker02也会生成对应的目录，并且和docker01同步
# docker01是主容器
# 即使把docker01删了，数据依然会存在
# 如果还有另外的容器03，即使01删了02和03之间的数据共享不会受到影响
```

# Dockerfile

## 什么是Dockerfile

在`Docker`中创建镜像最常用的方式，就是使用`Dockerfile`。`Dockerfile`是一个`Docker`镜像的描述文件，我们可以理解成火箭发射的`A`、`B`、`C`、`D`…的步骤。`Dockerfile`其内部**包含了一条条的指令**，**每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建**。 

一个`Dockerfile`的示例如下所示：

```dockerfile
# 基于centos镜像
FROM centos

# 维护人的信息
MAINTAINER init-center <init-center@foxmail.com>

# 安装httpd软件包
RUN yum -y update
RUN yum -y install httpd

# 开启80端口
EXPOSE 80

# 复制网站首页文件至镜像中web站点下
ADD index.html /var/www/html/index.html

# 复制该脚本至镜像中，并修改其权限
ADD run.sh /run.sh
RUN chmod 775 /run.sh

# 当启动容器时执行的脚本文件
CMD ["/run.sh"]
```

由上可知，`Dockerfile`结构大致分为四个部分：

1. 基础镜像信息
2. 维护者信息
3. 镜像操作指令
4. 容器启动时执行指令

## dockerfile指令

`Docker`以从上到下的顺序运行`Dockerfile`的指令。为了指定基本镜像，第一条指令必须是`FROM`（如果不想基于其他镜像，可以基于`scratch`，它是一个官方的空镜像）。一个声明以`＃`字符开头则被视为注释。可以在`Docker`文件中使用`RUN`，`CMD`，`FROM`，`EXPOSE`，`ENV`等指令。 

**指令必须大写。**

### FROM

指明构建的新镜像是来自于哪个基础镜像，例如：

```dockerfile
FROM centos:6
```

### MAINTAINER

指明镜像维护着及其联系方式（一般是邮箱地址），例如：

```dockerfile
MAINTAINER init-center <init-center@foxmail.com>
```

不过，`MAINTAINER`并不推荐使用，更推荐使用`LABEL`来指定镜像作者，例如：

```dockerfile
# LABEL 可以定义一些元信息
# 可以理解为注释吧
LABEL maintainer="init.center"
LABEL version="1.0"
LABEL description="This is description"
```

### RUN

构建镜像时运行的`Shell`命令，例如：

```dockerfile
RUN ["yum", "install", "httpd"]
RUN yum install httpd
```

又如，我们在使用微软官方`ASP.NET Core Runtime`镜像时往往会加上以下`RUN`命令，弥补无法在默认镜像下使用`Drawing`相关接口的缺憾：

```dockerfile
FROM microsoft/dotnet:2.2.1-aspnetcore-runtime
RUN apt-get update
RUN apt-get install -y libgdiplus
RUN apt-get install -y libc6-dev
RUN ln -s /usr/lib/libgdiplus.so /lib/x86_64-linux-gnu/libgdiplus.so
```

为了美观，复杂的`RUN`请用斜线换行，避免无用分层，合并多条命令成一行。

```dockerfile
# 使用 && 合并成一条，使用 \ 换行
RUN yum update && yum install -y vim \
	python-dev
```

### CMD

启动容器时执行的`Shell`命令，例如：

```dockerfile
CMD ["-C", "/start.sh"] 
CMD ["/usr/sbin/sshd", "-D"] 
CMD /usr/sbin/sshd -D
```

`CMD`是容器启动时默认执行的命令，**如果`docker run`指定了其他命令，`CMD`会被忽略（比如指明了运行的命令是`/bin/bash`），如果定义了多个`CMD`，只有最后一个会执行**。

### EXPOSE

声明容器运行的服务端口，例如：

```dockerfile
EXPOSE 80 443
```

### ENV

设置容器内环境变量，例如：

```dockerfile
ENV MYSQL_ROOT_PASSWORD 123456
ENV JAVA_HOME /usr/local/jdk1.8.0_45
```

由`EVN`指令声明的环境变量也可以用在`Dockerfile`的一些指令中作为变量使用。转义符也将类似变量的语法转义为语句。
在`Dockerfile`引用环境变量可以使用`$variable_name`或`${variable_name}`。它们是等同的，其中大括号的变量是用在没有空格的变量名中的，如`${foo}_bar`。 

### ADD

拷贝文件或目录到镜像中，例如：

```dockerfile
ADD <src>...<dest>
ADD html.tar.gz /var/www/html
ADD https://xxx.com/html.tar.gz /var/www/html
```

***PS：***如果是`URL`或压缩包，会自动下载或自动解压。

### COPY

拷贝文件或目录到镜像中，用法同`ADD`，只是不支持自动下载和解压，例如：

```dockerfile
COPY ./start.sh /start.sh
```

### ENTRYPOINT

启动容器时执行的`Shell`命令，同`CMD`类似，只是由`ENTRYPOINT`启动的程序**不会被`docker run`命令行指定的参数所覆盖**，而且，**这些命令行参数会被当作参数传递给`ENTRYPOINT`指定的程序(`CMD`会直接被命令行指定的的参数，比如`/bin/bash`替换掉，而`ENTRYPOINT`会追加到自己后面）**，例如：

```dockerfile
ENTRYPOINT ["/bin/bash", "-C", "/start.sh"]
ENTRYPOINT /bin/bash -C '/start.sh'
```

***PS：***`Dockerfile`文件中也可以存在多个`ENTRYPOINT`指令，但仅有最后一个会生效。

### VOLUME

指定容器挂载点到宿主机自动生成的目录或其他容器，例如：

```dockerfile
VOLUME ["/var/lib/mysql"]
```

***PS：***一般不会在`Dockerfile`中用到，更常见的还是在`docker run`的时候指定`-v`数据卷。

### USER

为`RUN`、`CMD`和`ENTRYPOINT`执行`Shell`命令指定运行用户，例如：

```dockerfile
USER <user>[:<usergroup>]
USER <UID>[:<UID>]
USER init-center
```

### WORKDIR

为`RUN`、`CMD`、`ENTRYPOINT`以及`COPY`和`AND`设置工作目录，例如：

```dockerfile
# 如果没有会自动创建
# WORKDIR 相当于 CD
WORKDIR /data
WORKDIR demo
RUN pwd		# 输出结果应该是/data/demo
```

### HEALTHCHECK

告诉`Docker`如何测试容器以检查它是否仍在工作，即健康检查，例如：

```dockerfile
HEALTHCHECK --interval=5m --timeout=3s --retries=3 \
CMD curl -f http:/localhost/ || exit 1
```

其中，一些选项的说明：

- ` --interval=DURATION (default: 30s)`：每隔多长时间探测一次，默认`30`秒
- `-- timeout= DURATION (default: 30s)`：服务响应超时时长，默认`30`秒
- `--start-period= DURATION (default: 0s)`：服务启动多久后开始探测，默认`0`秒
- `--retries=N (default: 3)`：认为检测失败几次为宕机，默认`3`次

一些返回值的说明：

- `0`：容器成功是健康的，随时可以使用
- `1`：不健康的容器无法正常工作
- `2`：保留不使用此退出代码

### ARG

在构建镜像时，指定一些参数，例如：

```dockerfile
FROM centos:7
ARG user # ARG user=root
USER $user
```

这时，我们在`docker build`时可以带上自定义参数`user`了，如下所示：

```bash
docker build --build-arg user=init-center Dockerfile .
```

## 根据dockerfile构建镜像

```bash
docker build -t ImageName:TagName dir
```

**选项**

- `-t` − 给镜像加一个`Tag`
- `ImageName` − 给镜像起的名称
- `TagName` − 给镜像的`Tag`名
- `Dir` − `Dockerfile`所在目录

## 发布自己的镜像

首先要在[镜像仓库]( https://hub.docker.com)注册账号。

```bash
# 首先要登录
# 输入id直接回车
# 然后输入密码即可

docker login -u id
password：
```

登录成功后开始发布：
```bash
# 首先使用tag命令修改镜像为规范的镜像
# 规范的名称为 dockerhub用户名/镜像名
# 比如 initcenter/mycentos
# 不指定tag默认为latest
# docker tag 可以使用 -f参数强制覆盖
docker tag oldImageName:tag newImageName:tag
docker push ImageName:TagName
```

# Docker网络

在各个容器之间，可以通过`docker0`进行转发通信，所以两个容器之间通过`ip`地址是可以相互`ping`通的，但现实中的情况往往是不确定具体的`ip`的，因为`ip`地址并不是一定不变的，假如`ip`地址改变，那么容器之间的通信就会出现问题？

## --link

在`docker run`时使用` --link`这个参数就可以实现两个容器之间使用容器名相互通信，比如：

```bash
# 启动时指定--link tomcat01
docker run -d -P --name tomcat02 --link tomcat01 tomcat
# 这时就可以通过名称ping通tomcat01
dockerexec -it tomcat02 ping tomcat01
```

但这个是**单向**的，可以通过`tomcat02`连通`tomcat01`，但是相反却是不行的。

因为`--link`的原理就是做了一个`hosts`的映射。`tomcat02`使用`--link`指定`tomcat01`，所以`tomcat02`的`hosts`中就将`tomcat01`这个名称以及它的`容器id`映射到了`tomcat01`的`ip`地址。

**现在已经不建议使用`--link`。**

## 自定义网络

我们可以通过`docker network ls`查看所有的网络：

```bash
[root@init_center ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
df750859e40a        bridge              bridge              local
1b76706f249f        host                host                local
a4a90daa4228        none                null                local
```

### 四种网络模式

| Docker网络模式 | 配置                         | 说明                                                         |
| -------------- | ---------------------------- | ------------------------------------------------------------ |
| `host`         | `-–net host`                 | 容器和宿主机共享`Network namespace`。                        |
| `container`    | `--net container:NAME_or_ID` | 容器和另外一个容器共享`Network namespace`。 `kubernetes`中的`pod`就是多个容器共享一个`Network namespace`。 |
| `none`         | `-–net none`                 | 容器有独立的`Network namespace`，但并没有对其进行任何网络设置，如分配`veth pair` 和网桥连接，配置`IP`等。 |
| `bridge`       | `-–net bridge`               | （默认为该模式）                                             |

#### host模式

如果启动容器的时候使用`host`模式，那么这个容器将不会获得一个独立的`Network Namespace`，而是和宿主机共用一个`Network Namespace`。容器将不会虚拟出自己的网卡，配置自己的`IP`等，而是使用宿主机的`IP`和`端口`。但是，容器的其他方面，如文件系统、进程列表等还是和宿主机隔离的。

使用`host`模式的容器可以直接使用宿主机的`IP`地址与外界通信，容器内部的服务端口也可以使用宿主机的端口，不需要进行`NAT`，`host`最大的优势就是网络性能比较好，但是`docker host`上已经使用的端口就不能再用了，网络的隔离性不好。

#### container模式

这个模式指定新创建的容器和已经存在的一个容器共享一个 `Network Namespace`，而不是和宿主机共享。新创建的容器不会创建自己的网卡，配置自己的 `IP`，而是和一个指定的容器共享 `IP`、端口范围等。同样，两个容器除了网络方面，其他的如文件系统、进程列表等还是隔离的。两个容器的进程可以通过 `lo` 网卡设备通信。

#### none模式

使用`none`模式，`Docker`容器拥有自己的`Network Namespace`，但是，并不为`Docker`容器进行任何网络配置。也就是说，这个`Docker`容器没有网卡、`IP`、路由等信息。需要我们自己为`Docker`容器添加网卡、配置`IP`等。

这种网络模式下容器只有`lo`回环网络，没有其他网卡。`none`模式可以在容器创建时通过`--network=none`来指定。这种类型的网络默认无法联网，封闭的网络能很好的保证容器的安全性。

#### bridge模式

当`Docker`进程启动时，会在主机上创建一个名为`docker0`的虚拟网桥，此主机上启动的`Docker`容器会连接到这个虚拟网桥上。虚拟网桥的工作方式和物理交换机类似，这样主机上的所有容器就通过交换机连在了一个二层网络中。

从`docker0`子网中分配一个`IP`给容器使用，并设置`docker0`的`IP`地址为容器的默认网关。在主机上创建一对虚拟网卡`veth pair`设备，`Docker`将`veth pair`设备的一端放在新创建的容器中，并命名为`eth0`（容器的网卡），另一端放在主机中，以`vethxxx`这样类似的名字命名，并将这个网络设备加入到`docker0`网桥中。可以通过`brctl show`命令查看。

`bridge`模式是`docker`的默认网络模式，不写`--net`参数，就是`bridge`模式。使用`docker run -p`时，`docker`实际是在`iptables`做了`DNAT`规则，实现端口转发功能。可以使用`iptables -t nat -vnL`查看。

### 创建和使用自定义网络

```bash
# 首先使用create命令创建自定义网络
# --driver 定义模式
# --subnet 定义子网
# --gateway 定义网关
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mybridge
```

然后就可以通过`--net`参数来使用自定义的这个网络：

```bash
docker run -d -P --name tomcat02 --net mybridge tomcat
```

当多个容器都连接到这个网络时，它们之间就可以通过容器名来`ping`通。

这样就可以让不同的集群使用不同的网络，保证集群是安全和健康的。

已经创建的容器想要连接到这个自定义的网络，可以：
```bash
docker network connect 网络名 容器名
# 比如
docker network connect my-bridge tomcat01
```

此时，`tomcat01`这个容器，不仅连接了默认的`bridge0`，同时还连接了我们自定义的`my-bridge`。

# Docker Compose

 `Compose` 是用于定义和运行多容器 `Docker` 应用程序的工具。通过 `Compose`，您可以使用 `YML` 文件来配置应用程序需要的所有服务。然后，使用一个命令，就可以从 `YML` 文件配置中创建并启动所有服务（容器）。 

`Compose` 使用的三个步骤：

- 使用 `Dockerfile` 定义应用程序的环境。
- 使用 `docker-compose.yml` 定义构成应用程序的服务，这样它们可以在隔离环境中一起运行。
- 最后，执行 `docker-compose up` 命令来启动并运行整个应用程序。