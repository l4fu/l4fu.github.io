# dockerfile安全介绍
## 一、Dockerfile 实践

> 避免不必要的安装包 
> 
> 一个容器一个进程 
> 
> 最小化layers的数量 
> 
> 排序多行参数

## 二、Dockerfile 示例

```
# Base _v_s 基础镜像
FROM centos
#MAINTAINER 维护者信息
MAINTAINER lorenwe 
#ENV 设置环境变量
ENV PATH /usr/local/nginx/sbin:$PATH
#ADD  文件放在当前目录下，拷过去会自动解压
ADD nginx-1.13.7.tar.gz /tmp/
#RUN 执行以下命令
RUN rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7 \
    && yum update -y \
    && yum install -y vim less wget curl gcc automake autoconf libtool make gcc-c++ zlib zlib-devel openssl openssl-devel perl perl-devel pcre pcre-devel libxslt libxslt-devel \
    && yum clean all \
    && rm -rf /usr/local/src/*
RUN useradd -s /sbin/nologin -M www
#WORKDIR 相当于cd
WORKDIR /tmp/nginx-1.13.7
RUN ./configure --prefix=/usr/local/nginx --user=www --group=www --with-http_ssl_module --with-pcre && make && make install
RUN cd / && rm -rf /tmp/
COPY nginx.conf /usr/local/nginx/conf/
#EXPOSE 映射端口
EXPOSE 80 443
#ENTRYPOINT 运行以下命令
ENTRYPOINT ["nginx"]
#CMD 运行以下命令
CMD ["-h"]
```

ENV 设置环境变量，简单点说就是设置这个能够帮助系统找到所需要运行的软件，比如我上面写的是 “ENV PATH /usr/local/nginx/sbin:$PATH”，这句话的意思就是告诉系统如果运行一个没有指定路径的程序时可以从 /usr/local/nginx/sbin 这个路径里面找，只有设置了这个，后面才可以直接使用 ngixn 命令来启动 nginx，不然的话系统会提示找不到应用。

ADD 用于添加文件，源文件可以是一个文件，或者是一个 URL 都行，如果源文件是一个压缩包，在构建镜像的时候会自动的把压缩包解压开来，示例写的是ADD nginx-1.13.7.tar.gz /tmp/其中 nginx-1.13.7.tar.gz 这个压缩包是必须要在 dockefile 文件目录内。

RUN 就是执行命令，RUN 可以执行多条命令， 用 && 隔开就行，如果命令太长要换行的话在末尾加上 ‘\\’ 就可以换行命令， yum clean all ，把所有的 yum 缓存清掉，这可以减少构建出来的镜像大小，rm -rf /usr/local/src/清除用户源码文件，都是起到减少构建镜像大小的作用，每用一次 RUN 命令就会生成一层镜像。

当 ENTRYPOINT 和 CMD 都存在时 CMD 中的命令会以 ENTRYPOINT 中命令的参数形式来启动容器，例如上面的示例 dockerfile，在启动容器时会以命令为 nginx -h来启动容器，遗憾的是这样不能保持容器运行，所以可以这样启动 docker run -it lorenwe/centos\_nginx -c /usr/local/nginx/conf/nginx.conf，那么容器启动时运行的命令就是 nginx -c /usr/local/nginx/conf/nginx.conf，可以自定义启动参数了。

如果一个服务不需要任何权限即可运行,可以使用USER指令改变为非Root用户,只需在Dockerfile中添加RUN指令 RUN groupadd -r postgres && useradd -r -g postgres postgres

VOLUME指令用于公开docker容器创建的数据库存储区域,配置,文件或者文件夹，建议使用VOLUME来维护用户服务可变的部分，即把可变的数据公开的外部存储。

## 三、Dockerfile 安全

### 3.1 使用非 root 用户运行容器

Docker默认使用root用户运行容器。 然后将该名称空间映射到正在运行的容器中的root用户时，这意味着该容器可能对Docker主机具有root访问权限。 如果容器本身具有漏洞，则由root用户在容器上运行应用程序可以进一步扩大攻击面，并为特权升级提供简便的途径。 为了最大程度地减少曝光，请选择在Docker映像中为应用程序创建一个专用用户和一个专用组； 在Dockerfile中使用USER指令来确保容器以最小的特权访问来运行应用程序。 图像中可能不存在特定用户； 使用Dockerfile中的说明创建该用户

```
FROM ubuntu:latest
RUN useradd -r -u 1001 -g appuser appuser
USER appuser
ENTRYPOINT [“sleep”, “infinity”]
```

```
FROM ubuntu
RUN mkdir /app
RUN groupadd -r lirantal && useradd -r -s /bin/false -g lirantal lirantal
WORKDIR /app
COPY . /app
RUN chown -R lirantal:lirantal /app
USER lirantal
CMD node index.js
```

创建一个没有密码，没有设置主目录并且没有shell的系统用户（-r） 

将创建的用户添加到先前创建的现有组中（使用groupadd） 

与创建的组关联，将最终参数集添加到我们要创建的用户名中

### 3.2 配置使用可信镜像源

Docker内容信任（DCT）  
![](_v_images/311025108230292.jpeg)

Docker Content Trust（DCT）提供了使用数字签名处理发送到远程Docker注册表和从远程Docker注册表接收的数据的功能。这些签名允许客户端或运行时验证特定镜像标签的完整性和发布者。

如果使用者启用DCT，则他们只能使用受信任的镜像进行拉取，运行或构建。 启用DCT有点像对注册表应用“过滤器”。 消费者仅“看到”签名的镜像标签，而对他们而言“不可见”的是不太希望使用的未签名的镜像标签。

签名映像的前提条件是已连接公证服务器的Docker注册表（例如Docker Hub或Docker Trusted Registry）。在Docker CLI中，我们可以使用$ docker trust命令语法签署并推送容器映像。

将委托私钥添加到本地Docker信任库中。 （默认情况下，它存储在~/.docker/trust/中）。如果使用$ docker trust key generate生成委托密钥，则私钥会自动添加到本地信任存储中。如果要导入单独的密钥（例如，从UCP客户端捆绑包中导入），则需要使用$ docker trust key load命令。接下来，需要将委派公钥添加到Notary服务器。 这特定于公证人中称为全局唯一名称（GUN）的特定镜像仓库。 如果这是您第一次向该镜像仓库添加委派，则此命令还将使用本地Notary规范根密钥来启动存储库。

```
# 生成key
$ docker trust key generate jeff
# 或者使用已有的key
$ docker trust key load key.pem --name jeff
# 配置DCT的环境变量，启用DCT
$ export DOCKER_CONTENT_TRUST=1
#对镜像打标签，从而可以将其推送到目标镜像库。本例中，会将其推送到位于我 Docker Hub 个人账户命名空间下的镜像库。
$ docker  tag alpine:latest nigelpoulton/dockerbook:v1
# 登录到 Docker Hub（或其他镜像库）以便推送镜像。
$ docker login
# 使用push推送镜像
$ docker push dtr.example.com/admin/demo:1
# 查看标签或存储库的远程信任数据
$ docker trust inspect --pretty dtr.example.com/admin/demo:1
```

Docker Enterprise Engine中的Docker Content Trust阻止用户使用来自未知来源的容器映像，也将阻止用户从未知来源的基础层构建容器映像。 受信任的来源可能包括在Docker Hub上找到的Official Docker Images或用户受信任的来源，其中包含使用上述命令签名的存储库和标签。

引擎签名验证可防止以下情况：

> docker容器运行的未签名或更改镜像。
> 
> docker 拉取未签名或更改的镜像。
> 
> docker build其中FROM镜像未签名或未擦除。
> 
> 获取镜像时，通过配置 docker 守护进程/etc/docker/daemon.json配置获取可信来源的镜像。**Docker CE 不支持**

```
{
  "content-trust": {
    "trust-pinning": {
      "official-library-_v_s": true
    },
    "mode": "enforced"
  }
}
```

拉去镜像

```
$ docker pull dtr.example.com/user/:1
Error: remote trust data does not exist for dtr.example.com/user/: dtr.example.com does not have trust data for dtr.example.com/user/
$ docker pull dtr.example.com/user/@sha256:d149ab53f8718e987c3a3024bb8aa0e2caadf6c0328f1d9d850b2a2a67f2819a
sha256:ee7491c9c31db1ffb7673d91e9fac5d6354a89d0e97408567e09df069a1687c1: Pulling from user/
ff3a5c916c92: Pull complete
a59a168caba3: Pull complete
Digest: sha256:ee7491c9c31db1ffb7673d91e9fac5d6354a89d0e97408567e09df069a1687c1
Status: Downloaded newer  for dtr.example.com/user/@sha256:ee7491c9c31db1ffb7673d91e9fac5d6354a89d0e97408567e09df069a1687c1
```

### 3.3 使用最小的基础镜像，不安装任何多余的软件包

如果不是这样的话增加了镜像文件大小，增加了受攻击面。

### 3.4 执行镜像扫描

一些开源的镜像扫描工具

> [Anchore Engine](https://github.com/anchore/anchore-engine)：Anchore Engine是用于分析容器镜像的工具。 除了基于CVE的安全漏洞报告之外，Anchore Engine还可以使用自定义策略评估Docker映像。策略基于白名单或黑名单、凭据、文件内容、配置类型或其他用户生成的提示。打包为Docker容器映像的Anchore可以独立运行，也可以在业务平台（例如Kubernetes）上运行。 它还可以与 CI/CD 的 Jenkins 和 GitLab 集成。
> 
> [Clair](https://coreos.com/clair/docs/latest/)：Clair 摄取了许多漏洞数据源，例如Debian Security Bug Tracker，Ubuntu CVE Tracker和Red Hat Security Data。 由于Clair使用了许多CVE数据库，因此其审核是全面的。Clair首先为容器映像中的特征列表建立索引。 然后，使用Clair API，开发人员可以在数据库中查询与特定图像有关的漏洞。。
> 
> [dagda](https://github.com/eliasgranderubio/dagda)：a tool to perform static analysis of known vulnerabilities, trojans, viruses, malware & other malicious threats in docker _v_s/containers and to monitor the docker daemon and running docker containers for detecting anomalous activities
> 
> OpenScap：一套自动审核工具，用于按照NIST认证的安全内容自动化协议（SCAP）来检查软件中的配置和已知漏洞。 不是特定用于容器的，但确实包含一定程度的支持。

介绍下 Clair  
![](_v_images/307925108237623.jpeg)

安装过程：

```
$ mkdir $PWD/clair_config
$ curl -L https://raw.githubusercontent.com/coreos/clair/master/config.yaml.sample -o $PWD/clair_config/config.yaml
$ docker run -d -e POSTGRES_PASSWORD="password" -p 5432:5432 postgres:9.6
$ docker run --net=host -d -p 6060-6061:6060-6061 -v $PWD/clair_config:/config quay.io/coreos/clair:latest -config=/config/config.yaml
```

config.yaml的配置文件需要做下修改（对应行）

```
source: host=localhost port=5432 user=postgres password=password sslmode=disable statement_timeout=60000
```

测试部署成功

```
[root@localhost ~]# curl -X GET -I http://localhost:6061/health
HTTP/1.1 200 OK
Server: clair
Date: Wed, 18 Mar 2020 13:32:33 GMT
Content-Length: 0
[root@localhost ~]# curl http://localhost:6060/v1/namespaces
...
```

> API 接口使用不方便，整合其他工具[https://github.com/quay/clair/blob/master/Documentation/integrations.md](https://github.com/quay/clair/blob/master/Documentation/integrations.md)

安装第三方客户端 claircli，执行扫描，本地生成 html 的报告。

```
[root@localhost home]# pip install claircli
[root@localhost home]# claircli -l 127.0.0.1 postgres 2020-03-18 21:45:50,870|INFO|Starting local http server
2020-03-18 21:45:50,871|INFO|Local http server serving at port: 10963
2020-03-18 21:45:50,872|INFO|*****************************1******************************
2020-03-18 21:45:50,872|INFO|Analyzing <Image: postgres>
2020-03-18 21:45:56,449|INFO|Push layer [1/14]: 5c77fc16775dbda7fafd2db94684c22de93066e29dd686a2f46d995615458476
2020-03-18 21:45:56,715|INFO|Push layer [2/14]: 54a426d01ba59fb558c7cf6681c6840736e5d0654a62f0c383d227637cdee0db
2020-03-18 21:45:56,777|INFO|Push layer [3/14]: a31210340f3a4905656af9be0ee9ffe3291380e6764c5d2d62831c2626451231
2020-03-18 21:45:56,787|INFO|Push layer [4/14]: fb1bc94faf7fe27b7f7a36c980c8407d9f28bab2c047cf389dda8eb9349cfa32
2020-03-18 21:45:56,801|INFO|Push layer [5/14]: 8f5652501fa074cb5b9d54c6dceaa966c6591000837076de4c8aaf90ad3c919a
2020-03-18 21:45:56,834|INFO|Push layer [6/14]: 3f89d1179df8c1c036baf12dd6204a6f17f752e3c0f96b0d25eb408bcb6f2313
2020-03-18 21:45:56,876|INFO|Push layer [7/14]: 1034326be7395271eed2e8e3fbb6e8719ec21533776646eda190ab6d5d690404
2020-03-18 21:45:56,886|INFO|Push layer [8/14]: eba8f87a20b5efae0ab4540e5932e3920879ffbfcde08fd2664bd35a8392a48e
2020-03-18 21:45:56,896|INFO|Push layer [9/14]: 9fcb1e2984e4683b4c8e724b64836097b19d03e77f98fbe226ffd581fdfe0bcd
2020-03-18 21:45:57,101|INFO|Push layer [10/14]: 17685e9204242fce5aa038a05cf160bd4a7f6b516bb461a5c8ca5977f2fa1e74
2020-03-18 21:45:57,110|INFO|Push layer [11/14]: 4a4b72a0e224e07071b6d60e8c4335b5996ff58f894fc3c10e7bbf523de924e8
2020-03-18 21:45:57,121|INFO|Push layer [12/14]: 02053f40fe92249821c297ab7b9a10ee2071b005c541e022d74c3d1fbde5a28f
2020-03-18 21:45:57,132|INFO|Push layer [13/14]: 92e8c07039ed217658e3d78e3681dc19bcf8193caee4c85445ffa2a77cce1925
2020-03-18 21:45:57,142|INFO|Push layer [14/14]: 9d681078d309476626ccd11c7d2fa2f9010d6d4f67b3dd935075fd9ab16fad88
2020-03-18 21:45:57,153|INFO|Fetch vulnerabilities for <Image: postgres>
2020-03-18 21:45:57,172|INFO|Defcon1 : 0
2020-03-18 21:45:57,172|INFO|Critical : 0
2020-03-18 21:45:57,172|INFO|High : 0
2020-03-18 21:45:57,172|INFO|Medium : 0 2020-03-18 21:45:57,172|WARNING|Low : 10
2020-03-18 21:45:57,173|WARNING|Negligible : 47
[root@localhost home]# ls
clair-postgres.html
```

### 3.5 执行 HEALTHCHECK

HEALTHCHECK指令具有两种形式： 

> HEALTHCHECK \[OPTIONS\] CMD命令（通过在容器内部运行命令来检查容器的运行状况） 
> 
> HEALTHCHECK NONE（禁用从基本映像继承的任何健康检查）

HEALTHCHECK指令告诉Docker如何测试容器以检查其是否仍在工作。这样可以检测到诸如Web服务器陷入无限循环并且无法处理新连接的情况，即使服务器进程仍在运行。

指定容器的运行状况检查后，除了其正常状态外，它还具有运行状况。此状态最初开始。只要运行状况检查通过，它就会变得健康（无论以前处于什么状态）。在一定数量的连续故障之后，它变得不健康。

在CMD之前可以显示的选项是：

> \--interval=DURATION (default: 30s)
> 
> \--timeout=DURATION (default: 30s)
> 
> \--start-period=DURATION (default: 0s)
> 
> \--retries=N (default: 3)

运行状况检查将首先在容器启动后的间隔秒数内运行，然后在每次之前的检查完成后的间隔秒数内运行。

如果单次检查花费的时间超过超时秒数，则认为检查失败，需要重新尝试连续进行的健康检查失败，才能将容器视为不健康。

开始时间段为需要时间进行引导的容器提供了初始化时间。在此期间的探针故障将不计入最大重试次数。但是，如果运行状况检查在启动期间成功，则认为该容器已启动，并且所有连续失败将计入最大重试次数。

Dockerfile中只能有一条HEALTHCHECK指令。如果您列出多个，则只有最后一个“健康检查”才会生效。

CMD关键字后面的命令可以是shell命令（例如HEALTHCHECK CMD / bin / check-running）或exec数组（与其他Dockerfile命令一样;有关详细信息，请参见ENTRYPOINT）。

命令的退出状态指示容器的健康状态。可能的值为：

> 0：成功-容器健康且可以使用
> 
> 1：不健康-容器无法正常工作
> 
> 2：保留-请勿使用此退出代码

例如，要每五分钟检查一次，以便网络服务器能够在三秒钟内为网站的首页提供服务：

```
HEALTHCHECK --interval=5m --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
```

为了帮助调试失败的探针，该命令在stdout或stderr上写入的任何输出文本（UTF-8编码）都将以健康状态存储，并可以通过docker inspect查询。此类输出应保持简短（当前仅存储前4096个字节）。

当容器的健康状态发生变化时，将以新状态生成health\_status事件。

### 3.6 不应单独或在Dockerfile中的一行中使用OS软件包管理器更新指令，如apt-get update 或yum update

在Dockerfile上的一行中添加更新指令将导致更新层被缓存。 当您以后再使用同一指令构建任何映像时，这将导致使用先前本地缓存的更新层，从而有可能阻止将任何新的更新应用于以后的构建。

在安装软件包时，应同时使用更新说明，安装说明和版本说明。 这样可以防止缓存并强制提取所需的版本。 或者，可以在docker构建过程中使用--no-cache标志，以避免使用缓存的图层。

```
docker build [OPTIONS] PATH | URL | --no-cache
```

### 3.7 删除 setuid 和 setgid 权限

可执行文件上可以设置两种特殊权限：设置用户ID（setuid）和设置组ID（sgid）。 这些权限允许使用所有者或组的特权来执行要执行的文件。 例如，如果文件由root用户拥有并且设置了setuid位，则无论是谁执行该文件，它都将始终以root用户特权运行。

应用程序可能不需要任何setuid或setgid二进制文件。 如果可以禁用或删除此类二进制文件，则可以停止将它们用于缓冲区溢出，路径遍历/注入和特权升级攻击的任何可能性。

setuid位此位用于具有可执行权限的文件。 setuid位仅表示运行可执行文件时，它将权限设置为创建它的用户（所有者），而不是将其设置为启动它的用户。 类似地，有一个setgid位对gid做同样的事情。setgid会影响文件和目录。 当在文件上使用时，它以拥有该文件的用户组的特权执行，而不是以执行该文件的用户组的特权执行。将目录设置为该位后，该目录中的文件集将与父目录的组具有相同的组，而不是创建这些文件的用户的组。 这用于文件共享，因为属于父目录组的所有用户现在都可以对其进行修改。

在 linux 里查找和删除有 setuid 权限的文件

```
find / -perm +6000 -type f -exec \  ls -ld {} \; 2> /dev/null 
# 在容器里查找
$ docker run debian find / -perm +6000 -type f -exec ls -ld {} \; 2> /dev/null
```

运行容器时删除 setuid 权限

```
$ docker run -d --cap-drop SETGID --cap-drop SETUID <container_id>
```

在 dockerfile build 成功后执行 setuid 和 setgid 的检查

```
docker run --rm defanged-debian find / -perm +6000 -type f -exec ls -ld {} \; 2> /dev/null | wc -l
```

结果为 0 表示创建的镜像没有特殊权限的文件。

### 3.8 在 Dockerfile 里使用 CPOY 指令不用 ADD 指令

COPY指令仅将文件从本地主机复制到容器文件系统。 ADD指令可能会从远程URL检索文件并执行诸如将它们解压缩之类的操作。 因此，ADD指令会带来安全风险。 例如，恶意文件可能直接从URL访问而无需扫描，或者可能存在与解压缩程序有关的漏洞。

### 3.9 Dockerfile 中不应包含密码、密钥等信息

dockerfile 自身访问权限可能存在访问控制漏洞，另外有权限在 docker 主机上运行docker history的用户都能看到 dockerfile 中的密钥信息。

以下示例 Dockerfile 只是为了证明可以访问密钥，密钥信息显示在构建输出中。 构建的最终镜像没有密钥文件：

```
$ docker build --no-cache --progress=plain --secret id=mysecret,src=mysecret.txt .
...
#8 [2/3] RUN --mount=type=secret,id=mysecret cat /run/secrets/mysecret
#8       digest: sha256:5d8cbaeb66183993700828632bfbde246cae8feded11aad40e524f54ce7438d6
#8         name: "[2/3] RUN --mount=type=secret,id=mysecret cat /run/secrets/mysecret"
#8      started: 2018-08-31 21:03:30.703550864 +0000 UTC
#8 1.081 WARMACHINEROX
#8    completed: 2018-08-31 21:03:32.051053831 +0000 UTC
#8     duration: 1.347502967s
#9 [3/3] RUN --mount=type=secret,id=mysecret,dst=/foobar cat /foobar
#9       digest: sha256:6c7ebda4599ec6acb40358017e51ccb4c5471dc434573b9b7188143757459efa
#9         name: "[3/3] RUN --mount=type=secret,id=mysecret,dst=/foobar cat /foobar"
#9      started: 2018-08-31 21:03:32.052880985 +0000 UTC
#9 1.216 WARMACHINEROX
#9    completed: 2018-08-31 21:03:33.523282118 +0000 UTC
#9     duration: 1.470401133s
...
```

最佳的方法是从网络服务器上获取密钥信息

```
FROM busyboxRUN echo "The secret is: " && \
    wget -O - -q http://localhost:8000/secret.txt
```

### 3.10 使用软件白名单，只安装验证过的软件包

![](_v_images/304895108217457.jpeg)  
通常是集成到 devops 的过程中

dependency check，支持各种语言，持续社区更新，有命令行、docker部署、丰富的报告格式。原理是基于开源的 NVD 开源漏洞数据库来监测安全漏洞的，缺点是基于文件名来使用Lucene分词匹配cpe不准确，这个环节引入一部分误报的问题。

dependency track，是owasp新出的工具，界面较好，方便管理。

SAP公司开源了Java和Python SCA 工具，提供静态代码安全性测试功能。

> 对于node应用，有npm audit fix命令可以自动升级版本，原理是通过nodesecurity.io的接口查询漏洞，详情参看[https://www.npmjs.com/solutions/security-compliance](https://www.npmjs.com/solutions/security-compliance)。

如果基于github，[那么有snyk.io](http://xn--snyk-uj5f642k731d.io/)，git alert、lgtm、sonar oss免费服务可以使用。另外gitlab企业版，也直接提供了安全功能。

商业产品  
![](_v_images/302815108229590.jpeg)

## 参考

> [https://docs.docker.com/engine/swarm/secrets/](https://docs.docker.com/engine/swarm/secrets/)
> 
> [https://www.secrss.com/articles/15038](https://www.secrss.com/articles/15038)